# API contracts: Docenda

> **v0.1 (17/11/2026). Planned: there is no code yet.** Says what goes into and comes out of each route. It becomes the Zod schemas of `packages/contracts`, which apply to the API and to the screen (0006 D5, 0013 D3). Each slice's specification may adjust a detail; when it does, it fixes this file in the same PR.
> Siblings: [INTERFACE](INTERFACE.md) (map of screens and routes), [DATA](DATA.md) (tables) and [DOMAIN](DOMAIN.md) (names and rules).

## 1. General rules

| Subject | Rule |
|---|---|
| Address | Everything under `/api/v1`, except `/healthz`, `/readyz` and `/api/auth/*` |
| Format | JSON in UTF-8, names in `camelCase`. An unknown field in the request is refused |
| Identifier | `uuid` as text |
| Date and time | ISO 8601 in UTC (`2026-11-17T12:00:00Z`). Date without time: `2026-11-17` |
| Money | Nano-dollars as **text** (`"1000000000"`), never a number: JSON loses precision on large integers (INV-12) |
| Text | Plain. No field accepts or returns HTML or Markdown (0006) |
| Session | Better Auth cookie. Without a session: `401`. Every route requires a session, except the authentication and health ones |
| Owner | Another author's resource answers `404`, the same as a resource that does not exist (RNF-40) |
| Request that changes data | Checks the `Origin` header (0006 D5) |
| Response | Always with `X-Request-Id` and `Cache-Control: no-store` |
| Size | Body up to 256 kB; above that, `413` (RNF-44) |

## 2. Two people, two tabs, two clicks

- **`revision`.** Every plan and every context has a number that goes up on every save. Whoever changes it sends the `revision` they read. If it has already changed, the answer is `409 revision_conflict` with the current state, and the screen decides what to show. Without this, one tab's autosave would silently erase what the other wrote.
- **Double click on generation** (RNF-37). The second `POST` returns the generation that is already active, with the same `id`. The database's unique index guarantees it (INV-05), not the screen.
- **Repeated decision.** Deciding a suggestion that has already been decided answers `409 suggestion_already_decided`.

## 3. Errors

Problem format (RFC 9457), with `Content-Type: application/problem+json`:

```json
{
  "type": "https://docenda.example/problems/validation-failed",
  "title": "Dados inválidos",
  "status": 422,
  "code": "validation_failed",
  "requestId": "0193…",
  "errors": [{ "path": "time.effectiveMinutes", "code": "too_small", "message": "Informe um tempo maior que zero." }]
}
```

- `code` is stable and is what the screen uses to choose the message. `title` and `message` are in Portuguese.
- `errors` only appears in `validation_failed`. `findings` only in `plan_not_approvable`. `fields` only in `privacy_alert`.
- There is never a stack trace, table name or internal message (0012 D7).

| `code` | Status | When |
|---|---|---|
| `validation_failed` | 422 | The body fails the schema |
| `malformed_request` | 400 | Broken JSON |
| `unauthenticated` | 401 | No session or an expired session. The screen opens the login dialog (0013 D7) |
| `forbidden` | 403 | Administrator route without the role |
| `not_found` | 404 | It does not exist or belongs to another author |
| `payload_too_large` | 413 | Body over the limit |
| `rate_limited` | 429 | General request limit (RNF-44), with `Retry-After` |
| `revision_conflict` | 409 | Section 2 |
| `privacy_alert` | 409 | A text seems to contain student data (RF-15). Carries `fields: [{fieldPath, terms}]` |
| `suggestion_pending` | 409 | Tried to write to a field that has an undecided suggestion |
| `suggestion_already_decided` | 409 | Section 2 |
| `plan_approved` | 409 | Tried to edit an approved plan without reopening it |
| `plan_not_approvable` | 409 | There is a blocking finding. Carries `findings` |
| `no_approved_version` | 409 | Exporting without an approved version (scenario F6.2) |
| `generation_active` | 409 | Deleting or reopening a plan with a generation in progress |
| `skill_not_found` | 422 | Code outside the catalog (scenario F4.2) |
| `email_not_verified` | 403 | AI request without a confirmed e-mail (RF-70) |
| `ai_blocked` | 403 | AI blocked for the person because of misuse (RF-91) |
| `ai_disabled` | 503 | AI turned off by the administrator (RF-75) |
| `ai_budget_reached` | 503 | The month's cap reached (RF-72) |
| `ai_credit_exhausted` | 429 | No balance left this month (RF-71) |
| `ai_rate_limited` | 429 | Per-minute or per-day limit (RF-71), with `Retry-After` |
| `input_too_large` | 422 | Input over the operation's limit (RF-73) |
| `internal_error` | 500 | A defect. Only the `requestId` appears |

