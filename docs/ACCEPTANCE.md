# Acceptance criteria per slice: Docenda

> **v0.1 (17/11/2026).** Says how to know that each slice of the [BACKLOG](BACKLOG.md) is done. Each row becomes at least one test, with the ID at the start of its name (0012 D9). It applies together with the BACKLOG's "definition of done", which is the same for all slices.
> The slices of milestone M1 (F00 to F08) are detailed. Those of M2 (F09 to F16) have the minimum and are closed in the specification, with whatever M1 teaches.
> Test types: `U` unit, `I` integration with PostgreSQL, `E2E` end-to-end, `Eval` evaluation of AI content, `Rev` manual review.

## How to use it with Spec Kit

In the slice's `/speckit-specify`, cite the section from here instead of rewriting the criteria. In `/speckit-tasks`, each criterion becomes the test task that comes **before** the code task. If the specification finds that a criterion is wrong, it is fixed here in the same PR.

## M0

### F00 `foundation`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | In a clean clone, `docker compose up` starts the database, test e-mail, migration, API and worker; the API only starts after the migration finishes | RNF-11, 0012 D5 | Rev |
| 2 | `GET /healthz` answers `200`; `GET /readyz` answers `200` with the database up and `503` with the database stopped | 0010 D7 | I |
| 3 | Connected as `docenda_app`, creating or altering a table fails. The three roles exist | 0012 D5 | I |
| 4 | When a required variable is missing, the process does not start and the message gives the variable's name, never its value | RNF-32 | U |
| 5 | A non-existent route under `/api/v1` answers `404` in the problem format, with `code` and `requestId`. An unexpected error answers `500 internal_error`, with no stack trace | 0012 D7 | I |
| 6 | Every response carries `X-Request-Id`; the same identifier appears on the request's log line | 0010 D2 | I |
| 7 | The log contains no request body, cookie, authorization header or IP. Test with decoy values | RNF-39, 0010 D3 | I |
| 8 | Responses with the helmet headers (CSP `default-src 'self'`, `nosniff`, `frame-ancestors 'none'`) and `Cache-Control: no-store` on the API | 0006 D5 | I |
| 9 | A body over the limit answers `413`; excess requests from the same IP answer `429` | RNF-44 | I |
| 10 | dependency-cruiser fails on a boundary violation planted in a test file | 0001, 0012 D1 | CI |
| 11 | On each PR, CI runs types, lint, boundaries, unit tests, integration tests with PostgreSQL and the image build; CodeQL, Dependabot and Gitleaks on; `main` protected | RNF-09, RNF-41, 0009 | Rev |
| 12 | The image runs without root, with a read-only file system | 0007 D2 | Rev |
| 13 | `apps/web` builds and the API serves the home page without violating the CSP | 0002, 0006 | I |
| 14 | Spec Kit pinned to the version in 0014, with the constitution written; the "Commands" section of `CLAUDE.md` filled in | 0014 | Rev |
| 15 | CI rejects a PR with a commit title that has a Portuguese accent (language rule, 0001, note of 17/11) | 0001 | CI |

Outside the slice: product tables, login and any screen besides the home page.

## M1

### F01 `bncc-catalog`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | The import command stores 140 skills: 11 for early childhood education and 26 for high school | RF-40 | I |
| 2 | Each skill's text equals the official document: an automatic check of all of them and a sample of 14 checked by hand, recorded | RF-40 | U, Rev |
| 3 | No text has words stuck together or a double space (a known defect of the PDF extraction) | RF-40 | U |
| 4 | Importing the same version twice changes nothing. A new version creates new rows and does not alter the old ones | RF-40 | I |
| 5 | `GET /skills?stage=high_school` returns only high-school skills; `grade` and `q` filter | RF-42 | I |
| 6 | `formatted` follows `(CÓDIGO) Texto literal. (Competência específica N)`; without a competency, it comes out without the final parenthesis | RF-41 | U |
| 7 | A non-existent code is refused with `skill_not_found` | RF-41 | U |
| 8 | `skill-stage-mismatch`: an `EM13` skill in an elementary class raises a warning; `EF15` in a 7th-grade class raises a warning; `EF69` in 7th grade does not | RF-43 | U |
| 9 | As `docenda_app`, writing to `skill` fails | DATA, section 5 | I |

