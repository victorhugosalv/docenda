# Long-term roadmap

> **v0.1 (16/11/2026).** What comes after v1 and what would make each decision change. v1 (M0 to M2) is in the [BACKLOG](BACKLOG.md). Nothing here is a promised date: it is the order and the reason.

## Milestones

| Milestone | Delivers | Done when |
|---|---|---|
| M0 to M2 | v1: see the [BACKLOG](BACKLOG.md) | Scenarios F1 to F8 of REQUIREMENTS pass as tests |
| **M3 Publication** | v1 live for the invited users | The "before publishing" list below is complete |
| **M4 Product** | What [PURPOSE](PURPOSE.md) promises beyond the plan for one lesson | A student teacher goes through the full cycle with the advisor inside the system |
| **M5 Scale and sustainability** | The system pays for itself and can grow | Cost per plan known and a pricing model tested |

### M3 Publication: the "before publishing" list
It gathers what the decisions have already left for this moment.

| Item | Origin |
|---|---|
| Threat map (STRIDE): v0.1 of [THREATS](THREATS.md) was written on 17/11; review it against the built system and resolve each "Missing" | 0009 D10 |
| Administrator 2FA | 0009 D10 |
| OWASP ZAP scan and a security audit of the code | 0009 D10 |
| ASVS level 1 checklist checked item by item | RNF-06 |
| Versioned terms of use and privacy notice; legal basis of each processing checked; contact channel for the data subject | REQUIREMENTS, section 8 |
| Lightsail server in São Paulo, Caddy, SES out of the sandbox | 0011, 0003 |
| Daily backup, an off-server copy and **a restore test** | 0011 D4 |
| Incident drill, with the time of each step | 0010 D8 |
| Vulnerability scan of the Docker image | new |
| Final values for credit, limits and cap, after a minimal load test | REQUIREMENTS, section 8 |
| Usability test with a teacher or student teacher (15-minute target) | RNF-01 |

### M4 Product
| Block | Requirements | Note |
|---|---|---|
| Reviewers: invite, comment, grade, remove access | RF-80 to RF-85 | Brings the "in review" state and the improvement consent (RF-86) |
| Other production steps: simplified script, slides, standardized script, study guide | RF-29 | Each one only after the previous one is approved |
| ENADE templates and the length and deadline rules | RF-31; section 4 | Comes in as a new template, not new code (DOMAIN, section 8) |
| Adapting a plan to an unexpected event; regenerating a field with an instruction | RF-28, RF-33 | Uses the stored versions |
| Sources suggested by the AI, with a real search, and checking the fact in the source | RF-50, RF-55 | AI.md, section 8 |
| File upload and export to PDF and DOCX | RF-22, RF-60 | File checks from ASVS V5 |
| Passkeys, authenticator code, Google login | RF-09, RF-10 | Better Auth plugins (0003 D5) |
| Downloading one's own data | RF-08 | |
| Plans for a week, month and semester | RF-30 | Depends on versions and the catalog |

### M5 Scale and sustainability
| Block | Note |
|---|---|
| Cost per plan and subscription study | The usage data already allows it (RNF-14) |
| Payment | Its own port, like the other external services (RNF-30) |
| The user's own API key; free models | RF-76, RF-77 |
| Schools and teams | An organization column on resources; the owner already exists everywhere |
| Other BNCC components | New data in the generic catalog |
| Semantic search in the catalog | Only when the catalog no longer fits in the request; then, `pgvector` in PostgreSQL itself |

## Signals for changing the architecture

Each change has a measured signal. Without the signal, the change does not happen.

| Today | Signal | Later | Where it is decided |
|---|---|---|---|
| One API container | More than one copy is needed | Limit counters in the database or in Redis | 0006 |
| Queue in PostgreSQL, no Redis | A cache with a measured gain, counters weighing on the database or a real-time screen | Redis for cache and counters; the queue stays in PostgreSQL | 0008 D8 |
| The screen polls progress every 2 s | Many simultaneous generations or phone battery | Server push (SSE) | 0008 D3 |
| Global lock on the month's cap | A waiting line at the reservation | An atomic counter per period, without a row lock | DOMAIN, section 6 |
| Worker with 2 tasks at a time | The queue growing while budget is left | More workers; the real limit becomes the provider's rate | 0008 D7 |
| Lightsail with everything on one server | More than one copy, high availability or the database weighing | ECS with RDS, same image | 0011 D6 |
| Logs to standard output and 3 alerts | Errors nobody sees, or more than one server | Tracing and metrics (OpenTelemetry) and an error collector | 0010 D7 |
| Owner check in the code | An authorization error in production, or many new modules | Row-level security in the database (RLS), as a second layer | 0012 |
| ASVS level 1 | More sensitive data or a client that demands it | Level 2 items, chapter by chapter | 0006 |
| One AI provider | Outage, price or a requirement to keep data in Brazil | A second adapter (Bedrock or another), after the evaluation | 0005 |
| Reusable folders inside the API | A second live project needs the same module | A package in the workspace | 0001 |

## What does not change

Whatever the milestone, the guarantees of [PURPOSE](PURPOSE.md) and the invariants of [DOMAIN](DOMAIN.md) hold: the author decides, nothing invented, no identifiable student and AI use within limits.
