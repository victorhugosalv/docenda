# Requirements: Docenda

> **v0.3 (16/11/2026).** Went through the independent review on 16/11; what changed is in section 9. Sibling documents: [DOMAIN](DOMAIN.md) (entities, states and invariants), [AI](AI.md) (pipeline, prompts and evaluation), [BACKLOG](BACKLOG.md) (milestones and slices) and [ROADMAP](ROADMAP.md) (after v1).
> Based on: [PURPOSE.md](PURPOSE.md), study notes on the references (private, with no personal data) and a checklist for LGPD and privacy by design.

## 0. How to read
- **Must** = mandatory. **Should** = desirable; it is the first to go if time runs short.
- **Phase:** `v1` = first published version. `later` = following versions. The v1 date is in open point 1. v1 is delivered in milestones (M0 to M2) and `later` in M3 to M5; the order is in the [BACKLOG](BACKLOG.md) and the [ROADMAP](ROADMAP.md).
- **Verification:** `U` unit test, `I` integration test, `E2E` end-to-end test (BDD scenario), `Eval` evaluation of AI-generated content, `Rev` manual review. The whole system follows TDD; only the AI **content** is measured by evals.
- **Personal data?** says whether the requirement touches personal data. The detail is in section 7.
- **General principle (section 5.4):** everything is modular and decoupled. No feature depends directly on a specific external service.

## 1. Actors
| Actor | Who | Phase |
|---|---|---|
| Student teacher | A teacher-education student doing an internship; creates plans | v1 |
| Teacher | A basic-education teacher; creates and evaluates their own plans | v1 |
| Reviewer | Invited by the student teacher: the university advisor or the school supervisor | later |
| Administrator | Maintains the system: invites (in v1, sign-up is by invitation only), limits, spending cap, turning the AI on and off, reviewing misuse blocks | v1 |

"Author" = student teacher or teacher, the owner of the plan.

## 2. Glossary
- **Lesson plan:** a document for one or more lessons, in a template (the internship script or ENADE). Stored in one internal format and exported to the template.
- **Production step:** simplified script → plan → slides → standardized script. Each one only starts once the previous one is approved by the author.
- **Education stage:** early childhood education, elementary school (early and final years) or high school. "Etapa" alone is not used in this document.
- **Plan identification:** school, teacher, class, date and shift. It is optional, kept apart from the content and never sent to the AI.
- **Class context:** an **aggregate** description of the class and the school. Never of a single student.
- **Nominal length:** the official length of the lesson (e.g. 50 min or 1 class hour).
- **Effective time:** the time the plan is made for, after discounting lateness, changing rooms, turning on equipment (e.g. 40 or 45 min in a 50-minute lesson).
- **Class hour:** the school's unit; 50 minutes by default, adjustable by the author.
- **Skill:** an item of the BNCC Computing supplement, cited with its code and literal text.
- **Verified source:** a source from an accepted domain, with a link that responds, approved by the author.
- **AI suggestion:** the AI's proposal for a plan field, not yet accepted, edited or rejected by the author. The AI never writes directly into the plan.
- **Finding:** the result of a rule (section 4) or of the plan evaluation. It may block approval, warn or only inform.
- **Misuse:** an attempt to use the AI outside its purpose or against the rules (section 3.9).
- **Anonymize:** remove from the data any reasonable possibility of linking it to a person (LGPD, art. 5º, XI). Anonymized data is no longer personal data (art. 12).

## 3. Functional requirements

### 3.1 Account and access
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-01 | The system must allow sign-up with name, e-mail and password, with acceptance of the terms. In v1, by invitation only (RF-78) | Without a valid invitation, sign-up is refused; opening sign-up to everyone is a setting, with no code change. Password of 8 to 128 characters, with no composition rules and not on the list of banned passwords (decision 0003); acceptance stored with the date and version of the terms | Yes | Must | v1 | U, E2E |
| RF-02 | The system must confirm the e-mail with a link that expires in 24 h | An account only uses the AI after confirmation; an expired link does not confirm and can be resent. An account created from an invitation is born confirmed, because the link reached that e-mail | Yes | Must | v1 | I, E2E |
| RF-03 | The system must allow logging in with e-mail and password | The same message for any failure ("E-mail ou senha incorretos"), without saying whether the e-mail exists; at most 3 attempts every 10 s per IP (decision 0006); a per-account limit after v1 | Yes | Must | v1 | U, I, E2E |
| RF-04 | The system must keep the session (see RNF-07) and end it after 30 min without use; the user can log out | After 30 min idle, the next action asks for login; on logout, the session is deleted on the server | Yes | Must | v1 | I |
| RF-05 | The user must be able to recover the password by automatic e-mail, with a single-use link that expires in 1 h | A used or expired link does not work; the message does not reveal whether the e-mail exists; changing the password ends the other sessions | Yes | Must | v1 | I, E2E |
| RF-06 | The user must be able to see and correct their name and e-mail | Changing the e-mail requires confirmation at the new address; the change is recorded in the log | Yes | Must | v1 | E2E |
| RF-07 | The user must be able to delete the account and the data (see section 7.1) | Confirmation in 3 steps: password, typing the account name and clicking the link sent by e-mail. The account stays deactivated for 30 days; logging in within that period cancels the deletion; afterwards, the data follows section 7.1 | Yes | Must | v1 | I, E2E |
| RF-08 | The user should be able to download all their data in one file (portability) | JSON file with account, contexts and plans | Yes | Should | later | I |
| RF-09 | The user should be able to turn on two-factor authentication (authenticator app code) | With 2FA on, login asks for the code; recovery codes generated once | Yes | Should | later | I, E2E |
| RF-10 | The user should be able to log in with Google and other providers, and link those logins to the same account | The same account, whether by password or by provider; linking only with a confirmed e-mail | Yes | Should | later | I, E2E |