Failures **during** generation are not HTTP errors: the request has already answered `202`. They appear in `failureReason` (section 4.6).

## 4. Formats

TypeScript notation. `?` is optional; `| null` means "may be empty".

### 4.1 Plan content (`schemaVersion: 1`)

```ts
type PlanContent = {
  schemaVersion: 1;
  justification: string;
  generalObjective: string;
  specificObjectives: string[];
  contents: string[];
  priorKnowledge: string;
  activities: Activity[];
  resources: string[];
  assessment: { criteria: string[]; instruments: string[] };
  adaptations: string;
  crossSubjectLink: string;
};

type Activity = {
  moment: 'introduction' | 'development' | 'systematization' | 'closing';
  title: string;
  description: string;
  minutes: number;        // integer, greater than zero
  setupHeavy: boolean;    // install, create an account or configure (RF-14)
};
```

- **A draft accepts empty values.** Empty text and an empty list are valid until approval; the `empty-required-field` rule is what enforces them.
- **Units of suggestion** (RF-23): each key of `PlanContent` is one unit, and `assessment` is a single one. Ten units, plus the skills, which are decided one by one.
- `theme`, `subtheme` and the times stay outside `PlanContent`: they come from the author and are never suggested.

### 4.2 Context

```ts
type ContextFields = {
  educationStage: 'early_childhood' | 'elementary_early' | 'elementary_late' | 'high_school';
  grade: number | null;
  subject: string | null;
  nominalMinutes: number;
  classHourMinutes: number;      // 50 by default
  classSize: number | null;
  priorKnowledge: string | null;
  commonDifficulties: string | null;
  socialContext: string | null;
  schoolResources: string | null;
  accessibility: string | null;
};

type ClassContext = ContextFields & { id: string; label: string | null; revision: number; updatedAt: string };

type ContextSnapshot = ContextFields & { schemaVersion: 1 };   // without `label`
```

### 4.3 Plan

```ts
type Plan = {
  id: string;
  state: 'draft' | 'approved';
  version: number;
  revision: number;
  theme: string;
  subtheme: string | null;
  time: { nominalMinutes: number; effectiveMinutes: number; classHourMinutes: number };
  lesson: { examWeek: boolean; weekNotes: string | null; teacherNotes: string | null };
  sourceMaterial: string | null;
  classContextId: string | null;
  context: ContextSnapshot;
  content: PlanContent;
  skills: PlanSkill[];
  sources: Source[];                       // empty until F13
  identification: PlanIdentification;
  suggestions: Suggestion[];               // pending ones only
  findings: Finding[];
  activeGeneration: Generation | null;
  lastGeneration: Generation | null;       // to show the message of the last failure
  approvedVersions: { version: number; approvedAt: string; lastExportedAt: string | null }[];
  createdAt: string;
  updatedAt: string;
};

type PlanSummary = Pick<Plan, 'id' | 'state' | 'version' | 'theme' | 'updatedAt'> & {
  educationStage: ContextFields['educationStage'];
  grade: number | null;
  hasActiveGeneration: boolean;
};

type PlanIdentification = {
  educationOffice: string | null; school: string | null; city: string | null;
  teacherName: string | null; classLabel: string | null;
  lessonDate: string | null; shift: string | null;
};
```

The identification is shown to the author, who owns it. It only does not exist in the type the AI port receives (INV-09).

### 4.4 Skill

```ts
type Skill = {
  code: string;                 // "EM13CO21"
  text: string;                 // literal
  formatted: string;            // "(EM13CO21) Texto literal. (Competência específica 6)" (RF-41)
  educationStage: ContextFields['educationStage'];
  grades: number[];
  axis: 'computational_thinking' | 'digital_world' | 'digital_culture';
  competenceNumber: number | null;
};

type PlanSkill = Skill & { origin: 'author' | 'ai' };
```

`formatted` is built by the server from the catalog. The screen never builds this text, so it has no way to alter it.

### 4.5 Suggestion and finding

```ts
type Suggestion = {
  id: string;
  fieldPath: keyof Omit<PlanContent, 'schemaVersion'> | 'skills';
  value: unknown;               // the whole field's value; for 'skills', { skill: Skill, reason: string }
  origin: 'from_material' | 'added';
  status: 'pending';
  generationId: string;
};

type Finding = {
  ruleId: string;               // list in DOMAIN, section 5
  severity: 'block' | 'warn' | 'info';
  fieldPath: string | null;
  message: string;
  suggestionId: string | null;
  stale: boolean;               // evaluation finding made on an old revision
};
```

