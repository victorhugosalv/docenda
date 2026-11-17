# Docenda

A web platform for basic-education Computing teachers and student teachers on internship to **create, evaluate and script lesson plans** aligned with BNCC Computing (the computing part of Brazil's national curriculum). The plan structure is fixed; the AI only adapts the content to the class context (prior knowledge, difficulties, social context and accessibility). The interface is in Brazilian Portuguese.

> **Status (17/11/2026):** requirements, domain, architecture, data, API contracts, acceptance criteria, design, AI evaluation and threat map are planned; the code starts with the foundation ([backlog](docs/BACKLOG.md)). **There is no code yet.** The technologies below are planned and each choice is recorded in an architecture decision record (ADR).

## The problem
Putting together a lesson plan aligned with the BNCC and adapted to the real class takes time. Ready-made templates are generic, and ordinary AIs invent skills and sources.

## What Docenda guarantees
- **The teacher decides:** the AI suggests, the author accepts, edits or rejects, and the system shows what came from the AI.
- **Nothing invented:** skills only from the official BNCC Computing catalog; sources only real ones approved by the teacher.
- **No identifiable student:** the class is described in aggregate, with no name, diagnosis or medical report (LGPD, Brazil's data protection law).
- **Controlled AI use:** authenticated users only, with a per-person limit and a spending cap.

## Planned architecture
| Topic | Decision |
|---|---|
| Structure | Modular monolith with ports and adapters; nothing tied to a vendor ([0001](docs/decisions/0001-modular-monolith-ports-and-adapters.md)) |
| Stack | Node.js 26 LTS with TypeScript, Express 5, React + Vite, Prisma 7 ([0002](docs/decisions/0002-web-stack.md)) |
| Authentication | Server-side sessions with Better Auth ([0003](docs/decisions/0003-authentication.md)) |
| Database | PostgreSQL 18, UUID v7 IDs ([0004](docs/decisions/0004-postgresql.md)) |
| AI | Anthropic API, no training on the data; guardrails from the OWASP Top 10 for LLM 2025 and a token budget ([0005](docs/decisions/0005-ai-provider.md)) |
| Web security | OWASP ASVS 5.0 level 1 ([0006](docs/decisions/0006-web-security-asvs-level-1.md)) |
| Environment | Docker Compose, container without root and an isolated environment for the coding agent ([0007](docs/decisions/0007-docker-and-isolated-environment.md)) |
| Queue | AI generation in a queue on PostgreSQL itself (pg-boss), separate from the screens ([0008](docs/decisions/0008-ai-job-queue-postgresql.md)) |
| CI/CD | Types, tests, CodeQL, Dependabot and secret scanning on every pull request; image published by commit hash; threat map (STRIDE), administrator 2FA and OWASP ZAP before publishing ([0009](docs/decisions/0009-ci-cd-and-security-checks.md)) |
| Logs and incidents | JSON logs without personal data, separate access records and audit, alerts to the administrator and an incident response plan ([0010](docs/decisions/0010-logs-monitoring-and-incident-response.md)) |
| Transactions and modules | A module folder template, a transaction shared across modules, the database tested for real and separate roles in PostgreSQL ([0012](docs/decisions/0012-transactions-repositories-and-module-layout.md)) |
| Front-end | React Router, TanStack Query, React Hook Form and accessible components; WCAG 2.2 AA checked on every screen ([0013](docs/decisions/0013-front-end-stack.md)) |
| Method | One specification per slice with Spec Kit ([0014](docs/decisions/0014-spec-driven-workflow-spec-kit.md)) |
| Deployment | v1 as a local demo; later, a Lightsail server in São Paulo with the same Compose, automatic HTTPS and 7-day backups, under US$ 20 a month ([0011](docs/decisions/0011-low-cost-aws.md)) |

Architecture track finished on 16/11/2026 ([track](docs/ARCHITECTURE-TRACK.md)) and reviewed the same day. The next step is code, one slice at a time.

## How the project is built
One architecture topic at a time: concept, good practice with an official source checked on the day, two or three options with their cost and a decision recorded in an ADR. Development uses Claude Code as an assistant, with a human review of every decision.

## Documents
- [Purpose](docs/PURPOSE.md): problem, audience, guarantees and what the product does not do.
- [Requirements](docs/REQUIREMENTS.md): functional and non-functional requirements, LGPD and open points.
- [Domain](docs/DOMAIN.md): entities, states, invariants and the Portuguese–code glossary.
- [AI](docs/AI.md): the path of a generation, output schema and prompts; [evaluation](docs/EVALUATION.md): test cases, rubric and targets.
- [Data](docs/DATA.md): tables, database roles and what happens on deletion.
- [API contracts](docs/CONTRACTS.md): formats, errors and limits of each route.
- [Backlog](docs/BACKLOG.md), [acceptance criteria](docs/ACCEPTANCE.md) and [long-term roadmap](docs/ROADMAP.md): milestones, slices, how to know each one is done and the signals for changing the architecture.
- [Interface](docs/INTERFACE.md) and [design](docs/DESIGN.md): map of screens and routes; colours, type and screen behaviour.
- [Threat map](docs/THREATS.md): data flow and defences, using the STRIDE method.
- [Architecture track](docs/ARCHITECTURE-TRACK.md) and [decisions (ADRs)](docs/decisions/).
- [Incident response plan](docs/INCIDENTS.md).

## Author
Victor Hugo Salviano, Computer Science teaching-degree (licenciatura) student at UFPB. [LinkedIn](https://www.linkedin.com/in/victor-hugo-salviano).

© 2026 Victor Hugo Salviano. All rights reserved.
