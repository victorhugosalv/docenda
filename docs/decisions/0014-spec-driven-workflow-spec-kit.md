# 0014: Spec-driven workflow with Spec Kit

- **Date and status:** 16/11/2026. Written in the planning review, decided by Claude at Victor's request; Victor reviews it in the pull request. Not installed yet: it comes with the first slice (BACKLOG, F00).
- **Context:** The architecture track was planned as the step "before the Spec Kit". Until now the method was manual: a purpose document, a requirements document, one ADR per decision. The code is written with a coding agent, one vertical slice at a time, and each slice needs a specification the agent can follow and Victor can review. Two tools were compared, both checked in their official repositories on 16/11/2026:
  - **Spec Kit** (GitHub, MIT): 1.4.1 released on 16/11/2026; needs Python 3.11 or later and `uv`. Each feature gets a specification, a plan and a task list, and a constitution holds the project rules.
  - **OpenSpec** (Fission AI, MIT): 2.0.0 released on 11/11/2026; needs Node.js 22.12 or later, or a standalone binary. It keeps a living specification per domain and proposes each change as a delta that is merged when the change is archived.
- **Decision:**
  - **D1. Spec Kit**, pinned to **v1.3.3** (09/11/2026), the newest release that is at least 7 days old, by the same rule as the dependency cooldown (0009 D5). Installed in the repository with the guide for existing projects: `specify init --here --force --integration claude`, on its own branch, with the diff reviewed before merging.
  - **D2. What each existing document becomes:**

    | Document | Role in the workflow |
    |---|---|
    | [PURPOSE](../PURPOSE.md), the ADRs and the invariants in [DOMAIN](../DOMAIN.md) | Evidence for the constitution. It states only rules that are already true or decided |
    | [REQUIREMENTS](../REQUIREMENTS.md) | The product catalog. It stays the single place where a requirement is written |
    | [BACKLOG](../BACKLOG.md) | The order of features. Each slice is one Spec Kit feature |
    | `specs/NNN-<slice>/` | The specification, plan and tasks of that slice. It cites requirements by ID and never copies their text |
    | ADRs | Input to the plan step. A plan follows them; a new decision is a new ADR, not a line in a plan |

  - **D3. The loop of one slice:** specify, clarify, plan, tasks, analyze, implement, converge. One branch and one pull request per slice. One chat per slice.
  - **D4. Rules that keep it lean:**
    - the first feature is a bounded slice (F00), never a retroactive specification of the whole system;
    - if a slice changes a requirement, a domain rule or a decision, the document changes in the same pull request. The slice specification says what changed and why;
    - generated files that only repeat an existing document are trimmed before the merge.
  - **D5. To confirm when installing:** the exact way commands are called in Claude Code (the project page shows the hyphen form, such as `/speckit-specify`), and which files the `claude` integration adds.
- **Alternatives considered:**
  - OpenSpec 2.0.0: its model fits the requirements document well, since a living specification per domain plus deltas is what the "changes in REQUIREMENTS" notes in the ADRs do by hand. Against it: 2.0.0 was 5 days old and had just renamed every command; adopting it would mean moving REQUIREMENTS into a second format; and Victor already knows Spec Kit. Revisit if keeping REQUIREMENTS in step with the code starts to hurt.
  - Staying manual: nothing to install, but each slice would need its own hand-written specification and task format.
  - Spec Kit 1.4.1: released on the day of this decision; it adds catalog entries and workflow options that are not needed here.
- **Consequences:**
  - Pros: every slice has the same shape; the agent starts each chat from a specification instead of the whole history; the review step is a document, before any code.
  - Cons: one more toolchain on the machine (Python and `uv`); more Markdown per slice; commands and flags changed across recent releases (the `--ai` flags are gone, command names use a hyphen), so older guides mislead.
  - Changes elsewhere: none in the other ADRs.
  - Cost: none.
- **Sources (checked on 16/11/2026):** [Spec Kit](https://github.com/github/spec-kit), [Spec Kit releases](https://github.com/github/spec-kit/releases), [Spec Kit: existing projects](https://github.github.io/spec-kit/how-to/existing-projects.html), [OpenSpec](https://github.com/Fission-AI/OpenSpec), [OpenSpec releases](https://github.com/Fission-AI/OpenSpec/releases), [OpenSpec: getting started](https://github.com/Fission-AI/OpenSpec/blob/main/docs/getting-started.md).