### 4.6 Generation

```ts
type Generation = {
  id: string;
  operation: 'generate_plan' | 'evaluate_plan';
  status: 'queued' | 'running' | 'succeeded' | 'failed';
  stage: 'checking_input' | 'generating' | 'validating' | 'checking_output' | null;
  failureReason:
    | 'input_blocked_privacy' | 'input_blocked_misuse' | 'off_topic' | 'provider_refusal'
    | 'invalid_output' | 'output_check_failed' | 'budget_exhausted'
    | 'provider_unavailable' | 'worker_lost' | null;
  failureDetails: { fields?: string[] } | null;   // field names only
  createdAt: string;
  finishedAt: string | null;
};
```

### 4.7 Size limits (initial values, in configuration)

| Field | Maximum |
|---|---|
| `theme`, `subtheme`, `label` | 200 characters |
| Each context text; `weekNotes`, `teacherNotes` | 1,500 |
| `sourceMaterial` | 12,000 |
| Each text in `PlanContent`; the activity's `description` | 3,000 |
| List item; the activity's `title` | 500 |
| Items per list | 12 |
| Activities | 10 |
| Skills per plan | 6 |
| Each identification field | 150 |

These limits are the first barrier of RF-73. The second is the token count before the paid call ([AI](AI.md), section 6). F04 measures the values with the test contexts and adjusts them.

## 5. Routes

### 5.1 Health (F00)

| Route | Response |
|---|---|
| `GET /healthz` | `200 {"status":"ok"}` if the process is alive |
| `GET /readyz` | `200` if the database answers; `503` if not |

### 5.2 Catalog (F01)

| Route | Input | Response |
|---|---|---|
| `GET /skills` | `stage` (required), `grade?`, `q?` (search in code and text) | `200 { frameworkVersion, items: Skill[] }` |
| `GET /skills/:code` | — | `200 Skill & { explanation, examples, knowledgeObject }`; `404` |

### 5.3 Account and session (F05)

| Route | Input | Response |
|---|---|---|
| `/api/auth/*` | Better Auth routes: log in and log out | Defined by the library; to be confirmed in F05 |
| `GET /me` | — | `200 { id, name, email, role, emailVerified, aiBlocked }` |

Logging in with wrong data always returns the same message, "E-mail ou senha incorretos" (RF-03).

### 5.4 Class context (F06)

| Route | Input | Response | Own errors |
|---|---|---|---|
| `GET /class-contexts` | — | `200 { items: ClassContext[] }`, newest first | |
| `POST /class-contexts` | `ContextFields & { label?, privacyConfirmations? }` | `201 ClassContext` | `privacy_alert` |
| `GET /class-contexts/:id` | — | `200 ClassContext` | |
| `PATCH /class-contexts/:id` | `{ revision, …fields, privacyConfirmations? }` | `200 ClassContext` | `revision_conflict`, `privacy_alert` |
| `DELETE /class-contexts/:id` | — | `204`. The plans made with it stay (RF-17) | |

**Privacy alert.** If a text contains a term from the list (for example "laudo", "CID", "diagnosticado"), nothing is stored and the answer is `409 privacy_alert` with the fields and the terms. The author fixes the text or resends it with `privacyConfirmations: ["commonDifficulties"]`, declaring that the excerpt identifies nobody. The same applies to the plan.

### 5.5 Plan (F06)

