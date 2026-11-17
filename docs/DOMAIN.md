# Domain: Docenda

> **v0.2 (17/11/2026).** Born from the planning review of 16/11; v0.2 aligns the names with [DATA](DATA.md) and [CONTRACTS](CONTRACTS.md) (changes in section 9). It says **what exists** in the system and **which rules can never break**. The "how" lives in the [decisions](decisions/) and in each slice's specification ([BACKLOG](BACKLOG.md)).
> Based on: [REQUIREMENTS](REQUIREMENTS.md), decisions 0001 to 0014.

## 1. Domain language (Portuguese ↔ code)

Code is in English and the screen is in Portuguese (decision 0001). This table is the only accepted translation: a new name goes here before it goes into the code. The first column is the screen text and stays in Portuguese.

| Term (screen) | In code | Note |
|---|---|---|
| Plano de aula | `LessonPlan` | The document for one lesson |
| Modelo do plano | `PlanTemplate` | The internship script (v1) or ENADE (later). It is **data**, not code (section 8) |
| Identificação do plano | `PlanIdentification` | School, teacher, class, date, shift. Kept in a separate table and **never sent to the AI** (INV-09) |
| Contexto da turma | `ClassContext` | Aggregate description, with no identifiable student |
| Tamanho da turma | `classSize` | How many students usually attend; a number, no names |
| Cópia do contexto | `ContextSnapshot` | The context as it was when the plan was created; this is what goes to the AI |
| Etapa de ensino | `EducationStage` | Early childhood, elementary (early and final years), high school |
| Ano ou série | `grade` | 1 to 9 in elementary, 1 to 3 in high school |
| Etapa de produção | `ProductionStep` | Simplified script → plan → slides → standardized script (after v1) |
| Atividade (momento da aula) | `Activity` | A timed block: introduction, development, consolidation or closing |
| Duração nominal | `nominalMinutes` | The official length of the lesson |
| Tempo efetivo | `effectiveMinutes` | The time the plan is made for |
| Hora-aula | `classHourMinutes` | 50 by default, adjustable |
| Habilidade | `Skill` | Catalog item, with code and literal text |
| Catálogo | `CurriculumFramework` | BNCC Computing is the first; the table accepts others (section 8) |
| Fonte | `Source` | A reference with a link, checked and approved by the author |
| Sugestão da IA | `Suggestion` | A proposal for one field, not yet decided by the author |
| Achado | `Finding` | Result of a rule or of the evaluation: blocks, warns or informs |
| Geração | `AiGeneration` | One generation request and its progress |
| Autor | `ownerId` | Owner of the plan and of the context |
| Revisão (número de gravação) | `revision` | Goes up on every save of the plan or the context; stops two tabs from overwriting each other. Not to be confused with `version`, which counts approvals |
| Convite | `Invite` | |
| Crédito de IA | `CreditEntry` | An entry in the ledger (section 6) |
| Uso de IA | `AiUsageRecord` | Tokens and cost of one call, with no content |
| Ocorrência de uso indevido | `MisuseOccurrence` | |
| Bloqueio da IA | `MisuseBlock` | Stops the person from using the AI; does not deactivate the account |
| Aceite dos termos | `TermsAcceptance` | Date and version of the accepted terms |
| Recibo de exclusão | `DeletionReceipt` | Proves a deletion request was fulfilled, without identifying anyone |
| Auditoria | `AuditEvent` | Who did what, with IDs only |
| Registro de acesso | `AccessRecord` | Account, IP and time of login |

> **Careful with "etapa".** The word had two meanings in REQUIREMENTS v0.2. From v0.3 on: **etapa de ensino** (`EducationStage`, the education stage) and **etapa de produção** (`ProductionStep`, the production step). "Etapa" alone is not used.

## 2. Entities and the owner of each

The "owner" is the module that creates the tables and the only one that queries them (decision 0012). Other modules ask through the front door (`index`).

