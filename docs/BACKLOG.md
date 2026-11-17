# Backlog: Docenda

> **v0.1 (16/11/2026).** Vertical slices, in execution order. Each slice is one Spec Kit feature (decision [0014](decisions/0014-spec-driven-workflow-spec-kit.md)): a folder in `specs/`, a branch and a pull request.
> Milestones: M0 to M2 make up the **v1** of [REQUIREMENTS](REQUIREMENTS.md). M3 onwards is in the [ROADMAP](ROADMAP.md).
> What each slice must prove to be done is in [ACCEPTANCE](ACCEPTANCE.md); the tables, in [DATA](DATA.md); the routes, in [CONTRACTS](CONTRACTS.md).

## Milestones

| Milestone | What it delivers | Slices |
|---|---|---|
| **M0 Foundation** | The repository starts with one command and CI passes | F00 |
| **M1 Demonstrable core** | An author logs in, describes the class, generates a plan with real skills, decides each suggestion, approves and exports | F01 to F08 |
| **M2 v1 for invited users** | Invitations, credit, protection against misuse, administration, audit and account deletion | F09 to F15 |

## Slices

| # | Slice | Delivers | Areas | Requirements | Milestone | Status |
|---|---|---|---|---|---|---|
| F00 | `foundation` | Workspaces, TypeScript, lint, boundaries, Compose (PostgreSQL, Mailpit, migration), database roles, `/healthz` and `/readyz`, logging, validated configuration, error format, CI, Dockerfile | infra, back | RNF-09, RNF-11, RNF-32, RNF-41 | M0 | to do |
| F01 | `bncc-catalog` | Catalog imported, checked and versioned; query by education stage and grade; validation of code and stage | database, back | RF-40, RF-41, RF-43 | M1 | to do |
| F02 | `plan-core` | Pure rules: class-hour conversion, effective time, sum of times, plan states, suggestions, findings and the approval condition | back (domain) | RF-12, RF-13, RF-14, RF-23, RF-25, RF-27, section 4 | M1 | to do |
| F03 | `ai-generation` | AI port, fake adapter and Anthropic adapter; output schema with skills in a closed list; generation pipeline; usage and cost record; a command to generate from a file | AI, back | RF-20, RF-21, RF-42, RF-73, RF-74, RNF-35, RNF-38 | M1 | to do |
| F04 | `evals-min` | Evaluation command with the deterministic layer, synthetic set and a cost and time table; confirmation test of 0005 (D7) | AI | RNF-10, RNF-14 | M1 | to do |
| F05 | `auth-sessions` | Better Auth with server-side sessions, Argon2id, per-IP limit, account creation by command, owner check | back, database | RF-03, RF-04, RF-79, RNF-07, RNF-40 | M1 | to do |
| F06 | `plans-api` | Class context and plan saved; list, open, edit, duplicate and delete; decide suggestions; approve; versions | back, database | RF-11, RF-16, RF-17, RF-24, RF-25, RF-32 | M1 | to do |
| F07 | `generation-queue` | pg-boss and worker; generation in a transaction with a reservation against the system cap (the ledger tables are born here); progress by step; double click; closing orphaned generations; cap and on/off switch by configuration | back, database, infra | RF-70, RF-72, RF-75, RNF-33, RNF-34, RNF-36, RNF-37 | M1 | to do |
| F08 | `web-core` | Screens: log in, plan list, step-by-step new plan, editor with suggestions and findings, Markdown export | front | RF-60, RF-61, RNF-02, RNF-03, RNF-04, RNF-05; scenarios F1 to F4 and F6 | M1 | to do |
| F09 | `guardrails` | `input_check` and `output_check`; privacy alert on screen; misuse occurrences; explained refusal | AI, back, front | RF-15, RF-90, RF-91, RF-92, RF-95, RNF-08 | M2 | to do |
| F10 | `invites-email` | Invitation by e-mail, sign-up by invitation only, versioned terms, password recovery, e-mail change | back, front | RF-01, RF-02, RF-05, RF-06, RF-78 | M2 | to do |
| F11 | `ai-credit` | Monthly per-person credit on top of the F07 ledger, limits per day and per minute, balance on screen | back, front | RF-71, RNF-42 | M2 | to do |
| F12 | `admin` | Panel: invitations, usage and cost, occurrences and block review, settings; audit, access records and the three e-mail alerts; the precautionary reset command (INCIDENTS, step 1) | back, front | RF-93, RF-94, RNF-39; decision 0010 | M2 | to do |
| F13 | `sources` | Source brought by the author: domain list, safe link check, "to review" state, ABNT reference | back, front | RF-51, RF-52, RF-53, RF-54, RNF-43 | M2 | to do |
| F14 | `account-lifecycle` | Deletion request in 2 steps with a 30-day period, cancel and delete-now links by e-mail, inactive accounts, retention and cleanup jobs | back, front | RF-07; section 7.1 | M2 | to do |
| F15 | `evaluate-plan` | Plan evaluation: rules and rubric, with findings and suggestions | AI, back, front | RF-26 | M2 | to do |
| F16 | `quality-gate` | Accessibility review with a screen reader, e2e scenarios F1 to F8, full evaluation, README with a demo | all | RNF-01, RNF-03, RNF-09, RNF-10 | M2 | to do |

