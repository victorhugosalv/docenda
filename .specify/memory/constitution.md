# Docenda Constitution

This file holds only rules that are already decided. Each principle names where it comes from; that source keeps the detail, and this file never copies it. Versions of tools live in the ADRs only.

## Core Principles

### I. The author decides (NON-NEGOTIABLE)
AI output is only ever stored as a `Suggestion`. Only an authenticated action of the plan's owner writes plan content or approves a plan.
Source: [PURPOSE](../../docs/PURPOSE.md); [DOMAIN](../../docs/DOMAIN.md) INV-03, INV-04.

### II. Nothing invented (NON-NEGOTIABLE)
A skill enters a plan only from the BNCC catalog, with the catalog text: a foreign key in the database and a closed list in the AI output schema.
Source: RF-41; DOMAIN INV-02; [AI](../../docs/AI.md).

### III. Privacy by design (NON-NEGOTIABLE)
- The class is described in aggregate: no identifiable student, no diagnosis, no medical report.
- The plan identification and the user's name, e-mail and ID never enter a prompt. Nothing goes to the AI while a privacy alert is unresolved.
- The technical log never holds personal data, a request body, a cookie, an authorization header or an IP.

Source: PURPOSE; DOMAIN INV-09, INV-10; RNF-39; [0010](../../docs/decisions/0010-logs-monitoring-and-incident-response.md) D3.

### IV. Every AI call is limited and recorded
- Each call has a maximum input and output size and records tokens, cost, model and prompt version, without the content.
- Money is an integer in nano-dollars, never a floating-point number.
- No real AI call in tests or in CI: tests use the fake adapter. Evals run by command, with a spend cap.

Source: RF-73, RF-74, RNF-42; DOMAIN INV-12, INV-13; [0005](../../docs/decisions/0005-ai-provider.md), [0008](../../docs/decisions/0008-ai-job-queue-postgresql.md), [0012](../../docs/decisions/0012-transactions-repositories-and-module-layout.md) D8.

### V. Tests first (NON-NEGOTIABLE)
- A test is written and seen failing before the code that makes it pass.
- Every acceptance criterion of a slice in [ACCEPTANCE](../../docs/ACCEPTANCE.md) has a test. A test for a requirement or an invariant starts its name with the ID (`RF-41`, `INV-03`).
- The database is not faked: repositories are tested against the real PostgreSQL. Fakes are for the AI, e-mail, queue and clock ports.

Source: RNF-09; [0009](../../docs/decisions/0009-ci-cd-and-security-checks.md); 0012 D2, D9.

### VI. Boundaries and ownership by construction
- A module is used only through its `index`; `domain/` is pure; `reusable/` never imports `modules/`; only `entrypoints/` imports `providers/`. dependency-cruiser fails the build otherwise.
- Every repository method for an author-owned resource takes `ownerId` first. Another author's resource answers `404`.

Source: [0001](../../docs/decisions/0001-modular-monolith-ports-and-adapters.md); 0012 D1, D4; RNF-40; DOMAIN INV-01.

### VII. Secure by default
- No secret in the repository; configuration comes from validated environment variables.
- The web layer follows ASVS 5.0 level 1. The container runs without root, on a read-only file system.

Source: RNF-32, RNF-41, RNF-44; [0006](../../docs/decisions/0006-web-security-asvs-level-1.md), [0007](../../docs/decisions/0007-docker-and-isolated-environment.md), 0009.

### VIII. Simple and stable
- Stable versions only, pinned as the ADRs say, and at least 7 days old when adopted.
- Every new dependency is justified in its pull request. Unused files, exports and dependencies fail CI.
- Build what the current slice needs, nothing for a later one.

Source: [0002](../../docs/decisions/0002-web-stack.md); 0009 D5; [BACKLOG](../../docs/BACKLOG.md).

## Technical Constraints

The stack is decided in the ADRs, which hold the versions. A plan follows them; a different choice needs a new ADR first.

| Layer | Decision |
|---|---|
| Architecture and module layout | [0001](../../docs/decisions/0001-modular-monolith-ports-and-adapters.md), [0012](../../docs/decisions/0012-transactions-repositories-and-module-layout.md) |
| Back-end, front-end build, ORM, runtime, workspaces | [0002](../../docs/decisions/0002-web-stack.md) |
| Authentication | [0003](../../docs/decisions/0003-authentication.md) |
| Database | [0004](../../docs/decisions/0004-postgresql.md) |
| AI provider | [0005](../../docs/decisions/0005-ai-provider.md) |
| Web security | [0006](../../docs/decisions/0006-web-security-asvs-level-1.md) |
| Containers | [0007](../../docs/decisions/0007-docker-and-isolated-environment.md) |
| Job queue | [0008](../../docs/decisions/0008-ai-job-queue-postgresql.md) |
| CI/CD and security checks | [0009](../../docs/decisions/0009-ci-cd-and-security-checks.md) |
| Logs and monitoring | [0010](../../docs/decisions/0010-logs-monitoring-and-incident-response.md) |
| Hosting | [0011](../../docs/decisions/0011-low-cost-aws.md) |
| Front-end libraries | [0013](../../docs/decisions/0013-front-end-stack.md) |
| This workflow | [0014](../../docs/decisions/0014-spec-driven-workflow-spec-kit.md) |

## Development Workflow

- One slice of the [BACKLOG](../../docs/BACKLOG.md) is one feature: one branch, one pull request. `main` is protected.
- A specification cites requirements and invariants by ID and never copies their text.
- If a slice changes a requirement, a domain rule or a decision, that document changes in the same pull request. A new technical decision is a new ADR.
- Language follows the reader: English for developers and the coding agent; Brazilian Portuguese for what teachers see ([0001](../../docs/decisions/0001-modular-monolith-ports-and-adapters.md), note of 17/11/2026).
- A slice is done when it meets the BACKLOG's definition of done.

## Governance

- Precedence: PURPOSE, then this constitution, then the slice's specification, plan and tasks.
- This file records decisions; it does not make them. A change comes in a pull request approved by the owner, with the source that justifies it.
- Versioning: MAJOR when a principle is removed or redefined, MINOR when one is added, PATCH for wording.

**Version**: 1.0.0 | **Ratified**: 2026-11-17 | **Last Amended**: 2026-11-17