### F02 `plan-core`

Everything here is a pure function: no database, no AI, no clock.

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | 1 class hour is 50 minutes; with a 45-minute class hour, 2 class hours are 90; hours and minutes convert to minutes | RF-12 | U |
| 2 | For 50 minutes, the system suggests 40 and 45 as effective time; the author can choose another value or keep 50 | RF-13 | U |
| 3 | `time-sum`: an equal sum raises no finding; a larger or smaller sum raises `block` with the difference in minutes; a plan with no activity raises `block` | Section 4 | U |
| 4 | `pending-suggestion`, `skill-required`, `empty-required-field`: each rule with the passing case, the blocking case and the edge | RF-23, RF-41, RF-20 | U |
| 5 | `setup-heavy`: warns when there is a `setupHeavy` activity together with another development activity and the effective time is up to 60 minutes; never blocks | RF-14 | U |
| 6 | `privacy-hint`: "laudo", "CID" and "diagnosticado" raise an alert, regardless of case or accents; "cidade" does not; confirmed text does not; text changed after confirmation raises it again | RF-15 | U |
| 7 | INV-03: approving is only possible with no `block` finding; approving an approved plan is an error | RF-25 | U |
| 8 | Reopening an approved plan returns a draft with `version + 1` | RF-25 | U |
| 9 | Suggestion: accepting copies the value; editing copies the author's text; rejecting copies nothing; deciding twice is an error | RF-23 | U |
| 10 | A new generation replaces only the pending suggestions of the same field | RF-23 | U |
| 11 | The link to other subjects is text and does not accept a skill code | RF-27 | U |
| 12 | No file in `domain/` imports anything outside `shared/` | 0012 D1 | CI |

Two rules have a proposed value and may change in the specification: the effective-time suggestion (nominal minus 10 and minus 5 minutes) and the 60-minute limit of `setup-heavy`.

### F03 `ai-generation`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | INV-09: the type the AI port receives has no identification, name, e-mail or user ID. With decoy values in those fields, none appears in the request | RNF-45 | U |
| 2 | INV-10: with an unresolved privacy alert, the fake adapter is not called even once | RF-15 | U |
| 3 | Cleaning applies NFKC and removes invisible characters; a block that looks encoded stops the generation | 0005 D8 | U |
| 4 | The output schema lists as `enum` only the codes for the class's education stage; a response with a code from outside is refused by validation | RF-42, INV-02 | U |
| 5 | Invalid JSON is retried at most 2 times; on the third failure, `invalid_output`. The fake adapter is called exactly 3 times | RNF-35 | U |
| 6 | A provider refusal becomes `provider_refusal` and records no misuse | AI, section 3 | U |
| 7 | Minutes below 1 or a sum different from the effective time: validation points it out and the generation retries within the limit of criterion 5 | AI, section 4 | U |
| 8 | Each call stores an `ai_usage_record` with tokens, cost, model, effort and prompt version, and no text | RF-74, INV-13 | I |
| 9 | INV-12: cost is computed as an integer. 1 million input tokens at US$ 2 gives exactly 2,000,000,000 nano-dollars | RNF-42 | U |
| 10 | Input over the limit is refused before any paid call | RF-73 | U |
| 11 | The prompt comes from `prompts/<operation>/v<N>.md`; two requests for the same education stage have an identical fixed prefix, byte for byte | AI, section 5 | U |
| 12 | Each generated field carries its origin (`from_material` or `added`) | RF-21 | U |
| 13 | The Anthropic adapter is tested against recorded responses, with no network; no test and no CI run uses a real key | AI, section 6 | U |
| 14 | `npm run generate -- --context <file>` shows the suggestions and the cost, using the key from the local environment | RNF-38 | Rev |
| 15 | The generation log contains no prompt or response text | RNF-39 | U |

The "store suggestions" step sits behind an interface; storing in the database comes in F07.