**If time gets tight, these go first:** F15, then F14 (simple deletion stays, without inactive accounts) and F13 (the plan ships without a bibliography checked by the system). F09 and F11 do not go: without them, nobody from outside should use the AI.

## Order and reason

1. **Rules and AI before screens (F01 to F04).** Docenda's value lies in "nothing invented" and "it fits the time". These parts are pure functions and a pipeline testable by command: they can be built and measured with no screen or login.
2. **Login and persistence (F05, F06)** come when there is something to store.
3. **Queue (F07) separate from the pipeline (F03).** The pipeline does not know who called it; the queue only carries it.
4. **Screens (F08)** close M1 with a complete flow.
5. **M2** adds what protects the system when other people come in.

## Suggested pace (an estimate, not a promise)

| Day | Slices |
|---|---|
| 17/11 | Planning review in a PR; F00 |
| 18/11 | F01, F02 |
| 19/11 | F03, F04 |
| 20/11 | F05, F06 |
| 21/11 | F07, start of F08 |
| 22/11 | F08, F09 |
| 23/11 | F10, F11 |
| 24/11 | F12, F16 and whatever fits of F13 to F15 |

Two slices a day only work with same-day review. If a slice overruns, the cut follows the list above; the deadline is not stretched by cutting tests.

## Definition of done (every slice)

- [ ] The specification in `specs/NNN-<slice>/` cites requirements by ID, without copying the text.
- [ ] Every criterion of the slice in [ACCEPTANCE](ACCEPTANCE.md) has a passing test.
- [ ] Tests were written before the code; each rule test carries the requirement or invariant ID in its name.
- [ ] CI passes: types, lint, boundaries, tests and image build.
- [ ] No real AI call in tests; no secret in the code; no personal data in logs.
- [ ] If the slice has a screen: usable at 360 px, navigable by keyboard alone and with no axe errors.
- [ ] If the slice changed a requirement, a domain rule or a decision, the document was updated in the same PR.
- [ ] A small pull request, with what was done, how to test it and what was left out.

## Input of each slice for Spec Kit

The text below is the starting point for `/speckit-specify`. The technical plan (`/speckit-plan`) follows the decisions already made and only creates a new decision through an ADR.

- **F00 `foundation`.** Set up the repository so that `docker compose up` starts the database, test e-mail, migration, API and worker, and so that each PR runs types, lint, boundaries, tests and build. No product feature. Decisions: 0001, 0002, 0007, 0009, 0010 (D2, D7), 0012.
- **F01 `bncc-catalog`.** Import the 140 skills of the BNCC Computing supplement once, from the official document, with an automatic text check and a sample checked by hand. Query by education stage and grade. Refuse a non-existent code and warn when the skill does not match the class.
- **F02 `plan-core`.** Every rule in section 4 of REQUIREMENTS as pure functions, plus the plan and suggestion states and the approval condition (DOMAIN, sections 4, 5 and 7). No database and no AI.
- **F03 `ai-generation`.** Generate a plan's suggestions from theme, context and pasted material, along the path in section 3 of AI.md, with the provider behind the port. Deliver a command that generates from a context file and shows the cost.
- **F04 `evals-min`.** Run a set of contexts through the pipeline and produce the deterministic-layer table, with cost and time. Run the confirmation test of 0005.
- **F05 `auth-sessions`.** Log in and out with e-mail and password, with a server-side session and a single error message. Accounts created by command until F10. Every resource checks the owner.
- **F06 `plans-api`.** Store context and plan; list, open, edit, duplicate and delete; accept, edit or reject each suggestion; approve and store the version.
- **F07 `generation-queue`.** Requesting a generation immediately returns a number; the work runs in the worker; the screen follows the steps. Record, reserve and enqueue in a single transaction. No generation gets stuck.
- **F08 `web-core`.** The screens of the main flow, on a phone and with a screen reader (INTERFACE, screen map).
- **F09 to F16.** Specified when M1 closes, with whatever real use of M1 teaches.