| Entity | Owner module | Stores | Milestone |
|---|---|---|---|
| `User`, `Session` | `reusable/auth` (Better Auth tables) | Name, e-mail, password hash, role, sessions | M1 |
| `Invite`, `TermsAcceptance` | `reusable/auth` | Invited e-mail, token hash, expiry; acceptance of the terms | M2 |
| `ClassContext` | `modules/class-context` | Education stage, grade, subject, length, class size and the aggregate texts. The week's workload and the teacher's opinion apply to one lesson and live in the plan | M1 |
| `LessonPlan` | `modules/plans` | Owner, context snapshot, template, state, version, theme, times, pasted material and accepted content (`jsonb` with a schema version) | M1 |
| `PlanIdentification` | `modules/plans` | School, teacher, class, date and shift, in a separate table | M1 |
| `LessonPlanVersion` | `modules/plans` | Immutable copy of the plan at each approval | M1 |
| `PlanExport` | `modules/plans` | Event: exported version, format, time | M1 |
| `Suggestion` | `modules/plans` | Target field, proposed value, origin, state | M1 |
| `PlanSkill` | `modules/plans` | Links the plan to a catalog `Skill` (foreign key) | M1 |
| `Skill`, `CurriculumFramework` | `modules/bncc` | Code, literal text, education stage, grades, axis, competency, explanation, examples, catalog version | M1 |
| `Source` | `modules/sources` | Link, domain, state, check result, ABNT reference | M2 |
| `AiGeneration` | `modules/plans` | State, current step, prompt version, model, failure reason | M1 |
| `AiUsageRecord`, `CreditEntry`, `AiBudgetPeriod`, `CreditAccount` | `reusable/ai-usage` | Tokens, cost, credit entries, the month's spend | M1 (usage, reservation and system cap), M2 (per-person credit) |
| `MisuseOccurrence`, `MisuseBlock` | `reusable/misuse` | Category, severity, minimal excerpt, admin decision; the block and the review request | M2 |
| `AuditEvent`, `AccessRecord`, `DeletionReceipt` | `reusable/audit` | Event type, IDs and time; account, IP and time; deletion receipt | M2 |
| `Setting` | `reusable/settings` | AI on/off switch, limits and caps changed without a deploy (RF-75) | M1 (by command), M2 (on screen) |

## 3. Inside the plan

The plan is stored in **one internal format** and exported to the chosen template. Accepted content lives in `jsonb`, validated by a versioned schema in `packages/contracts`. Whatever the database queries or checks lives in tables: skills, sources, suggestions and generations.

| Internal field | Type | Does the AI suggest it? | In the internship script |
|---|---|---|---|
| `identification` | separate table | **Never** | Identificação |
| `theme`, `subtheme` | text | No (comes from the author) | Tema, subtema |
| `skills` | link to the catalog | Yes, by code | Habilidades da BNCC Computação |
| `justification` | text | Yes | Justificativa |
| `generalObjective` | text | Yes | Objetivo geral |
| `specificObjectives` | list of texts | Yes | Objetivos específicos |
| `contents` | list of texts | Yes | Conteúdos |
| `priorKnowledge` | text | Yes | Conhecimentos prévios |
| `activities` | list of `Activity` (moment, title, description, minutes, `setupHeavy`) | Yes | Procedimentos |
| `resources` | list of texts | Yes | Recursos |
| `assessment` (`criteria`, `instruments`) | two lists of texts, a single unit | Yes | Avaliação |
| `adaptations` | text | Yes | Adaptações |
| `crossSubjectLink` | text | Yes | Ligação com outras matérias (RF-27) |
| `sources` | link to `Source` | Not in v1 | Bibliografia |

Format rules:
- **Plain text.** No field stores or displays HTML or Markdown coming from the AI. The screen writes text as text; the export is what builds the Markdown.
- **`setupHeavy`** marks an activity that installs software, creates an account or configures equipment (RF-14). The AI marks it during generation, or the author during editing.
- **Unit of suggestion: the field.** Each row of the table above is one unit that the author accepts, edits or rejects as a whole (RF-23). Skills and sources are decided item by item, because each already has its own state. Per-item suggestions inside a list come after v1.

## 4. States

### 4.1 Plan
```
draft ──approve──▶ approved
    ▲                    │
    └──────edit──────────┘   (the approved version is kept)
```
- **`draft`:** being edited. May have pending suggestions and findings.
- **`approved`:** only the author gets here, and only under the conditions of INV-03. Approving saves a `LessonPlanVersion`.
- **Editing an approved plan** requires reopening: an author action takes the plan back to draft with the next version number. The approved version stays stored and exportable. The identification is not content: it can change without reopening.
- **"Exported" is not a state, it is an event** (`PlanExport`). A plan can be exported many times; the screen shows the date of the last export of that version.
- **`in_review`** is reserved for when reviewers exist (RF-80 onwards). In v1 there is no reviewer besides the author.

