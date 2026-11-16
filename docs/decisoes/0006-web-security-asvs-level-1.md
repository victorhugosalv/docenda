# 0006: Web security (OWASP ASVS 5.0 level 1)

- **Date and status:** 15/11/2026, approved by Victor.
- **Context:** RNF-06 asks for ASVS 5.0 level 1 before publishing. Decision 0003 left four points to check: hashed session tokens, cookie names per environment, `trust proxy` and rate limiting. They were checked in the Better Auth 1.7.25 source code and docs on 14/11/2026. A short first deadline asks for a lean result: configuration first, own code only where needed.
- **Decision:**
  - **D1. Session tokens are stored as is, not hashed.** Better Auth saves a random 32-character token and looks it up directly, with no option to hash it. The cookie carries `token.signature`, an HMAC of the token with `BETTER_AUTH_SECRET`, and the server checks it before reading the database. So a leaked database alone does not give a usable cookie. ASVS 5.0 level 1 (V7: 7.2.1 to 7.2.4, 7.4.1, 7.4.2) does not ask for hashing, and Better Auth meets those items. Rules for the secret:
    - only in an environment variable, never in code, database or backups;
    - one per environment (local, test, production), random, at least 32 bytes (`openssl rand -base64 32`);
    - if it leaks, replace it: every session ends and everyone signs in again.
  - **D2. Cookies:** the default `__Secure-` prefix, which Better Auth adds on HTTPS (`better-auth.session_token` locally, `__Secure-better-auth.session_token` in production). `__Host-` cannot be set through the options (a custom name still gets `__Secure-` in front), and it would add nothing here: no subdomains, `Path=/`, no `Domain`. `useSecureCookies` on in production; `localhost` out of `trustedOrigins` in production.
  - **D3. Client IP:** read only from the header our own proxy writes (`advanced.ipAddress.ipAddressHeaders` in Better Auth, `trust proxy` set to 1 in Express). Locally, without a proxy, the header list is empty. `X-Forwarded-For` from the client is never trusted. This works the same with Caddy or the AWS load balancer; the choice belongs to topic 7.
  - **D4. Brute force:** no account lockout in v1. Controls:
    - Better Auth rate limit per IP: 3 sign-in attempts every 10 s (built in), using the IP from D3;
    - Argon2id and the password blocklist (0003, D3) make each guess expensive and cut the easy ones;
    - one message for every failure: "E-mail ou senha incorretos", with no attempt counter and no hint that the account exists or is limited.

    Account lockout was dropped because anyone who knows an e-mail could lock its owner out (ASVS 6.1.1 asks to prevent malicious lockout), it would store e-mails of people who are not users (LGPD), and it adds code and tests. ASVS 5.0 6.3.1 (level 1) asks for documented controls against brute force and credential stuffing, not lockout. **After v1:** a per-account limit with growing waits, below the NIST SP 800-63B-4 ceiling of 100 consecutive failures, together with 2FA (0003, D5).
  - **D5. Other level 1 items**, checked before publishing (RNF-06):

    | ASVS chapter | How Docenda meets it |
    |---|---|
    | Validation (V1, V2) | Zod 4 schemas at the HTTP edge (`packages/contracts`); Prisma queries only, never raw SQL built from strings |
    | Front-end (V3) | helmet 8 headers: CSP `default-src 'self'`, HSTS, `nosniff`, `frame-ancestors 'none'`. React escapes text by default; AI or user text shown as Markdown is sanitized, never injected as raw HTML |
    | API (V4) | JSON only; for state-changing routes, `SameSite=Lax` plus an `Origin` check against the app URL (CSRF) |
    | Files (V5) | No upload in v1. When RF-22 arrives: size and type limits, and the file is deleted after extraction |
    | Authorization (V8) | Deny by default; owner check on every resource and non-sequential IDs (RNF-40) |
    | Crypto (V11) | No own crypto: Argon2id, CSPRNG from the platform |
    | HTTPS (V12) | TLS at the proxy, HSTS |
    | Configuration (V13) | Secrets in environment variables (RNF-32); no stack trace or debug mode in production |
    | Data (V14) | No personal data or tokens in URLs; `Cache-Control: no-store` on API responses |
    | Logs and errors (V16) | Generic error messages; logs without password, token or sensitive content (RNF-39, topic 9) |

    helmet 8 and Zod 4 come from 0002 (checked on 14/11/2026).
- **Scaling path:** the rate limit keeps its counters in memory by default. With one container (v1) that is enough. With two or more API copies, each copy would count on its own, so the real limit would multiply by the number of copies, which goes against RNF-33. Then the counters move to the database (`rateLimit.storage: "database"`) or to Redis (`"secondary-storage"`), only by configuration, inside `reusable/auth`.
- **Alternatives considered:**
  - Hash the session token ourselves: needs changes to how Better Auth finds sessions, for little gain once the cookie is signed.
  - Account lockout (5 failures, 15 min, as decided on 14/11): see D4.
  - `__Host-` prefix: not available through the options; same effect here as `__Secure-`.
- **Consequences:**
  - Pros: almost everything is configuration; brute force protection without our own code; nothing stored about e-mails that are not accounts.
  - Cons: if the database and the secret leak together, active sessions can be used until they expire or the secret is replaced; a distributed attack (many IPs) is only slowed by Argon2id and the blocklist until the per-account limit arrives.
  - Changes in REQUISITOS: RF-03, F1 scenario 2, REQ-15 and REQ-22.
- **Sources:** [ASVS 5.0 V6](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x15-V6-Authentication.md), [ASVS 5.0 V7](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x16-V7-Session-Management.md), [NIST SP 800-63B-4](https://pages.nist.gov/800-63-4/sp800-63b.html), [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html), Better Auth docs ([rate limit](https://www.better-auth.com/docs/concepts/rate-limit), [security](https://www.better-auth.com/docs/reference/security), [cookies](https://www.better-auth.com/docs/concepts/cookies)) and the `better-auth@1.7.25` package source.