### F04 `evals-min`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | `npm run eval -- --suite <name> --max-cost <value>` runs the set and stops before going over the spending cap | AI, section 7.4 | U |
| 2 | The deterministic layer (checks C01 to C16 and G01 to G06) produces the targets table of [EVALUATION](EVALUATION.md), section 6 | RNF-10 | U |
| 3 | The result goes to `evals/results/<date>-<version>.md`, with cost and time per plan | RNF-14 | Rev |
| 4 | The synthetic set covers the four education stages, lessons of 25 to 90 effective minutes and the situations of [EVALUATION](EVALUATION.md), section 2 | RNF-10 | Rev |
| 5 | The confirmation test of 0005 (D7) was run and the result became a dated note in 0005 | 0005 D7 | Rev |
| 6 | `evals/private/` is outside git; no file of the reference set was committed | AI, section 7.1 | CI |
| 7 | The evaluation command itself has tests with the fake adapter | RNF-09 | U |

### F05 `auth-sessions`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | Logging in with correct data creates a server-side session and an `HttpOnly`, `SameSite=Lax` cookie; the identifier changes on every login | RNF-07 | I |
| 2 | A wrong password and a non-existent e-mail get the same answer: "E-mail ou senha incorretos" | RF-03 | I |
| 3 | The fourth attempt within 10 seconds from the same IP is refused | RF-03, scenario F1.2 | I |
| 4 | After 30 minutes without use, the next request answers `401` | RF-04 | I |
| 5 | Logging out deletes the session on the server; the old cookie answers `401` | RNF-07 | I |
| 6 | The password is stored with Argon2id; a password of 7 characters, of 129 or from the banned list is refused | 0003 D3 | U, I |
| 7 | Accounts and administrators are created only by command. No route creates or promotes an administrator | RF-79 | I |
| 8 | Every `/api/v1` route answers `401` without a session. The test walks the route list, so a new route is covered automatically | RF-70, 0006 D5 | I |
| 9 | A request that changes data with an `Origin` from another site is refused | 0006 D5 | I |
| 10 | An `X-Forwarded-For` sent by the client does not change the IP used for the limit | 0006 D3 | I |
| 11 | `GET /me` returns the account data | CONTRACTS, 5.3 | I |
| 12 | The two "to confirm" points of DATA (`uuid` identifier; IP and browser in the session) have a recorded answer | DATA, section 8 | Rev |

### F06 `plans-api`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | INV-01: for each route with an ID, author B gets `404` when using the ID of a resource of author A; in lists, B sees nothing of A's. The test walks the route list | RNF-40 | I |
| 2 | Create, list, open, change and delete a context; only education stage and length are required | RF-11, RF-17 | I |
| 3 | Deleting a context does not delete the plans made with it; the plan keeps the copy | RF-17 | I |
| 4 | Text with "laudo" is not stored and answers `privacy_alert`; with confirmation, it is stored; the same in the plan | RF-15, scenario F2.2 | I |
| 5 | The plan list shows only the author's plans, newest first, with pagination | RF-32 | I |
| 6 | A `PATCH` with an old `revision` answers `revision_conflict` and stores nothing | CONTRACTS, section 2 | I |
| 7 | A field with a pending suggestion refuses `PATCH`; after the decision, it accepts it | RF-23, RF-24 | I |
| 8 | Accepting, editing and rejecting a suggestion change the plan as in the domain; deciding again answers `409` | RF-23 | I |
| 9 | INV-02: a code outside the catalog answers `skill_not_found`; inserting a non-existent skill directly into the database fails on the foreign key | RF-41, scenario F4.2 | I |
| 10 | Approving with a `block` finding answers `plan_not_approvable` with the findings; with no block, it creates the version | RF-25, INV-03 | I |
| 11 | An approved plan refuses editing; `reopen` returns a draft with the next version; the approved version stays the same | RF-25 | I |
| 12 | As `docenda_app`, changing a row of `lesson_plan_version` fails | DATA, section 4 | I |
| 13 | INV-09: the function that loads the generation input returns no field of `plan_identification` | RNF-45 | I |
| 14 | Changing the identification of an approved plan changes neither the state nor the version | DATA, D3 | I |
| 15 | Duplicating copies accepted content and skills; it does not copy identification or pending suggestions | RF-32 | I |
| 16 | Deleting the plan deletes versions, suggestions, generations and identification | RF-32 | I |
| 17 | Two copies of the API, pointing to the same database, serve the same session | RNF-33 | I |

