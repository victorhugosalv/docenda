# 0004: PostgreSQL

- **Date and status:** decided by Victor on 14/11/2026, written on 15/11/2026.
- **Context:** 0002 chose Prisma 7 and left the database open. Docenda's data is relational: accounts and sessions (Better Auth), plans with states and versions, the BNCC catalog, sources, AI usage records, misuse occurrences and the audit log. The server keeps no state in memory between requests (RNF-33), so everything shared lives in the database, and 0006 may move the rate limit counters there. Money is stored as integers (RNF-42) and IDs in URLs are not sequential (RNF-40).
- **Decision:**
  - **PostgreSQL 18** (18.7 on 15/11/2026, supported until 14/11/2030). PostgreSQL 19 is still a release candidate, and 0001 asks for stable versions only.
  - Locally it runs as a Docker container (topic 5). Where it runs in production (managed service or container), and encryption at rest and backups (REQ-20, RNF-24), belong to topic 7.
  - Data that is queried or checked stays in columns. `jsonb` only for plan parts whose shape is still changing.
  - IDs are UUID v7 (PostgreSQL 18 has `uuidv7()` built in): not guessable in sequence, and ordered by time, which helps indexes. Costs and token counts are `bigint`.
- **Alternatives considered:**
  - SQLite: no server to run, but one file on disk does not serve two API copies (RNF-33), and local would differ from production.
  - MySQL 8: works with Prisma, but brings no gain here, and `jsonb` and native UUID v7 are stronger in PostgreSQL.
  - MongoDB: the data has many relations (plan, version, skill, source, owner), which a relational database handles better.
- **Consequences:**
  - Pros: the most common pairing with Prisma, plenty of material, and the same engine locally and in production.
  - Cons: one more container to run locally; backups and encryption still to plan in topic 7.
  - Cost: none locally; the production cost is decided in topic 7.
- **Sources:** [PostgreSQL versioning policy](https://www.postgresql.org/support/versioning/) (checked on 15/11/2026), [PostgreSQL 18 UUID functions](https://www.postgresql.org/docs/18/functions-uuid.html).