### 3.2 Class context and time
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-11 | The author must be able to describe the class in aggregate form: stage and grade, subject, class size, prior knowledge, common difficulties, social context, school resources, accessibility in general terms, the week's workload (e.g. exam week) and the teacher's opinion | Only stage and length are required; each field has short help saying "sem nomes, diagnósticos ou laudos" | No (if the warning is followed) | Must | v1 | E2E |
| RF-12 | The lesson length must be required and given in hours and minutes or in class hours | 1 class hour = 50 min by default, adjustable; the system shows the conversion | No | Must | v1 | U, E2E |
| RF-13 | The system must offer to plan for an effective time shorter than the nominal length | It suggests a discount (e.g. 50 → 40 or 45 min) and explains why; the author picks the value or declines | No | Must | v1 | U, E2E |
| RF-14 | The system must warn when the plan has activities that take a lot of time (installing software, creating an account on a platform, configuring equipment) and the effective time cannot fit other topics | The warning suggests the lesson focus on that activity only; the author decides | No | Must | v1 | U, Eval |
| RF-15 | The system must warn when a text seems to contain a student's name, a diagnosis or a medical report, before saving or sending to the AI | Test-set sentences with terms like "laudo", "CID", "diagnosticado" raise an alert, also in English, Spanish, mixed language and disguised text (decision 0005, D8); the author fixes it or confirms that it identifies nobody | Yes (it is the protection) | Must | v1 | U, E2E |
| RF-16 | The author should be able to reuse a class context across several plans | A saved context shows up when creating the next plan | No | Should | v1 | E2E |
| RF-17 | The author must be able to edit and delete a class context | Deleting a context does not delete the plans made with it; the plan keeps a copy of the context it used | No | Must | v1 | I, E2E |

### 3.3 Lesson plan
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-20 | The author must be able to create the plan for one lesson from a theme and a context, in the internship script template | Every template field filled as an AI suggestion, within the effective time | No | Must | v1 | E2E, Eval |
| RF-21 | The author must be able to create the plan from material they already have, pasted as text (mini script, slide text) | The plan reuses the material and marks what the AI added | No | Must | v1 | E2E, Eval |
| RF-22 | The author should be able to upload files (slides, PDF, DOCX) to create the plan | A warning about student data before uploading; the file is deleted after the text is extracted; only the text goes to the AI | Possibly | Should | later | I, E2E |
| RF-23 | Every field generated by the AI must stay marked as a suggestion until the author accepts, edits or rejects it | The unit is the plan field; skills and sources are decided item by item. A plan with a pending suggestion cannot be approved. A new generation never erases what the author has already decided | No | Must | v1 | U, E2E |
| RF-24 | The author must be able to edit any field of the plan | The edit is saved; the excerpt stops being a "suggestion" | No | Must | v1 | E2E |
| RF-25 | The plan must follow the states draft → approved. Only the author approves. Exporting is an event, not a state; "in review" comes with reviewers (RF-80) | The AI never changes the state to approved. Editing an approved plan takes it back to draft and keeps the approved version, which stays exportable | No | Must | v1 | U, I |
| RF-26 | The author should be able to request an evaluation of the plan: automatic checks (section 4) + AI evaluation against the rubric | Result as a list of points, each with a suggestion the author accepts or rejects | No | Should | v1 | I, Eval |
| RF-27 | The plan must have a field for links to other subjects, described in text | The link does not use skills from other BNCC components | No | Must | v1 | U |
| RF-28 | The author should be able to adapt a finished plan to a new context (e.g. exam week, an unexpected event) | Creates a new version; the previous one is kept | No | Should | later | E2E, Eval |
| RF-29 | The system should generate the other steps (simplified script, slides, standardized script, study guide), each after the previous one is approved | The next step is locked until approval | No | Should | later | I, Eval |
| RF-30 | The system should create plans for a week, month, semester and year | A sequence of lessons consistent with the workload | No | Should | later | Eval |
| RF-31 | The system should export to the ENADE templates (EF/EM and EI) | Official fields filled | No | Should | later | I |
| RF-32 | The author must be able to list, open, duplicate and delete their plans | The list shows only the author's plans, newest first; deleting asks for confirmation and also deletes the plan's versions, suggestions and generations | No | Must | v1 | I, E2E |
| RF-33 | The author should be able to ask the AI for a new suggestion for a single field, with a short instruction (e.g. "avaliação mais simples") | The new suggestion applies only to that field and goes through the same checks | No | Should | later | E2E, Eval |

