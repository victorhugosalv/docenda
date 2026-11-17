# Threat map: Docenda

> **v0.1 (17/11/2026). Planned: the system does not exist yet.** Required by decision [0009](decisions/0009-ci-cd-and-security-checks.md) (D10) before publishing; written now to guide the slices. It is reviewed at milestone M3, against the real system.
> Method: **STRIDE**, six questions for each part of the system: can someone pretend to be someone else (*Spoofing*)? Change something (*Tampering*)? Deny what they did (*Repudiation*)? See what they should not (*Information disclosure*)? Take down the service (*Denial of service*)? Gain power they do not have (*Elevation of privilege*)?
> Open risks are described in general terms, without the path to exploit them.

## 1. Data flow

```
                     ┌──────────── boundary 1: internet ─────────────┐
  Author's browser ──HTTPS─▶ Proxy (TLS) ─▶ API (Express) ─▶ PostgreSQL
  Admin's browser ───HTTPS─┘                     │   ▲          ▲  (data, sessions,
                                                 │   │          │   pg-boss queue)
                                      job with IDs only         │
                                                 ▼   │          │
                                               Worker ──────────┘
                                                 │
           boundary 2: outside Brazil ───────────┼──────────▶ Anthropic API
           boundary 3: open internet ────────────┼──────────▶ Source link (F13)
           boundary 4: e-mail ───────────────────┴──────────▶ Mailpit / Amazon SES
  
  GitHub (code, CI) ─▶ GHCR (image) ─▶ Server              Administrator ─SSH─▶ Server
```

| Flow | What passes | Personal data? |
|---|---|---|
| Browser → API | Login, class context, plan, plan identification | Yes: account and identification |
| API → PostgreSQL | Everything that is stored | Yes |
| API → queue → worker | IDs only (0008 D6) | No |
| Worker → Anthropic | Theme, aggregate context, pasted material, plan fields, skills | **Should not:** never identification or account data (INV-09); privacy alert first (INV-10) |
| Worker → source link | The address the author entered | No |
| API → e-mail | Address and invitation, confirmation or password link | Yes: e-mail |
| Administrator → server | SSH access, only from their IP | No |

## 2. Threats, defences and what is missing

Defences cited by the decision or requirement that creates them. "Missing" is what is not covered yet.

| # | STRIDE | Threat | Planned defence | Missing |
|---|---|---|---|---|
| T01 | S | Guessing someone's password or testing passwords leaked from other sites | Argon2id, banned-password list, 3 attempts every 10 s per IP, a single message (0003, 0006 D4) | Per-account limit and 2FA for everyone, after v1 |
| T02 | S | Stealing the session | `HttpOnly`, `SameSite`, `Secure` cookie; server-side session, revocable; 30 min idle (0003, 0006 D1, D2) | — |
| T03 | S | Taking over the administrator account | Created by command (RF-79); 2FA required for administrator routes (0009 D10) | 2FA comes in before publishing |
| T04 | S | Pretending to be another IP to escape the limit | IP read only from the header of our own proxy (0006 D3) | — |
| T05 | T | Using the author's session from another site (CSRF) | `SameSite=Lax` and checking the `Origin` (0006 D5) | — |
| T06 | T | Changing another author's plan by swapping the ID | `ownerId` in every repository; 404 for someone else's resource; a test that walks every route (0012 D4, INV-01) | Row-level security in the database, as a second layer (ROADMAP) |
| T07 | T | Pasted text that makes the model change behaviour (prompt injection) | Text marked as data; tag escaped; output bound to a schema; input and output checks; the AI only creates suggestions (AI.md, 0005 D5) | There is no complete defence against injection; the damage stays limited because the AI does not act on the system |
| T08 | T | The AI writing content directly into the plan or approving | The worker only creates suggestions; approval only through the owner's route (INV-03, INV-04) | — |
| T09 | T | An invented skill or source getting into the plan | Closed list in the schema; foreign key; a source only approved by the author (INV-02, RF-51) | — |
| T10 | T | A compromised dependency or CI action | Lockfile, Dependabot with a 7-day wait, CodeQL, pinned actions, Gitleaks (0009) | Image scan before publishing (ROADMAP, M3) |
| T11 | R | An author or administrator denying what they did | Audit with IDs only, which the application does not change (INV-14, 0010 D6); access records for 6 months | — |
| T12 | I | Student data reaching the AI or the database | Class described as a whole; term list and model check; text with an alert is not stored (RF-15, DATA D6) | The automatic check makes mistakes; a human review of the stored data is a criterion of PURPOSE |
| T13 | I | Identification or account data going to the AI | Separate table; AI port type without these fields; test with decoy values (INV-09, RNF-45) | — |
| T14 | I | Personal data in the technical log | Request body never logged; `redact`; IP out of the log; test with decoy values (0010 D3) | — |
| T15 | I | An error revealing internal details | Problem format with `code` and the request number, no stack trace (0012 D7) | — |
| T16 | I | Using the link check to reach internal addresses (SSRF) | `https` only; address checked after resolving the name; redirects checked; maximum time and size (RNF-43) | Comes in with F13 |
| T17 | I | Leak of the database or of a backup | Password as a hash; cookie signed with a secret outside the database; backup encrypted and with an expiry (0006 D1, 0011) | Encryption at rest and a restore test before publishing |
| T18 | I | A secret in the repository | Gitleaks in CI; `.env` ignored; CI without the AI key (0009) | — |
| T19 | I | AI or author text turning into code on screen (XSS) | Plain text in every field; React escapes; CSP `default-src 'self'` (0006) | — |
| T20 | D | Draining the AI money | Per-person credit, system cap, per-minute and per-day limit, reservation before the call, spending limit at the provider (0005 D6, 0008, INV-06) | — |
| T21 | D | Taking down the API with many requests or huge requests | General limit per IP and per session; body up to 256 kB (RNF-44) | Network protection against a distributed attack is left to the hosting |
| T22 | D | An AI provider outage freezing the system | Circuit breaker; generation in the queue; the rest works without AI (RNF-36, RNF-34) | — |
| T23 | D | A generation stuck forever blocking the plan | Deadline per generation and orphan sweep (INV-08) | — |
| T24 | E | An author reaching an administrator route | Role checked on the route; denied by default; test (0006 D5) | — |
| T25 | E | The application altering its own schema or the audit | Three database roles; the application neither migrates nor deletes audit (0012 D5) | — |
| T26 | E | Escape from the API container | No root, no capabilities, read-only file system, no Docker socket (0007 D2) | — |
| T27 | E | A coding agent with access beyond the project | Dev container with an outbound firewall; cloud sessions without the private files (0007 D3) | Set up the dev container in F00 |

## 3. Risks accepted in v1

| Risk | Why it is accepted |
|---|---|
| Data sent to the AI is processed outside Brazil and kept for up to 30 days by the provider | It is the cost of using the provider; the privacy notice and the generation screen say so (RNF-21). Nothing identifiable should go |
| Database and secret leaked together allow using active sessions | It would take two leaks; changing the secret ends all sessions (0006 D1) |
| No external error collector | Up to 10 people in v1; the three e-mail alerts cover what costs money or loses work (0010 D7) |

## 4. Before publishing (M3)

- Review this map against the built system: each "Missing" becomes a backlog item or an accepted risk with a reason.
- Run the OWASP ZAP scan and a security audit of the code (0009 D10).
- Check ASVS level 1 item by item (RNF-06).
- Run the incident drill in [INCIDENTS](INCIDENTS.md).