In this slice, suggestions come in as test data; the real generation arrives in F07.

### F07 `generation-queue`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | Requesting the generation answers `202` and leaves, in the same transaction, the `queued` generation, the reservation and the job. If sending to the queue fails, none of these exists | 0008 D3 | I |
| 2 | INV-05: two simultaneous requests for the same plan return the same `id` and create only one generation | RNF-37 | I |
| 3 | INV-11: the job contains IDs only | 0008 D6 | U |
| 4 | INV-04: the worker, with the fake adapter, ends in `succeeded` with pending suggestions; `lesson_plan.content` does not change | RF-23 | I |
| 5 | The `stage` field goes through the four steps, in order | RNF-04 | I |
| 6 | On finishing, the settlement returns the difference between the reservation and the actual cost; repeating the settlement does not change the balance | 0008 D3, DATA D10 | I |
| 7 | INV-06: with system balance for one generation, two simultaneous requests for different plans let exactly one through | RF-72 | I |
| 8 | Reaching the month's cap answers `ai_budget_reached`; listing, editing, approving and exporting keep working | RF-72, scenario F7.1 | I |
| 9 | Turning off the AI by command applies on the next request, with no restart: `ai_disabled` | RF-75 | I |
| 10 | INV-08 and INV-07: an active generation past its deadline is closed as `worker_lost`; the whole reservation stays as cost; the plan accepts a new generation | 0008, note of 16/11 | I |
| 11 | With the fake provider down, the circuit breaker opens and the following requests fail immediately with `provider_unavailable`, with no charge and no storm of calls | RNF-36 | I |
| 12 | A known error closes the generation as `failed` without retrying the job; a job that runs out of retries goes to the failure queue, with IDs only | 0008 D4, D6 | I |
| 13 | An account without a confirmed e-mail gets `email_not_verified` | RF-70 | I |
| 14 | Stopping the worker does not take down the API; generation requests stay `queued` | RNF-34 | I |

In this slice, the reservation is checked against the system cap. Per-person credit and the per-day and per-minute limits come in F11, on the same tables.

### F08 `web-core`

| # | Criterion | Origin | Test |
|---|---|---|---|
| 1 | Scenarios F1.1 and F1.2 | RF-03 | E2E |
| 2 | Scenarios F2.1 and F2.2: class-hour conversion with the choice of effective time; privacy alert with nothing sent to the AI | RF-12, RF-13, RF-15 | E2E |
| 3 | Scenarios F3.1, F3.2 and F3.3, with the fake adapter: fields as suggestions, origin marked and installation alert | RF-20, RF-21, RF-14 | E2E |
| 4 | Scenarios F4.1 and F4.2: skills with literal text; non-existent code refused | RF-41 | E2E |
| 5 | Scenarios F6.1 and F6.2: exports the approved plan; a draft with no approved version does not export | RF-60 | E2E |
| 6 | The exported file follows the sections of the internship script, opens without internet and carries the AI note, which the author can turn off. Special characters in the text do not turn into formatting | RF-60, RF-61 | I |
| 7 | Every screen of the flow fits in 360 px without horizontal scrolling | RNF-02 | E2E |
| 8 | The whole flow (log in, create, generate, decide, approve, export) is done with the keyboard alone | RNF-03 | E2E |
| 9 | No screen has an axe violation; a form error is tied to its field and focus goes to the first error | RNF-03, 0013 D4 | E2E |
| 10 | Generation progress is announced to the screen reader; each of the nine failure reasons has its own message | RNF-04, DOMAIN 4.2 | E2E, U |
| 11 | A session that expires in the middle of editing opens the login dialog; after login, the save is repeated and the typed text stays on screen | RF-04, RNF-05 | E2E |
| 12 | After using the editor, `localStorage` and `sessionStorage` hold no plan text | 0013 D7 | E2E |
| 13 | Pages load without CSP violations, with no inline style or script | 0006, 0013 D5 | E2E |
| 14 | All screen text comes from the single text module | 0013 D6 | CI |
| 15 | The generation screen warns that the text is processed by an AI provider outside Brazil | RNF-21 | E2E |

The look follows [DESIGN](DESIGN.md).