### 3.4 Skills (BNCC Computing)
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-40 | The system must have the BNCC Computing catalog imported once, checked and versioned | 140 codes; the text of each equal to the official document | No | Must | v1 | U, Rev |
| RF-41 | The plan must only accept skills from the catalog, in the format `(CÓDIGO) Texto literal. (Competência específica N)` | A non-existent code or altered text is blocked with a clear message | No | Must | v1 | U, E2E |
| RF-42 | The AI must suggest skills by consulting the catalog (official explanation and examples), not the model's memory | Every suggested skill exists in the catalog | No | Must | v1 | I, Eval |
| RF-43 | The system must warn when the skill does not match the class's stage | E.g. an EM13 code in an elementary class raises a warning | No | Must | v1 | U |

### 3.5 Sources
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-50 | A source suggested by the AI can only come from a real search, restricted to the list of accepted domains (government, institutions, encyclopedias, museums, publishers, journals). In v1 the AI does not suggest sources: the author brings the source (RF-53) | A domain outside the list is discarded before it appears; no link comes from the model's memory | No | Must (once there are suggestions) | later | U, I, Eval |
| RF-51 | Every source comes in as "to review" and only goes into the plan after the author approves it | A plan with a source "to review" cannot be approved | No | Must | v1 | U, E2E |
| RF-52 | The system must check that the source link responds before showing it | A broken link is discarded or flagged | No | Must | v1 | I |
| RF-53 | A source brought by the author goes through the domain list and the link check | A domain outside the list raises a warning, and the author may keep it with a justification; a link that does not respond is flagged; the check follows RNF-43 | No | Must | v1 | E2E |
| RF-54 | References should come out in ABNT format, with "Acesso em" | Format checked in the tests | No | Should | v1 | U |
| RF-55 | The system should check that the cited fact is written in the source and show the excerpt | Excerpt shown to the author; any mismatch recorded | No | Should | later | Eval |