### 4.2 Generation
`queued` → `running` → `succeeded` or `failed`.

Every failure has a reason from a closed list, and the screen shows its own message for each:

| `failureReason` | When | Counts as misuse? | Charged? |
|---|---|---|---|
| `input_blocked_privacy` | Unresolved student-data alert (RF-15) | No | Only the check |
| `input_blocked_misuse` | Input check (RF-90) | Yes, depending on severity | Only the check |
| `off_topic` | Request outside the purpose (RF-95) | No | Only the check |
| `provider_refusal` | The provider refused (`stop_reason: refusal`) | **No** | What was used |
| `invalid_output` | Invalid JSON after 2 retries (RNF-35) | No | What was used |
| `output_check_failed` | The response failed the output check | No | What was used |
| `budget_exhausted` | The reservation ran out midway (0008 D3) | No | What was used |
| `provider_unavailable` | Circuit breaker open or provider error | No | Nothing |
| `worker_lost` | The worker crashed and the job ran out of retries | No | **The whole reservation** (INV-07) |

### 4.3 Suggestion
`pending` → `accepted`, `edited` or `rejected`. Accepting copies the value into the plan content; editing copies the author's version; rejecting copies nothing. While a field has a pending suggestion, the author does not write to it directly: they decide the suggestion first. A new generation replaces the **pending** suggestions of the same field and never touches what the author has already decided.

### 4.4 Source
`to_review` → `approved` or `rejected`. Separately, the result of the link check: `unchecked`, `ok` or `broken`.