## M2 (minimum; closed in the specification)

### F09 `guardrails`
- The guardrails set meets the targets of [EVALUATION](EVALUATION.md), section 6: no student data without an alert, no legitimate sensitive theme blocked, no injection that changes behaviour (`Eval`).
- Automatic blocking only in the two unambiguous categories; in the other severe ones, the request is refused, the occurrence recorded and the administrator decides (RF-91; `I`).
- A second mild occurrence within 30 days blocks; the first only warns (RF-92, scenario F7.2; `I`).
- An off-topic request and a provider refusal create no occurrence (RF-95, scenario F7.3; `I`).
- A response that fails the `output_check` does not appear to the author (RF-90; `I`).
- With the AI blocked, viewing, editing, exporting and deleting the account keep working; the user sees the reason and asks for a review (RF-91, RF-94; `E2E`).

### F10 `invites-email`
- A single-use invitation, valid for 7 days, tied to the invited e-mail; the token is stored only as a hash (RF-78; `I`).
- At most 10 active accounts, the value in configuration (RF-78; `I`).
- The invitation e-mail is deleted on acceptance, on cancellation and on expiry (REQ-17; `I`).
- Acceptance of the terms stored with date and version (RF-01; `I`).
- Password recovery: a 1-hour, single-use link; the message does not reveal whether the e-mail exists; changing the password ends the other sessions (RF-05, scenario F1.3; `I`, `E2E`).
- An e-mail change only applies after confirmation at the new address (RF-06; `I`).

### F11 `ai-credit`
- The month's credit comes in only once, and the rollover follows Brasília time: test on 31/10 at 23:59 and on 01/11 at 0:00 Brasília time (RF-71; `I`).
- INV-06: the balance never goes negative, with simultaneous requests from the same person (`I`).
- The sixth generation of the day and the third in the same minute answer `ai_rate_limited` (RF-71; `I`).
- Unused credit does not carry over to the next month (RF-71; `I`).
- The screen shows the month's balance (RF-71; `E2E`).

### F12 `admin`
- Administrator routes answer `403` to an author and never return plan or context content (REQ-16; `I`).
- INV-14: as `docenda_app`, changing or deleting `audit_event` fails (`I`).
- Each REQ-22 event creates an audit row with IDs only (`I`).
- One access record per successful login; the retention job deletes what is older than 6 months, with its own role (0010 D5, D6; `I`).
- The three e-mail alerts go out once each: 80% of the cap, circuit breaker open and a job in the failure queue (0010 D7; `I`).
- Changing a setting is recorded in the audit and applies on the next request (RF-75; `I`).

### F13 `sources`
- The link check refuses private, local and metadata addresses, also after a redirect and when the name resolves to an internal address (RNF-43; `I`).
- Only `https`; maximum time and size; only the status is read (RNF-43; `I`).
- `source-to-review`: a source comes in as "to review" and blocks approval until the author decides (RF-51, scenario F5.1; `I`).
- A domain outside the list asks for a justification (RF-53, scenario F5.2; `E2E`).
- ABNT reference with "Acesso em" (RF-54; `U`).

### F14 `account-lifecycle`
- The three deletion steps; a wrong account name deletes nothing (RF-07, scenario F8.2; `E2E`).
- Logging in within the 30 days cancels the deletion (RF-07; `I`).
- Once the period is over, no author table keeps a row of theirs, the usage records lose the link to the person and there is a receipt with no identification (section 7.1; scenario F8.1; `I`).
- An account idle for 6 months gets the warning and enters the same flow (section 7.1; `I`).

### F15 `evaluate-plan`
- The evaluation returns findings in the single format, each with an optional suggestion the author accepts or rejects (RF-26; `I`).
- A finding made on an old revision shows up as outdated (`I`).
- The evaluation reserves and settles the cost like the generation (`I`).

### F16 `quality-gate`
- Scenarios F1 to F8 pass as end-to-end tests (RNF-09).
- Each screen went through a manual review with keyboard and screen reader, recorded (RNF-03; `Rev`).
- The full evaluation was run and the result is in `evals/results/` (RNF-10; `Eval`).
- A teacher or student teacher created a lesson in under 15 minutes (RNF-01; `Rev`).
