# 0008: AI job queue on PostgreSQL (pg-boss)

- **Date and status:** 16/11/2026, approved by Victor.
- **Context:** Generating a plan takes about 30 to 60 s. RNF-34 asks for long tasks (AI generation, export) to run apart from the screens, so a failure in the queue never takes the site down. 0002 already planned a worker process, and 0004 picked PostgreSQL 18. Other requirements that shape the queue: no state in memory (RNF-33), validated JSON with at most 2 retries (RNF-35), retries with backoff and a circuit breaker (RNF-36), no second charge on a double click (RNF-37), progress on screen (RNF-04) and the usage limits (RF-71, RF-72). On 16/11 Victor set the v1 budget: up to 10 people, each with US$ 1 of credit, US$ 10 for the whole system (about 5 or 6 full generations per person at the 0005 D6 estimate). Versions and docs checked on 16/11/2026.
- **Decision:**
  - **D1. pg-boss 12** (12.38.2 on 16/11/2026, MIT, Node 22.12 or later, PostgreSQL 13 or later), pinned to `^12`. It lives in its own schema, `pgboss`, in the same database. It uses `SKIP LOCKED`: a worker that takes a job locks its row, and other workers skip it instead of waiting, so two workers never run the same job. It sits behind a `JobQueue` port, with the adapter in `src/providers/queue` and a fake in-memory adapter for tests (RNF-30).
  - **D2. One image, two entry points:** `entrypoints/web` (API) and `entrypoints/worker`. Day to day: `npm run dev` and `npm run dev:worker` in WSL. In Compose (0007), a `worker` service from the same image, with the same hardening as the API.
  - **D3. Flow of a generation:**
    1. `POST` on the plan opens **one transaction** that creates an `ai_generation` row (status `queued`), reserves the maximum cost of the operation from the author's credit (RF-73 limits times the model price, integers as in RNF-42), checks the limits (RF-71, RF-72) and sends the job with `fromPrisma(tx)`. It is all or nothing: there is never a job without a record, a record without a job, or a job without credit. The answer is `202` with the generation ID.
    2. The worker runs the 0005 D8 path and updates a `stage` field ("checking data", "generating", "validating", "checking the answer").
    3. The screen asks for the status every 2 s (polling). Showing stages meets RNF-04: the answer is JSON bound to a schema, which is not readable half-way, so streaming its text to the screen brings nothing. Between the worker and Anthropic, streaming stays, as the SDK recommends for long calls. This refines 0005 D4.
    4. The worker settles each call from `response.usage` (RF-74) and releases the rest of the reservation at the end. If the reservation runs out (for example, after retries), the generation stops with a clear message.
  - **D4. Retries in layers, without multiplying them** (three layers that each retry three times would make 27 calls):
    - the Anthropic SDK retries connection errors, 408, 409, 429 and 5xx twice, with short exponential backoff (its default);
    - a circuit breaker (cockatiel 4) wraps the AI adapter. After a few failures in a row, it refuses at once for about 60 s, with a clear message and no charge;
    - an invalid JSON answer is retried at most twice inside the job (RNF-35);
    - the queue retries only when the worker dies mid-job: `heartbeatSeconds: 30`, `expireInSeconds: 180`, `retryLimit: 1`. Known errors end the job with an error status in `ai_generation`, without throwing, so the queue does not retry them.
  - **D5. Double click (RNF-37):** a partial unique index allows only one active generation (`queued` or `running`) per plan. A second click gets the same generation ID back. The generate button is also disabled while a generation is active.
  - **D6. Personal data (LGPD):** a job carries **only IDs** (generation, plan, author), never the plan text or the class context; the worker reads them from the database. So the queue holds no copy of personal data, and deleting a plan or an account leaves nothing behind in it. Finished jobs are deleted after 1 day (`deleteAfterSeconds`). Jobs that fail every retry go to an `ai-generation-failed` queue (dead letter), also with IDs only, for the administrator to review.
  - **D7. Concurrency and scale:**
    - the worker runs 2 jobs at a time (`localConcurrency: 2`, in configuration). With a US$ 10 ceiling, more brings nothing;
    - to grow: more worker containers; `SKIP LOCKED` keeps them from doing the same job twice;
    - LISTEN/NOTIFY stays off in v1. The default 2 s polling is enough for a 40 s task, and LISTEN/NOTIFY does not work through PgBouncer in transaction mode;
    - export (RNF-34) uses the same queue with another job name, when it exists.
  - **D8. Redis later, with the path ready:**
    - Redis is not part of v1. If it comes, the queue stays on PostgreSQL to keep the all-or-nothing of D3. Redis would take other jobs: cache, shared counters and real-time messages;
    - Better Auth's per-IP counters already switch to the database or to Redis by configuration only (0006, scaling path). The AI limits are counted in the D3 transaction, so they stay on PostgreSQL;
    - a `Cache` port is created with the first real use of a cache, with Redis as one of its adapters;
    - signals that bring Redis in: more than one API container with the counters weighing on the database, a measured gain from a cache, or a real-time screen.
- **Alternatives considered:**
  - Graphile Worker 0.18: also on PostgreSQL, fast (LISTEN/NOTIFY) and written in TypeScript. It is still 0.x, retries 25 times over about 3 days by default and has no Prisma adapter.
  - BullMQ 6 with Redis: the market standard, with a ready dashboard. It needs one more container to secure, back up and pay for on AWS, and registering the generation and queuing the job become two separate writes that can end half done. Fixing that (the outbox pattern) needs a table in PostgreSQL anyway.
  - Own table with `SKIP LOCKED`: good for learning, but retries, expiry and heartbeat would be our code.
  - No queue (answer inside the HTTP request): the simplest, but it breaks RNF-34. Restarting the container loses the generation, and slow requests tie up the server.
- **Consequences:**
  - Pros: no new service; registering, charging and queuing are all or nothing; the same PostgreSQL locally and in production; one place to protect and to delete data from; changing the queue only means changing its adapter.
  - Cons: the queue adds load to the main database (irrelevant at this volume, watched in topic 9); the `pgboss` schema has its own migrations next to Prisma's; polling every 2 s makes a few extra requests while a generation runs.
  - Changes elsewhere: 0005 D4 (progress by stage on screen) and 0005 D6 with REQUISITOS RF-71, RF-72 and section 8, point 2 (credit per person). Updated on 16/11: monthly credit and invite-only sign-up (0005 D6, RF-01, RF-02, RF-71, RF-72, RF-78).
  - Cost: none.
- **Sources (checked on 16/11/2026):** [pg-boss](https://github.com/timgit/pg-boss), [pg-boss queues and retries](https://pgboss.io/api/queues), [pg-boss workers](https://pgboss.io/api/workers), [pg-boss adapters (Prisma)](https://pgboss.io/api/adapters), [Graphile Worker](https://worker.graphile.org/docs), [Anthropic TypeScript SDK: retries, timeouts and long requests](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/typescript), [PostgreSQL 18: `SKIP LOCKED`](https://www.postgresql.org/docs/18/sql-select.html#SQL-FOR-UPDATE-SHARE), npm registry (`pg-boss` 12.38.2, `graphile-worker` 0.18.0, `bullmq` 6.7.6, `cockatiel` 4.0.0).