### 4.5 Invite and account
- Invite: `pending` → `accepted`, `cancelled` or `expired` (7 days).
- Account: `active` → `deactivated` (30 days; logging in or the e-mail's cancel link reactivates it) → deleted or anonymized (REQUIREMENTS, section 7.1). "Delete now" in the e-mail, with the password, skips the rest of the 30 days.
- Misuse never changes this state: the AI block is a separate flag (`MisuseBlock`). There is no account suspension: after an incident, the account stays `active`, its sessions end and the password must be reset by e-mail (precautionary reset, [INCIDENTS](INCIDENTS.md), step 1).

## 5. Findings: one format for rules and evaluation

Every rule in section 4 of REQUIREMENTS and every critique from the AI evaluation (RF-26) returns the same thing:

| Field | Meaning |
|---|---|
| `ruleId` | Which rule (list below) or which rubric item |
| `severity` | `block` (prevents approval), `warn` (the author decides) or `info` |
| `fieldPath` | The plan field it refers to, when there is one |
| `message` | Text for the author, in Portuguese |
| `suggestionId` | Suggestion linked to the finding, when the evaluation proposes a fix |

Each deterministic rule is a **pure function** `(plan, context) → findings`, with a unit test and edge cases (RNF-09). They are recomputed on every read; only the findings of the AI evaluation are stored, because they cost money.

**A finding from the AI is never `block`.** The AI evaluation returns only `warn` or `info`, with a `ruleId` starting with `rubric.` ([EVALUATION](EVALUATION.md), section 4). Code rules are what stop approval; the AI gives an opinion, it does not decide (INV-03).

| `ruleId` | Rule | Severity | Requirement |
|---|---|---|---|
| `time-sum` | The sum of the activities' minutes equals the effective time | `block` | Section 4 |
| `pending-suggestion` | There is a pending AI suggestion | `block` | RF-23 |
| `source-to-review` | There is a source "to review" | `block` | RF-51 |
| `skill-required` | The plan has at least one skill | `block` | RF-41 |
| `skill-stage-mismatch` | The skill does not match the class's education stage or grade | `warn` | RF-43 |
| `setup-heavy` | There is a `setupHeavy` activity and the effective time cannot fit the rest | `warn` | RF-14 |
| `privacy-hint` | The text seems to contain a student's name, a diagnosis or a medical report | `block` until the author fixes or confirms it. On save, text with an alert is not even stored (DATA, D6); the finding covers old text when the term list grows | RF-15 |
| `empty-required-field` | A required field of the template is empty | `block` | RF-20 |

A skill outside the catalog is not a finding: it **cannot be saved** (INV-02).

## 6. AI credit: the ledger

A ledger is a table where entries are only appended; the balance is their sum. Nothing is rewritten, so every cent has an explanation.

| Entry | Amount | When |
|---|---|---|
| `grant` | + the month's credit (US$ 1 in v1) | On the first use of the month, or by the job on day 1 |
| `reserve` | − maximum cost of the operation | When the generation is queued (0008 D3) |
| `settle` | + (reservation − actual cost) | When it finishes, with `response.usage` |

- **Period:** the month in Brasília time. Credit does not accumulate because each period has its own sum.
- **Lock:** the reservation runs in a transaction that locks the user's credit account row and the month's cap row before adding up. Two simultaneous requests queue one behind the other.
- **Unknown outcome:** if the worker crashes and nobody knows whether the call was charged, there is no `settle`: the whole reservation stays as cost. The system may overcharge the user by mistake, never undercount against the cap.
- **Unit:** nano-dollar (US$ 1 = 1,000,000,000), in `bigint`. With a thousandth of a cent, the per-token price of the cheapest model would already be a fraction (RNF-42).

## 7. Invariants

Rules that always hold. Each has at least one test with its number in the name.

| # | Invariant | How it is enforced |
|---|---|---|
| INV-01 | An author only reaches what is theirs | Every repository for an author's resource requires `ownerId`; another author's resource answers 404; a generic test over every route |
| INV-02 | Only a catalog skill, with the catalog text, enters the plan | Foreign key from `PlanSkill` to `Skill`; the plan stores the code, and the text is read from the catalog |
| INV-03 | A plan is only approved with no `block` finding, and only by the author | Pure approval function; the operation exists only on the owner's route, not in the worker |
| INV-04 | The AI never writes plan content | The worker only creates `Suggestion`; there is no write operation on content outside the author's actions |
| INV-05 | At most one active generation per plan | Partial unique index (0008 D5) |
| INV-06 | The month's balance never goes negative, and the cap is never exceeded | Row lock on the reservation (section 6) |
| INV-07 | The recorded cost is never lower than the actual cost | An unknown outcome keeps the whole reservation |
| INV-08 | No generation stays "running" forever | A job that closes orphaned generations (0008, note of 16/11) |
| INV-09 | Plan identification, user name, e-mail and ID never enter a prompt | The AI port receives a type that lacks these fields; a test with decoy values |
| INV-10 | Nothing goes to the AI with an unresolved privacy alert | A pipeline step, before any call |
| INV-11 | The queue job carries IDs only | Job type; test (0008 D6) |
| INV-12 | Money is an integer | Its own type; no `number` for cost |
| INV-13 | Every generation knows which prompt, model and effort produced it | Required columns in `AiGeneration` and `AiUsageRecord` |
| INV-14 | The audit only grows | Database role without `UPDATE` or `DELETE` (0010 D6, 0012 D5) |

## 8. Decisions that are cheap now and expensive later

1. **Generic catalog.** `Skill` belongs to a `CurriculumFramework` (name and version). BNCC Computing is the first row. Other BNCC components come later as data, with no table change.
2. **Plan template as data.** The `PlanTemplate` says which internal fields appear, in what order and with what title. The exporter reads the template; adding ENADE means writing a new template and, at most, a new exporter.
3. **An owner on everything.** Every author resource has `ownerId`. When schools or teams exist, an optional organization column comes in, without rewriting the queries.
4. **Schema version in the content.** The plan's `jsonb` carries `schemaVersion`. Changing the format means writing a migration function from one version to the next.

## 9. Changes in v0.2 (17/11/2026)

| What changed | Why |
|---|---|
| Plan identification in a separate table, outside the approved version | The AI query cannot reach it; changing the date does not reopen the plan |
| Assessment became a single field, `assessment`, with criteria and instruments | It is one unit of suggestion; two loose names were confusing |
| Reopening is an author action of its own | A key pressed by mistake must not undo an approval |
| The week's workload and the teacher's opinion in the plan; class size in the context | The context lasts months; exam week lasts one lesson |
| New names in the glossary: `revision`, `ContextSnapshot`, `classSize`, `MisuseBlock`, `TermsAcceptance`, `DeletionReceipt` | A new name goes here before the code |
| A finding from the AI never blocks approval | The AI suggests and gives opinions; locking the plan would be deciding |
| `Setting` exists from M1, changed by command | The AI on/off switch (RF-75) is in slice F07 |