### 3.6 Export
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-60 | The author must be able to export the approved plan as Markdown in v1; PDF if time allows | The file opens without internet and follows the sections of the internship script | Yes (author's name, if filled in) | Must | v1 | I, E2E |
| RF-61 | The exported document should state that it was prepared with AI support | A short footer note; the author can turn it off | No | Should | v1 | I |

### 3.7 AI, limits and cost
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-70 | Only an authenticated user with a confirmed e-mail must be able to call the AI | A call without a valid session is refused | No | Must | v1 | I |
| RF-71 | Each user must have a monthly AI credit and a usage limit per day and per minute | In v1, US$ 1 per month (configurable), renewed on day 1 in Brasília time, not accumulating; the user sees the month's balance; the cost is reserved at request time (decision 0008). With no balance or over the limit, the AI refuses with a clear message and everything else works | Yes (usage per person) | Must | v1 | U, I |
| RF-72 | The system must have a monthly AI spending cap | On reaching the cap, the AI turns off for everyone and everything else works; the administrator is warned at 80%. In v1, US$ 10 (10 people × US$ 1) | No | Must | v1 | U, I |
| RF-73 | Each call must have a maximum input and output size | Input that is too large is refused before it reaches the provider | No | Must | v1 | U |
| RF-74 | The system must record AI usage per user and per operation (tokens and estimated cost), without the content | A record exists for each call; the basis of the cost study | Yes | Must | v1 | I |
| RF-75 | The administrator must be able to turn the AI on and off without a new deploy | The change applies on the next call | No | Must | v1 | I |
| RF-76 | The user should be able to use their own API key | Key stored encrypted, never in a log, with an option to delete it; only tested providers | Yes (secret) | Should | later | I, Eval |
| RF-77 | The system should allow free models | Only providers whose terms do not use the data for training | No | Should | later | Rev |
| RF-78 | The administrator must be able to invite people by e-mail; in v1, sign-up exists only by invitation | A single-use link that expires in 7 days; the account is only created with the invited e-mail; the administrator can cancel the invitation; at most 10 active accounts (configurable); the link token is stored only as a hash; only people who asked receive an invitation | Yes (invitee's e-mail) | Must | v1 | I, E2E |
| RF-79 | The first administrator must be created by a command on the server, never by a route | There is no public route that creates or promotes an administrator; the creation is in the audit log | Yes | Must | v1 | I |

### 3.8 Reviewers, evaluation and consent
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-80 | The student teacher should be able to invite reviewers (university advisor, school supervisor) to a plan | A reviewer only sees what was shared | Yes | Should | later | I, E2E |
| RF-81 | The reviewer should be able to evaluate the plan with comments and a grade per criterion | Evaluation separated by role | Yes | Should | later | E2E |
| RF-82 | The student teacher should be able to remove a reviewer's access | Access disappears immediately | Yes | Should | later | I |
| RF-83 | Script and slides only reach the reviewer if the student teacher sends them | Sending is always optional | No | Should | later | E2E |
| RF-84 | The teacher should be able to self-evaluate the plan after the lesson | Self-evaluation linked to the plan version | No | Should | later | E2E |
| RF-85 | The reviewer should be able to build their own evaluation rubric | Metrics defined by the reviewer | No | Should | later | E2E |
| RF-86 | Plans and evaluations may only be used to improve the system with separate, optional consent that is in force | Before the first evaluation, a screen explains the use and asks for acceptance; an easy-to-reach button in the settings turns it on and off with one click; when turned off, nothing new comes in and whatever was not anonymized leaves the set | Yes | Must (once there is evaluation) | later | I, E2E, Rev |

### 3.9 AI misuse
| ID | Requirement | Acceptance criterion | Personal data? | Priority | Phase | Verification |
|---|---|---|---|---|---|---|
| RF-90 | The system must check every request to the AI (and the pasted text) before sending it, and every response before showing it | Categories in section 4.1; a test set with examples of each category, also in English, Spanish, mixed language and disguised text; a response that fails the check does not appear (decision 0005, D8) | Possibly | Must | v1 | U, Eval |
| RF-91 | **Severe** misuse must block the user's AI use and notify the user and the administrator | Automatic blocking only in the unambiguous categories of section 4.1; in the others, the request is refused, the occurrence is recorded and the administrator decides on the block. The user sees the reason and how to ask for a review; the rest of the account (viewing, exporting, deleting data) remains accessible | Yes | Must | v1 | I, E2E |
| RF-92 | **Mild** misuse must raise a warning saying that the next occurrence blocks the AI until an administrator reviews it | A second mild occurrence within 30 days blocks; the administrator is notified | Yes | Must | v1 | U, E2E |
| RF-93 | The administrator must be able to see each occurrence, unblock or keep the block, with a justification | Decision recorded in the audit log | Yes | Must | v1 | E2E |
| RF-94 | The user must be able to ask for a review of a block and know the criterion used (LGPD, art. 20) | The request reaches the administrator; the answer is recorded | Yes | Must | v1 | E2E |
| RF-95 | A request that is merely outside the purpose (e.g. a subject unrelated to a lesson) must be refused with an explanation, without counting as misuse | A polite refusal; no occurrence recorded | No | Must | v1 | U, Eval |

## 4. Domain rules (condition → action)
| Condition | Action | Phase |
|---|---|---|
| Sum of the activities' times ≠ the chosen effective time | Block approval and show the difference | v1 |
| Length given in class hours | Convert (50 min per class hour, or the author's value) and show it | v1 |
| Nominal length given | Suggest a shorter effective time (e.g. 50 → 40 or 45 min); the author decides | v1 |
| Plan with installation, account creation or configuration and little effective time | Warn that the lesson should focus on that alone | v1 |
| Week marked as "exam week" | The AI suggests lighter activities; the author decides | v1 |
| Skill outside the catalog or with altered text | Block (RF-41) | v1 |
| Source "to review" or pending AI suggestion | Block approval (RF-23, RF-51) | v1 |
| Text with signs of a student's name, a diagnosis or a medical report | Warn before saving or sending (RF-15) | v1 |
| ENADE template, final years or high school, more than 3 class hours | Block | later |
| ENADE template, early years or early childhood, more than 5 class hours | Block | later |
| Sending to the supervisor less than 10 calendar days before the teaching session | Warn | later |

### 4.1 Misuse severity (proposal for review)
| Severity | Examples | Action |
|---|---|---|
| Severe, automatic block | Asking for sexual content involving minors; trying to extract keys or other users' data | Immediate AI block + notice to the user and the administrator (RF-91) |
| Severe, with review | Asking the AI to produce illegal content or hateful text against a person or a group | Request refused, occurrence recorded and the administrator notified, who decides on the block (RF-91, RF-93) |
| Mild | Trying to make the AI ignore the rules (e.g. "invent a skill"); trying to obtain the internal instructions, which hold no secrets (decision 0005, LLM07); offensive content with no target; insisting on getting around the usage limit | Warning; the next one within 30 days blocks until review (RF-92) |
| Not misuse | Off-topic request; typo; text with student data; **a lesson whose theme is** cyberbullying, hate speech, scams, misinformation or digital safety; a refusal from the AI provider itself | Explained refusal (RF-95), privacy alert (RF-15) or generation failure message, with no occurrence |

The automatic check is sometimes wrong (false positive). That is why there is administrator review and a review request by the user. The risk is higher here than in other systems: digital culture and safety are content of BNCC Computing itself, so the test set includes lessons on these themes and none of them may be blocked ([AI](AI.md), section 7).

## 5. Non-functional requirements

### 5.1 Product
| ID | Requirement | Acceptance criterion |
|---|---|---|
| RNF-01 | Usability: a teacher must create a lesson adapted to the class in under 15 minutes | Test with at least 1 teacher or student teacher |
| RNF-02 | The system must work on a phone | Main screens usable on a 360 px screen |
| RNF-03 | Accessibility: a blind person must be able to use everything with a screen reader, on their own | Follows WCAG 2.2 level AA; fields with clear labels and descriptions; logical navigation order; errors announced; a real test with a screen reader. Details in the front-end topic |
| RNF-04 | Generation should show progress and finish within 60 s | Measured on real calls |
| RNF-05 | An AI failure cannot lose the author's work | Draft saved before each call |
| RNF-06 | Application security follows OWASP ASVS 5.0 as a checklist | Level 1 items checked before publishing; HTTPS everywhere; password only as a strong hash; secrets outside the code |
| RNF-07 | Session stored on the server, identified by a protected cookie (HttpOnly, SameSite; Secure on HTTPS); a new identifier on every login; revocable (logout, password change, log out of all devices, suspension, deletion) | Test: a revoked session is refused on the next request. Decision 0003 |
| RNF-08 | AI security follows the OWASP Top 10 for LLM applications (2025) | User text, pasted material and source pages are treated as **data**, never as instructions (indirect injection); consumption limits (RF-71 to RF-73) |
| RNF-09 | Quality: TDD across the whole system, with unit, integration and e2e tests running on every change (CI) | Every rule in section 4 has a unit test with edge cases; every v1 BDD scenario has an e2e test |
| RNF-10 | AI quality: evals with a rubric and a set of examples | No invented skill or source in the evaluation set |

### 5.2 Organizational
| ID | Requirement | Acceptance criterion |
|---|---|---|
| RNF-11 | Node.js with TypeScript; packaged in Docker | Starts locally with one command |
| RNF-12 | Deploy on AWS at low cost | Monthly target defined in the infra (AWS) topic |
| RNF-13 | In v1 the AI uses the project's key, with limits and a cap (RF-71 to RF-73) | Values defined and configurable |
| RNF-14 | The usage data (RF-74) must allow computing the cost per plan | A cost report per operation; the basis of the future subscription study |

### 5.3 External (legal and ethical)
| ID | Requirement | Acceptance criterion |
|---|---|---|
| RNF-20 | Comply with the LGPD (Law 13.709/2018) | Section 7 filled in and reviewed |
| RNF-21 | Transparency: the user knows what came from the AI, which provider processes the data and for what | Notice in the terms and on the generation screen |
| RNF-22 | The AI provider cannot use the data sent for training | Provider terms checked and recorded in a decision |
| RNF-23 | Respect the copyright of sources: cite, without copying long excerpts | Review in the evals |
| RNF-24 | Backups kept for a limited time; deleted data disappears from backups when they expire | Period defined in the infra topic (proposal: 7 days) |

### 5.4 Decoupling, reuse and scale
| ID | Requirement | Acceptance criterion |
|---|---|---|
| RNF-30 | Every external service (AI, e-mail, social login, file storage, database, future payment) sits behind its own interface, with swappable adapters | Each interface has at least 2 implementations (a real one and a fake one for tests); switching provider means changing configuration and adapter, without touching the system's rules. Exception: the database has an interface but no fake adapter; it is tested against the real PostgreSQL (decision 0012) |
| RNF-31 | Authentication, AI limits and cost, misuse checking, audit and export are independent modules, reusable in other projects | None of these modules depends on the plans module; each has its own tests and usage documentation |
| RNF-32 | Configuration per environment (variables), with no secret or fixed address in the code | The same code runs locally, in test and in production, changing only the configuration |
| RNF-33 | The server keeps no in-memory state between requests | Two copies running at the same time serve the same user without error |
| RNF-34 | Long tasks (AI generation, export) can run in a queue, separate from serving the screens | The screen does not freeze during generation; a failure in the queue does not take down the site |
| RNF-35 | The AI response comes in a structured format (JSON with a schema) and is validated before use | An invalid response is retried at most 2 times and then becomes a clear error |
| RNF-36 | Calls to external services (AI, e-mail) retry with growing, random waits, and stop for a while when the service is down (circuit breaker) | Test with a fake provider that is down: no storm of calls |
| RNF-37 | The same request repeated in sequence (e.g. a double click) does not generate or charge again | Idempotency test |
| RNF-38 | Each AI task uses the right model: cheap for checks and classification, better for generation | Model per task defined in configuration |
| RNF-39 | Observability: cost, response time and quality per model and per operation; logs without password, token or sensitive content | A simple dashboard or report; a test that looks for secrets in the logs |
| RNF-40 | Non-sequential IDs in URLs and an owner check on every resource | Test on every route: user A cannot access B's resource even by changing the ID, and the answer is "not found", without revealing that the resource exists (decision 0012) |
| RNF-41 | Dependencies monitored (vulnerability alerts) and static security analysis in CI | CI fails on a known severe vulnerability |
| RNF-42 | Money and cost values stored as integers (nano-dollar: US$ 1 = 1,000,000,000), never as floating point | A sum test with no rounding error; the per-token price of the cheapest model is an integer in this unit |
| RNF-43 | When the server opens a link given by someone (source check), it cannot be used to reach internal addresses (SSRF) | HTTPS only; private, local and cloud-metadata addresses refused after resolving the name; redirects checked; maximum time and size. Test with internal addresses (decision 0006) |
| RNF-44 | Every request has a maximum size, and there is a general request limit per IP and per session, besides the login limit | A request that is too large or too frequent is refused with a clear message (decision 0006) |
| RNF-45 | The plan identification and the account data never enter a request to the AI | Test with decoy values: name, e-mail, school and class do not appear in any call (decision 0005) |

## 6. Stories and scenarios (v1)
Each scenario becomes an acceptance (e2e) test.

**F1. Log in**
As an author, I want to log in with e-mail and password, so I can reach my plans securely.
- *Scenario 1:* Given I have a confirmed account, when I enter the correct e-mail and password, then I land on the list of my plans.
- *Scenario 2:* Given I have already tried to log in 3 times in 10 seconds, when I try again, then the system asks me to wait, and the error message does not say whether the e-mail exists.
- *Scenario 3:* Given I forgot my password, when I ask for recovery, then I get a link by e-mail that is valid for 1 hour and only once.

**F2. Describe the class and the time**
As an author, I want to describe my class and the lesson time, so the plan fits reality.
- *Scenario 1:* Given I entered 1 class hour, when I move on to the plan, then the system shows 50 minutes and suggests planning for 40 or 45, and I choose.
- *Scenario 2:* Given I wrote "aluno com laudo de TDAH", when I try to save, then I see an alert asking me to describe the class without a diagnosis, and nothing is sent to the AI.

**F3. Create the plan**
As a student teacher, I want to generate a plan from the theme and the context, so I can start from something ready and review it.
- *Scenario 1:* Given I chose a theme and a context, when I ask for the plan, then I get every field of the internship script marked as an AI suggestion, adding up to the effective time.
- *Scenario 2:* Given I pasted my mini script, when I ask for the plan, then the plan reuses my text and marks only what the AI added.
- *Scenario 3:* Given the plan includes installing software and I have 40 effective minutes, when the plan is generated, then I see an alert suggesting the lesson be devoted to the installation alone.

**F4. Skills**
As an author, I want only real BNCC Computing skills to get in, so I make no mistake in the official document.
- *Scenario 1:* Given the AI suggested skills, when I see the list, then all of them exist in the catalog, with literal text and the competency number.
- *Scenario 2:* Given I typed a code that does not exist, when I save, then the code is refused with a clear message.

**F5. Sources**
As an author, I want to review sources before using them, so I do not cite something false.
- *Scenario 1:* Given I entered a source from an accepted domain with a working link, when I open the plan, then it appears as "to review" until I approve it.
- *Scenario 2:* Given I entered a blog link, when I save, then I see a warning that the domain is not on the list and I can only keep the source with a justification.

**F6. Export**
As an author, I want to export the approved plan, so I can hand it in or take it to class.
- *Scenario 1:* Given the plan is approved, when I export, then I get the document in the internship script template.
- *Scenario 2:* Given the plan is still a draft, when I try to export, then the final export is blocked.

**F7. AI limits and misuse**
As an administrator, I want to limit and protect AI use, so spending and abuse do not get out of control.
- *Scenario 1:* Given the monthly cap has been reached, when any user asks for a generation, then the AI is off and the rest of the system works.
- *Scenario 2:* Given a user has already received a mild warning, when they commit another mild misuse, then the AI is blocked for them until I review it, and I am notified.
- *Scenario 3:* Given a user asked for something off-topic, when the AI responds, then they get an explained refusal and no occurrence is recorded.

**F8. Delete the account**
As a user, I want to delete my account and my data, to exercise my right under the LGPD.
- *Scenario 1:* Given I confirmed my password, typed the account name and clicked the e-mail link, when the deletion finishes, then I can no longer log in and my plans no longer exist.
- *Scenario 2:* Given I typed the account name wrong, when I confirm, then nothing is deleted.

## 7. Privacy (LGPD checklist, Privacy by Design)
**Data inventory (v1):**
- **Account:** name, e-mail, password hash, acceptance of the terms (date and version).
- **Invitation:** the invitee's e-mail and date, only until acceptance, cancellation or expiry (RF-78).
- **Usage:** access records (date, time, IP) and AI usage records (no content).
- **Misuse:** occurrences with the minimal excerpt that triggered them, for review.
- **Plan and context:** class data in aggregate form. The plan identification (school, author, class) is optional and filled in by the author.
- **We do not collect:** any student data, sensitive data or children's data.

**Legal basis per processing (proposal, to be checked before publishing):**
| Data | What for | Proposed basis (LGPD, art. 7º) |
|---|---|---|
| Account (name, e-mail, password) and acceptance of the terms | Giving access to the service | Performance of a contract (V) |
| Invitation (e-mail) | Sending the invitation the person asked for | Preliminary procedures to a contract, at the data subject's request (V) |
| AI usage per person | Limit, credit and cost | Performance of a contract (V) |
| Misuse occurrences and audit | Security and abuse prevention | Legitimate interest (IX) |
| Access records | Record keeping under the Marco Civil | Legal obligation (II) when it applies; until then, legitimate interest (IX) |
| Plans and evaluations used to improve the system | Evaluation set | Consent (I), separate and optional (RF-86) |

Not every processing depends on consent: it is used only where the person can say no without losing the service.

### 7.1 Deletion: what disappears, what is anonymized and what stays
Basis: LGPD, arts. 15, 16 and 18, VI (elimination, except for the exceptions in art. 16); art. 12 (anonymized data is not personal data); Marco Civil da Internet, art. 15.

| Data | On deletion | Why |
|---|---|---|
| Name, e-mail, password, contexts, plans, sessions, API key | **Deleted** | End of processing (art. 15, III, and art. 16) |
| Reviewers' comments on the deleted plans | **Deleted** | They lose their purpose along with the plan |
| AI usage records (tokens, cost) | **Anonymized**: the link to the user is cut; only the aggregate number remains | Exclusive use by the controller, with no third-party access, with anonymized data (art. 16, IV) |
| Plans and evaluations already in the evaluation set, with consent | **Anonymized** before entering the set; whatever is not anonymized is deleted | Art. 12 and art. 16, IV; manual review of free text, because removing the name is not enough |
| Access records (date, time, IP) | **Kept for 6 months**, confidentially, and then deleted | Legal obligation (art. 16, I) for a legal entity with economic purposes (Marco Civil, art. 15). Followed from v1, so the system is ready to become a product (point 4 of section 8) |
| Audit (event type, IDs and time, no content) | **Kept for 6 months** and then deleted | Regular exercise of rights and security (arts. 7º, VI, and 46); decision 0010 |
| Minimal record of the deletion request (date and result, no name or e-mail) | **Kept** | To prove the request was fulfilled |
| Backups | Expire within the period of RNF-24 | The data disappears when the backup expires |

**Cooling-off period:** deletion is not immediate. The account stays deactivated for 30 days (the same period as X and Instagram); logging in during that period cancels the request. Afterwards, the table above applies.

**Inactive accounts:** after 6 months without use, the system warns by e-mail and the account enters the same flow: 30 days deactivated and then deleted or anonymized according to the table. Deactivating (soft delete) only hides the data; for the LGPD, what counts is deleting or anonymizing at the end of the period.

**About the confirmations:** the law asks that consent be revoked by a "procedimento gratuito e facilitado" (art. 8º, §5º) and that rights be fulfilled at no cost (art. 18, §5º). Therefore:
- **Deleting the account:** the 3 steps (password, account name and e-mail link) are there for **security**, against deletion by mistake or by someone who broke into the account. They cannot become an obstacle meant to discourage the user: the button is easy to find and the process is short.
- **Withdrawing the improvement consent (RF-86):** one click, with no extra confirmation.

### 7.2 Checklist items
| Item | Applies? | How it is met | Evidence |
|---|---|---|---|
| REQ-01 Ordinary data and purpose | Yes | Name and e-mail for the account; the invitee's e-mail only to send the invitation; optional plan identification, for the exported document | Inventory; RF-01, RF-60, RF-78 |
| REQ-02 Sensitive data | Yes, as a risk | We do not collect it. The risk is the author writing a diagnosis in free text: an alert before saving or sending | RF-15; section 4 |
| REQ-03 Purpose of each use | Yes | Account: access. Context: adapting the plan. AI usage: limit, cap and cost. Occurrences: security | RF-11, RF-74, RF-90 |
| REQ-04 Necessity and optional fields | Yes | Only stage and length are required in the context; plan identification is optional | RF-11, RF-12 |
| REQ-05 Minimization | Yes | Aggregate class; usage record without content; an occurrence keeps only the minimal excerpt; file deleted after extraction | RF-11, RF-22, RF-74, RF-90 |
| REQ-07 Children and adolescents | Yes, indirectly | The plans are for classes of children and adolescents, but none of their data is collected | RF-11, RF-15 |
| REQ-08 Extra protections | Yes | Alert for names, diagnoses and medical reports; help in the fields; nothing goes to the AI with an unresolved alert | RF-15; F2 scenario 2 |
| REQ-09 AI features | Yes | Generate plan (RF-20, RF-21), suggest skills (RF-42), evaluate (RF-26), check misuse (RF-90); suggest sources only after v1 (RF-50). Input: theme, aggregate context, pasted material. Output: suggestions | Section 3 |
| REQ-10 Human intervention | Yes | Everything from the AI is a suggestion; only the author approves; misuse blocks reviewed by an administrator | RF-23, RF-25, RF-93 |
| REQ-11 Information about the AI | Yes | Notice in the terms and on the generation screen; suggestion marking; note in the exported document | RNF-21, RF-61 |
| REQ-12 Access, correction, deletion | Yes | See and correct data; delete the account (section 7.1) | RF-06, RF-07 |
| REQ-13 Automated decisions | Yes | Rule blocks (section 4), usage limit and misuse block. The user knows the criterion and can ask for a review (art. 20) | RF-91 to RF-94 |
| REQ-14 Inferred profile | Yes, limited | The system infers nothing about students. About the user, it only counts misuse occurrences, for the repeat-offence rule | RF-92 |
| REQ-15 Authentication and recovery | Yes | Sign-up by invitation only in v1 (single-use link, 7 days); e-mail confirmation, strong password, attempt limit per IP (decision 0006), 30-min session revocable on the server, recovery by e-mail valid for 1 h; 2FA and social login later | RF-01 to RF-05, RF-09, RF-10, RF-78, RNF-07 |
| REQ-16 Who sees and changes | Yes | Each author sees only their own plans. A reviewer sees only what was shared (later). The administrator sees usage, limits and occurrences, not the content of the plans | RF-70, RF-80 to RF-82, RF-93 |
| REQ-17 Retention | Yes | Data while the account exists; access records 6 months; audit 6 months (decision 0010); misuse occurrences: proposal of 90 days; inactive accounts: warning after 6 months without use, 30 days deactivated and then deleted or anonymized; invitations: e-mail deleted on acceptance (it becomes the account), on cancellation or on expiry (7 days) | Section 7.1; RF-07, RF-78 |
| REQ-18 Disposal | Yes | Section 7.1, including backups | RF-07, RNF-24 |
| REQ-19 Anonymization | Yes | AI usage: the link to the user is cut. Evaluation set: remove author, school and class and review the free text | Section 7.1; RF-86 |
| REQ-20 Encryption | Yes | HTTPS in transit; database and backups encrypted at rest; user key encrypted (later) | RNF-06, RF-76 |
| REQ-21 Consent | Yes | Terms accepted at sign-up (date and version). Improvement: separate consent, asked before the first evaluation, withdrawn with one click | RF-01, RF-86 |
| REQ-22 Audit logs | Yes | Login, failures, attempt limit reached, account data change, deletion, plan state change, misuse occurrences and administrator decisions. Only the administrator can query them | RF-03, RF-07, RF-25, RF-93 |
| REQ-23 Portability | Yes (later) | Download all data as JSON | RF-08 |
| REQ-23b Backup | Yes | Automatic database backups, encrypted, accessible only to the administrator, expiring within the period of RNF-24 | RNF-24 |

## 8. Open points
1. ~~v1 date.~~ **Closed (14/11):** first version running locally; AWS later.
2. Values: limit per day and per minute, monthly cap, repeat-offence period (proposal: 30 days). **Initial values (15/11, decision 0005):** a cap of US$ 10 per month; per user, 5 generations per day and 2 per minute; estimated cost of US$ 0.15 to 0.20 per plan. **Update (16/11, decisions 0005 and 0008):** up to 10 invited people, each with US$ 1 of credit per month, renewed on day 1 and not accumulating; the US$ 10 cap remains for the whole system. **Final values after the test with the 3 reference plans and a minimal load test.**
3. ~~AI provider and model.~~ **Closed (15/11, decision 0005):** the Anthropic API in v1, behind the AI port (RNF-30); no training on the data (RNF-22). Claude Sonnet 5.5 to generate and evaluate; Claude Haiku 5.5 for checks. Models, effort and limits confirmed by the test with Haiku, Sonnet and Opus on the 3 reference plans. A provider's free tier that trains on the data only with fictional data. **E-mail closed in decision 0003:** Mailpit locally, Amazon SES in production.
4. ~~Access records.~~ **Closed (14/11):** kept for 6 months from v1 (section 7.1).
5. ~~Cooling-off period.~~ **Closed (14/11):** account deactivated for 30 days, cancellable; afterwards, section 7.1 (RF-07).
6. ~~Export format.~~ **Closed (14/11):** Markdown in v1; PDF if time allows (RF-60).
7. ~~Inactive accounts.~~ **Closed (14/11):** a warning after 6 months without use and the same flow as point 5 (section 7.1).
8. **Terms of use and privacy notice** (16/11): not written yet. RF-01 requires acceptance with a version, and RNF-21 requires the notice. They must exist before the first real user (ROADMAP, M3).
9. **Legal basis for each processing** (16/11): the table in section 7 is a proposal; check before publishing, along with the contact channel for the data subject.
10. **Retention of misuse occurrences:** proposal of 90 days (REQ-17), to be closed in the guardrails slice.

## 9. Changes in v0.3 (review of 16/11/2026)
| What changed | Where | Why |
|---|---|---|
| "Etapa" became "etapa de ensino" (education stage) and "etapa de produção" (production step) | Glossary | The same word had two meanings |
| Suggestion per field; the AI never writes directly into the plan | Glossary, RF-23 | It can be tested, undone and lives alongside the author's editing |
| Plan states: draft and approved; exporting is an event | RF-25 | A plan is exported many times and edited afterwards |
| New requirements: edit and delete a context, list and delete a plan, new suggestion for one field, first administrator | RF-17, RF-32, RF-33, RF-79 | Basic operations were missing |
| In v1 the AI does not suggest sources; the author brings them | RF-50, RF-53, scenarios F5 | A link coming from the model's memory is the weakest point of "nothing invented" |
| Automatic blocking only in the unambiguous categories; sensitive lesson themes are not misuse | RF-91, section 4.1 | Digital culture and safety are BNCC Computing content |
| Database without a fake adapter | RNF-30 | Decision 0012 |
| Nano-dollar as the unit | RNF-42 | A thousandth of a cent gives a fraction on the cheapest model |
| SSRF, request limits and identification kept out of the AI | RNF-43 to RNF-45 | Security and privacy gaps |
| Legal basis per processing and open points 8 to 10 | Sections 7 and 8 | Not everything is consent; the terms are missing |
| Milestones M0 to M5 | Section 0 | v1 has 40 mandatory functional requirements; it needed an order |
| Class size enters the aggregate description (17/11) | RF-11 | The rubric checks whether the activities fit the class, and that needs a number. It still identifies no student |