| Route | Input | Response | Own errors |
|---|---|---|---|
| `GET /plans` | `cursor?`, `limit?` (20 by default, 50 at most) | `200 { items: PlanSummary[], nextCursor }` | |
| `POST /plans` | `{ classContextId, theme, subtheme?, time: { effectiveMinutes, nominalMinutes?, classHourMinutes? }, lesson?, sourceMaterial?, privacyConfirmations? }` | `201 Plan`, as a draft, with the context snapshot | `privacy_alert` |
| `GET /plans/:id` | — | `200 Plan` | |
| `PATCH /plans/:id` | `{ revision, theme?, subtheme?, time?, lesson?, sourceMaterial?, context?, content?, privacyConfirmations? }` | `200 Plan` | `revision_conflict`, `privacy_alert`, `suggestion_pending`, `plan_approved` |
| `PUT /plans/:id/identification` | `PlanIdentification` | `200 Plan`. Works in any state and does not change the `version` | |
| `POST /plans/:id/skills` | `{ code }` | `201 Plan` | `skill_not_found`, `plan_approved` |
| `DELETE /plans/:id/skills/:code` | — | `200 Plan` | `plan_approved` |
| `POST /plans/:id/suggestions/:sid/decision` | `{ decision: 'accept' }`, `{ decision: 'reject' }` or `{ decision: 'edit', value }` | `200 Plan` | `suggestion_already_decided`, `privacy_alert` |
| `POST /plans/:id/approve` | `{ revision }` | `200 Plan`, now `approved`, with the new version in `approvedVersions` | `plan_not_approvable`, `revision_conflict` |
| `POST /plans/:id/reopen` | — | `200 Plan`, as a draft, with `version + 1` | `generation_active` |
| `POST /plans/:id/duplicate` | — | `201 Plan`, a new one, as a draft, version 1 | |
| `DELETE /plans/:id` | — | `204` | `generation_active` |

Rules:
- **`PATCH` replaces the whole field.** `content` carries only the changed keys, and each key replaces the old value. The screen saves one field at a time, with a short delay (0013 D7).
- **A field with a pending suggestion does not accept `PATCH`.** The author decides the suggestion first: accepts it, edits from it or rejects it. That way no suggestion disappears without a decision.
- **An approved plan is read-only.** To change it, the author reopens it (`reopen`). The approved version stays stored and still exportable (RF-25).
- **Duplicating** copies theme, times, context, accepted content, skills and approved sources. It does not copy the identification, the pending suggestions or the versions.
- **Deciding a suggested skill:** `accept` creates the link to the catalog; `edit` does not exist for skills, because the text is literal.
- `findings` comes in every response with `Plan`, already recomputed.

### 5.6 Generation (F07)

| Route | Input | Response | Own errors |
|---|---|---|---|
| `POST /plans/:id/generations` | — | `202 Generation`. If one is already active, that same one | `plan_approved`, `email_not_verified`, `ai_blocked`, `ai_disabled`, `ai_budget_reached`, `ai_credit_exhausted`, `ai_rate_limited`, `input_too_large` |
| `GET /plans/:id/generations/:gid` | — | `200 Generation` | |

The screen asks for progress every 2 s while `status` is `queued` or `running` (0008 D3). When it finishes, it reads the plan again: the suggestions come in `Plan.suggestions`.

### 5.7 Export (F08)

| Route | Input | Response | Own errors |
|---|---|---|---|
| `POST /plans/:id/exports` | `{ version?, format: 'md', includeAiNote? }`. Without `version`, the latest approved; `includeAiNote` is `true` if absent (RF-61) | `200` with `Content-Type: text/markdown; charset=utf-8` and `Content-Disposition: attachment` | `no_approved_version`, `not_found` |

It is a `POST` because exporting creates an event (`plan_export`). The server builds the file from the approved version and the identification as it is at that moment.

### 5.8 Milestone M2 (sketch; closed in each slice's specification)

| Route | Slice | What for |
|---|---|---|
| `GET /me/ai-credit` | F11 | `{ period, monthlyCreditNano, availableNano, generationsLeftToday, aiEnabled }` |
| `POST /invites/:token/accept` | F10 | Creates the account with the invited e-mail, the password and the acceptance of the terms |
| `PATCH /me`; e-mail and password change through the Better Auth routes | F10 | RF-05, RF-06 |
| `POST /me/deletion` (password and account name); `POST /account-deletion/cancel` and `POST /account-deletion/confirm` (e-mail token; confirm also takes the password) | F14 | RF-07. The e-mail links open a page; only its button calls these routes, so an e-mail scanner opening the link changes nothing |
| `GET, POST /plans/:id/sources`; `PATCH, DELETE /plans/:id/sources/:sid` | F13 | Source brought by the author |
| `POST /plans/:id/evaluations` | F15 | AI evaluation; returns a `Generation` with `operation: 'evaluate_plan'` |
| `POST /me/ai-block/review-request` | F09 | Request to review the block (RF-94) |
| `GET, POST /admin/invites`; `DELETE /admin/invites/:id` | F12 | RF-78 |
| `GET /admin/usage` | F12 | Cost per operation, per model and per day (RNF-14, RNF-39) |
| `GET /admin/occurrences`; `POST /admin/occurrences/:id/decision` | F12 | RF-93 |
| `GET, PUT /admin/settings/:key` | F12 | RF-75 |

No administrator route returns plan or context content (REQ-16).
