# 0003: Authentication

- **Date and status:** 14/11/2026, approved by Victor.
- **Context:** Login is required in v1 and also protects the AI routes. Docenda is a modular monolith (0001) with an Express API and a SPA on the same domain (0002). A suspended or deleted account must lose access immediately (misuse rules, LGPD). OWASP advises against JWT for user sessions unless it is really needed ([JWT cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html)).
- **Decision:**
  - **D1. Server-side sessions** stored in the database, sent in an `HttpOnly`, `SameSite` cookie (`Secure` and a host-bound prefix when served over HTTPS). Session ID renewed at login; revocable (logout, password change, "sign out of all devices", suspension, account deletion). Idle timeout of 30 min (RF-04), extended at most every few minutes to avoid a write on every request. No JWT or refresh token for users.
  - **D2. Better Auth** (1.7.x, MIT) as the adapter inside `reusable/auth`, with the Prisma adapter. The rest of Docenda only sees the module's public entry point ("who is the user of this request?", "protect this route").
  - **D3. Passwords:** Argon2id with OWASP parameters (19 MiB, t=2, p=1) through `@node-rs/argon2`, replacing the default scrypt. Minimum 8, maximum 128 characters; no composition rules; a small blocklist (common passwords, sequences, repeated characters, name, e-mail, "docenda"); no periodic change ([OWASP Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html); [ASVS 5.0 V6](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x15-V6-Authentication.md)).
  - **D4. Transactional e-mail:** Mailpit locally and in the demo; fake adapter in tests; Amazon SES in production (topic 7), sent through the job queue (topic 8).
  - **D5. After v1:** passkeys, then TOTP, then Google login, through Better Auth plugins. E-mail codes are never a second factor ([NIST SP 800-63B-4](https://pages.nist.gov/800-63-4/sp800-63b.html)).
- **Scaling path (no rewrite):** sessions in the database keep the API stateless (RNF-33), so more containers work as is. If session lookups become a bottleneck: Better Auth secondary storage (e.g. Redis) or its short signed cookie cache. If other services or partners need identity: Better Auth JWT plugin, which issues JWTs for services and *"is not meant as a replacement for the session"* ([docs](https://www.better-auth.com/docs/plugins/jwt)). All of this stays inside `reusable/auth`.
- **Alternatives considered:**
  - JWT with refresh token for users: avoids a lookup per request, but revocation needs a database check anyway, plus two tokens, rotation and a refresh flow.
  - Own implementation following the Lucia guide: fewer dependencies and full control, but more work now and protocol work later (OAuth, passkeys) by hand.
  - Managed service (Cognito, Auth0, Clerk): vendor lock-in (RNF-30), personal data outside our control, cost per user.
- **Consequences:**
  - Pros: revocation is immediate and simple; ready flows for e-mail verification, password reset and future 2FA; hashes stay portable because we choose Argon2id.
  - Cons: Better Auth has 17 direct dependencies, so it adds supply chain risk (mitigated by lockfile, alerts and audit in topic 6); it owns its tables and routes.
  - To check in topic 3: whether session tokens are stored hashed, cookie names per environment, `trust proxy` behind the AWS load balancer, rate limiting.
