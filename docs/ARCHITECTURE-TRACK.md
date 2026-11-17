# Architecture fundamentals track

Started on 13/11/2026, before Spec Kit. One topic at a time: explanation, good practice with a current source, options for Docenda and Victor's decision. Each decision lives in `docs/decisions/` in the short ADR (architecture decision record) format.

| # | Topic | Status | Decision |
|---|---|---|---|
| 1 | Decoupled architecture: modular monolith, ports and adapters, folder structure and reusable modules (nothing tied to a provider) | decided | [0001](decisions/0001-modular-monolith-ports-and-adapters.md), [0002](decisions/0002-web-stack.md), [0004](decisions/0004-postgresql.md) |
| 2 | Authentication: session × JWT, refresh token, password hashing, transactional e-mail; 2FA and social login later | decided | [0003](decisions/0003-authentication.md) |
| 3 | Web security: OWASP ASVS 5.0 level 1 applied to Docenda | decided | [0006](decisions/0006-web-security-asvs-level-1.md) |
| 4 | AI security and cost: OWASP Top 10 for LLM 2025, token budget, v1 provider and model (no training on our data) | decided | [0005](decisions/0005-ai-provider.md) (models and limits confirmed by the test with the 3 reference plans) |
| 5 | Docker and an isolated environment for Claude: container in WSL2 (preferred), sandbox as an alternative, cloud session; application container without root and with only what is needed | decided | [0007](decisions/0007-docker-and-isolated-environment.md) |
| 6 | CI/CD: tests, security analysis (SAST, dependencies and search for committed secrets) and a protected branch; a one-page threat map (STRIDE); 2FA only for the administrator in v1; OWASP ZAP scan (DAST) against the container | decided | [0009](decisions/0009-ci-cd-and-security-checks.md) |
| 7 | AWS at low cost (if time runs short, the local demo is enough) | decided | [0011](decisions/0011-low-cost-aws.md) (v1 local; later Lightsail in São Paulo, under US$ 20 a month) |
| 8 | Performance and scale: AI generation queue in PostgreSQL itself (pg-boss, `SKIP LOCKED`), layered retries, double click, worker concurrency and Redis only when there is a measured reason | decided | [0008](decisions/0008-ai-job-queue-postgresql.md) |
| 9 | Logs and monitoring; incident plan (rotate secrets, end sessions, notify the ANPD and the data subjects, LGPD art. 48, deadline to be checked) | decided | [0010](decisions/0010-logs-monitoring-and-incident-response.md); plan in [INCIDENTS](INCIDENTS.md) (3 business days, CD/ANPD Resolution 15/2024) |

## Planning review (16/11)
With the track finished, an independent review of requirements and decisions pointed out what was missing before code. It produced three decisions and dated notes on 0001, 0002, 0005, 0006 and 0008.

| # | Topic | Decision |
|---|---|---|
| 10 | Module template, transaction across modules, repositories without a fake database, database roles, error format | [0012](decisions/0012-transactions-repositories-and-module-layout.md) |
| 11 | Front-end: routes, server data, forms, accessibility | [0013](decisions/0013-front-end-stack.md) |
| 12 | Spec-driven development method (Spec Kit) | [0014](decisions/0014-spec-driven-workflow-spec-kit.md) |

The same review produced [DOMAIN](DOMAIN.md), [AI](AI.md), the [BACKLOG](BACKLOG.md), the [ROADMAP](ROADMAP.md), the [INTERFACE](INTERFACE.md) and REQUIREMENTS v0.3.

On 17/11, five packages detailed what the code needs to start without deciding in the dark: [DATA](DATA.md) (tables and roles), [CONTRACTS](CONTRACTS.md) (API), [ACCEPTANCE](ACCEPTANCE.md) (criteria per slice), [EVALUATION](EVALUATION.md) with the prompt drafts and the test sets, [DESIGN](DESIGN.md) and the [threat map](THREATS.md), brought forward from milestone M3.

## Execution order (15/11)
The numbering above is still the whole track. The working order, trimmed for the first version, is **5 → 8 → 6 → 9 → 7**:
1. **5. Docker:** `docker compose` with PostgreSQL and Mailpit, so the system starts with one command; container without root. The isolated environment for Claude along with it.
2. **8. Queue:** choose the AI generation queue (RNF-34), preferably on PostgreSQL itself; the rest is already in RNF-33 to 37.
3. **6. CI:** from the first code on, types, tests and dependency alerts, with work on branches and pull requests to the protected main. Before publishing: ZAP, threat map and admin 2FA.
4. **9. Logs:** a simple logger from the start; monitoring, alerts and incident plan before publishing.
5. **7. AWS:** a short note: local demo in v1, AWS later.

## Open points of REQUIREMENTS (section 8) linked to the track
| Point | Topic that closes it |
|---|---|
| 1. v1 date | Reassess at the end of the track |
| 2. Usage limits and monthly cap | 4: initial values in [0005](decisions/0005-ai-provider.md); final ones after the test |
| 3. AI provider and model; e-mail provider | Closed: AI in [0005](decisions/0005-ai-provider.md); e-mail in [0003](decisions/0003-authentication.md) |
| 4. Access records for 6 months | 9 |
| 5. Cooling-off period on deletion | Outside the track (requirements), unless the architecture calls for it |
| 6. Export format (PDF or DOCX) | Outside the track (requirements) |
| 7. Retention of inactive accounts | Outside the track (requirements) |
