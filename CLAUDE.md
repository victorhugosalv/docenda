# Docenda: instructions for the coding agent

Docenda is a web platform that helps teachers create, evaluate and script Computing lesson plans aligned with the Brazilian BNCC. The plan structure is fixed; the AI only adapts the content to the class context. **Status: planning done (17/11/2026), code starting.** Build one slice at a time, in the order of `docs/BACKLOG.md`.

## Where the truth lives

| Question | Document |
|---|---|
| Why the product exists and what it must never do | `docs/PURPOSE.md` (protected: do not edit without the owner's approval) |
| What the system must do | `docs/REQUIREMENTS.md` (requirement IDs `RF-..`, `RNF-..`) |
| Entities, states, invariants and the Portuguese-to-code glossary | `docs/DOMAIN.md` |
| The AI pipeline, prompts and evals | `docs/AI.md` |
| What to build next and the definition of done | `docs/BACKLOG.md` |
| What each slice must prove (acceptance criteria, one test each) | `docs/ACCEPTANCE.md` |
| Tables, constraints, database roles and what happens on deletion | `docs/DATA.md` |
| Request and response shapes, error codes and size limits | `docs/CONTRACTS.md` |
| Map of screens and API routes | `docs/INTERFACE.md` |
| Colors, type, components and screen behaviour | `docs/DESIGN.md` |
| How AI output is evaluated, and the datasets | `docs/EVALUATION.md`, `evals/datasets/` |
| Threats and where each one is handled | `docs/THREATS.md` |
| How and why each technical choice was made | `docs/decisions/` (ADRs 0001 to 0014) |

Read the documents a slice cites before writing code. If a slice changes a requirement, a domain rule or a decision, update the document in the same pull request. A new technical decision is a new ADR, not a comment in the code.

## Rules that are never broken

1. **The author decides.** AI output is only ever stored as a `Suggestion`. Nothing but an authenticated action of the owner writes plan content or approves a plan.
2. **Nothing invented.** Skills come from the catalog (foreign key, closed list in the output schema).
3. **No identifiable student, no personal data to the AI.** The plan identification, and the user's name, e-mail and ID, never enter a prompt. Nothing is sent while a privacy alert is unresolved.
4. **Every AI call is limited and recorded:** tokens, cost, model and prompt version. Money is an integer in nano-dollars.
5. **No real AI call in tests or in CI.** Use the fake adapter. Evals run by command, with a spend cap.
6. **No secret in the repository,** and no personal data, request body or IP in the technical log.
7. **Ownership by construction:** every repository method for an author-owned resource takes `ownerId` first; another author's resource is a `404`.
8. **Module boundaries** (ADRs 0001 and 0012): a module is used only through its `index`; `domain/` is pure; `reusable/` never imports `modules/`; only `entrypoints/` imports `providers/`. dependency-cruiser fails the build otherwise.

## How to work

- **Tests first.** Unit tests (`*.test.ts`) beside the code, integration tests (`*.int.test.ts`) against PostgreSQL, end-to-end tests in `e2e/`. A test for a requirement or an invariant starts its name with the ID (`RF-41`, `INV-03`).
- **One slice, one branch, one pull request.** `main` is protected. Small commits, in English, imperative mood.
- **Stable versions only**, pinned as the ADRs say. Before adding a dependency, check that it is maintained and say why it is needed in the pull request.
- **Language: decided by who reads it** (owner's decision of 17/11/2026; ADR 0001, note of 17/11):

  | Reader | Language | What |
  |---|---|---|
  | Developers and the coding agent | **English** | Everything in this repository that is not shown to end users: code, comments, commits, pull requests, ADRs, README, every document in `docs/`, Spec Kit specs and the instructions of the prompts |
  | Teachers, on screen | **Brazilian Portuguese** | Screen text, error messages, AI output (the plan), the exported file, the BNCC catalog and the content of evaluation cases |

  A commit title with Portuguese accents is a mistake: rewrite it in English. Use the names in the glossary of `docs/DOMAIN.md`; add a new name there before using it.
- **Explain as you go.** The owner is learning this stack: say in a few lines why a choice was made.
- **Stop and ask** before deleting data, changing a protected document, adding a paid service or weakening a rule above.

## Commands

Defined by the first slice (F00). Until then there is nothing to run.
