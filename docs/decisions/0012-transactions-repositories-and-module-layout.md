# 0012: Transactions, repositories and module layout

- **Date and status:** 16/11/2026. Written in the planning review, decided by Claude at Victor's request; Victor reviews it in the pull request.
- **Context:** The review of ADRs 0001 to 0011 found three gaps that the first lines of code would hit.
  - 0001 lists the top folders (`modules/`, `reusable/`, `providers/`, `entrypoints/`) and mentions `domain/`, while 0002 mentions `http/` and adapters. The inside of a module was never written down.
  - 0008 D3 needs **one transaction** that creates the generation record (`modules/plans`), reserves credit (`reusable/ai-usage`) and sends the job (queue port). 0001 puts the database behind a port. Nothing says how two modules and a port share a transaction.
  - 0001 and RNF-30 ask for a fake adapter for every port, the database included. An in-memory database does not reproduce transactions, unique indexes or `SKIP LOCKED`, which are exactly what 0008 relies on. 0009 D3 already runs integration tests against the real PostgreSQL.
  - 0010 D6 needs a database role that cannot update or delete audit rows. Roles are painful to add after the tables exist.
- **Decision:**
  - **D1. Module layout.** The same for `src/modules/<name>/` and `src/reusable/<name>/`:

    | Folder | Holds | May import |
    |---|---|---|
    | `index.ts` | The public entry point: types and a factory that receives its dependencies | Its own folders |
    | `domain/` | Pure functions and types. No I/O, no framework, no Prisma, no clock | `shared/` only |
    | `application/` | Use cases. They receive repositories and ports as parameters | `domain/`, `shared/`, other modules' `index` |
    | `adapters/` | Prisma repositories | Prisma, `domain/`, `shared/` |
    | `http/` | Routes, request and response mapping (schemas from `packages/contracts`) | `application/`, `shared/` |
    | `jobs/` | Queue job handlers | `application/`, `shared/` |

    A new top folder, `src/shared/`, holds the small kernel used everywhere: the result type, error types, IDs, money, the transaction handle and the clock port. `shared/` imports no module. dependency-cruiser enforces this table, on top of the rules in 0001.
  - **D2. The database is not faked.** Repositories are Prisma classes behind TypeScript interfaces owned by the module, so `domain/` and `application/` never import Prisma. There is one implementation, and it is tested against the real PostgreSQL (0009 D3). Fake adapters stay for the ports where they pay off: AI, e-mail, queue, clock and, later, file storage. This amends 0001 (item 2) and RNF-30.
  - **D3. Unit of work.** `shared/db` exposes `transaction(fn)`. It opens a database transaction and passes an opaque handle, `Tx`, to `fn`. Repository methods and the queue port accept that handle. Only `adapters/` and `providers/queue` can open it and reach the Prisma transaction client inside (pg-boss receives it through `fromPrisma`, 0008 D3). A use case then reads:

    ```ts
    await db.transaction(async (tx) => {
      const generation = await generations.create(tx, ownerId, planId);
      await aiUsage.reserve(tx, ownerId, maxCost, generation.id);
      await queue.send(tx, { generationId: generation.id });
    });
    ```

    `reusable/ai-usage` takes part in the transaction without knowing Docenda. The handle is always passed explicitly; nothing is hidden in a context.
  - **D4. Ownership by construction** (RNF-40). Every repository method for an author-owned resource takes `ownerId` as its first parameter. There is no method without it. Another author's resource answers `404`, never `403`, so its existence is not revealed. One generic e2e test tries every route of author A as author B.
  - **D5. Database roles.**

    | Role | Used by | Can |
    |---|---|---|
    | `docenda_owner` | The migration step only | Create and change tables; owns the `pgboss` schema setup |
    | `docenda_app` | API and worker | Read and write data; on `audit_event`, `access_record`, `lesson_plan_version` and `credit_entry`, insert and read only; on the skill catalog, read only (table by table in [DATA](../DATA.md), section 5) |
    | `docenda_retention` | The retention jobs | Delete expired rows from `audit_event` and `access_record` |

    A one-shot `migrate` service in Compose runs `prisma migrate deploy` as `docenda_owner`. The API and the worker start only after it finishes successfully. The application never runs migrations and cannot change the schema.
  - **D6. One schema file per module** (Prisma multi-file schema, to confirm with the first schema in F00). A module queries only its own tables. Foreign keys across modules are allowed in two cases only: to the user, and to reference data (the skill catalog). Everything else crosses modules through the `index`.
  - **D7. Errors and API shape.**
    - Expected failures (a rule that blocks, a missing resource, no credit) are returned as typed results from `domain/` and `application/`. Exceptions are for bugs.
    - `http/` maps each error to a problem document (RFC 9457) with a stable `code` and the request ID (0010 D2). No stack trace, no internal message.
    - Routes live under `/api/v1`.
  - **D8. Money and time.**
    - Costs are integers in nano-dollars (US$ 1 = 1,000,000,000), stored as `bigint`. At US$ 0.10 per million tokens, a thousandth of a cent per token is already a fraction. This refines RNF-42.
    - Timestamps are stored in UTC. Business periods (the monthly credit) use `America/Sao_Paulo`. Code reads time through the clock port, so tests can set it.
  - **D9. Test names and places.**

    | Kind | File | Runs against |
    |---|---|---|
    | Unit | `*.test.ts`, next to the code | Nothing external |
    | Integration | `*.int.test.ts` | PostgreSQL from Compose or the CI service |
    | End to end | `e2e/` | The built app, with Playwright |

    A test for a requirement or an invariant starts its name with the ID (`RF-41`, `INV-03`), so a search for the ID finds the test.
- **Alternatives considered:**
  - In-memory fake repositories: fast unit tests, but twice the code, and the fakes drift from what PostgreSQL really does.
  - An implicit transaction carried by `AsyncLocalStorage`: shorter signatures, but it hides which calls are inside the transaction and is easy to break across `await` boundaries in job handlers.
  - Row-level security in PostgreSQL for ownership: a stronger second layer, but it needs a session setting on every transaction, which fits Prisma poorly. Left as a later step (ROADMAP).
  - One database role for everything: simpler, but the audit trail would be editable by the application (0010 D6).
- **Consequences:**
  - Pros: every slice starts from the same layout; the three-way transaction of 0008 has a place to live; tests exercise the real database behaviour; the application cannot alter its own schema or its audit trail.
  - Cons: unit tests cover `domain/` only, so use cases need the database to be tested; the handle adds a parameter to repository methods; three roles mean grants to keep in the migrations.
  - Changes elsewhere: 0001 (item 2 and the layout), RNF-30, RNF-40 and RNF-42 in REQUIREMENTS, and 0008 (note of 16/11).
  - Cost: none.
- **Sources:** [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457), [PostgreSQL 18: GRANT](https://www.postgresql.org/docs/18/sql-grant.html), [Prisma: transactions](https://www.prisma.io/docs/orm/prisma-client/queries/transactions), [pg-boss adapters (Prisma)](https://pgboss.io/api/adapters) (checked for 0008 on 16/11/2026).
