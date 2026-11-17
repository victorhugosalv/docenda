# Interface: Docenda

> **v0.2 (17/11/2026). Everything here is planned: there is no code yet.** Each slice of the [BACKLOG](BACKLOG.md) updates this file when the screen or the route comes to exist, and changes "planned" to "done".
> Decisions: [0012](decisions/0012-transactions-repositories-and-module-layout.md) (API and error format) and [0013](decisions/0013-front-end-stack.md) (front-end).
> This file is the **map**. The detail of each route (fields, responses and errors) is in [CONTRACTS](CONTRACTS.md); the look of the screens, in [DESIGN](DESIGN.md). Paths stay in Portuguese because the teacher sees them.

## Screen map

| Screen | Path | What the author does | Slice | State |
|---|---|---|---|---|
| Log in | `/entrar` | E-mail and password; a single error message | F08 | planned |
| My plans | `/` | List from newest to oldest; new, open, duplicate, delete | F08 | planned |
| New plan, step 1: class | `/planos/novo` | Picks a saved context or describes the class; short help on each field; privacy alert | F08 | planned |
| New plan, step 2: time | `/planos/novo` | Length in minutes or class hours; picks the suggested effective time or another one | F08 | planned |
| New plan, step 3: theme and material | `/planos/novo` | Theme, subtheme and pasted material (optional); a notice that the AI processes the text | F08 | planned |
| Generation | `/planos/:id` | Follows the steps ("conferindo", "gerando", "validando"); its own message for each failure | F08 | planned |
| Plan editor | `/planos/:id` | Each field shows the AI suggestion with accept, edit and reject; skills picked from the catalog; findings beside the field; identification apart; approve. An approved plan opens read-only, with the reopen action | F08 | planned |
| Export | `/planos/:id` | Downloads the Markdown of the approved version | F08 | planned |
| My account | `/conta` | Name, e-mail, the month's AI balance, log out, delete the account | F10, F11, F14 | planned |
| Accept invitation | `/convite/:token` | Creates the account with the invited e-mail and accepts the terms | F10 | planned |
| Recover password | `/senha` | Asks for the link and sets the new password | F10 | planned |
| Administration | `/admin` | Invitations, usage and cost, occurrences and review requests, AI settings | F12 | planned |

Rules for every screen (decision 0013): usable at 360 px, with keyboard only and with a screen reader; errors tied to the field and announced; no plan text stored in the browser.

## API routes (`/api/v1`)

All of them require a session, except the authentication ones. Another author's resource answers `404`. Errors follow the problem format (RFC 9457), with `code` and the request number.

| Method | Route | Input | Output | Slice | State |
|---|---|---|---|---|---|
| `GET` | `/healthz`, `/readyz` (outside `/api/v1`) | — | Alive; ready | F00 | planned |
| — | `/api/auth/*` | Better Auth routes: log in, log out, session | — | F05 | planned |
| `GET` | `/me` | — | Account data and whether the AI is blocked | F05 | planned |
| `GET` | `/skills?stage=&grade=` | Education stage and grade | Catalog skills | F01 | planned |
| `GET`, `POST` | `/class-contexts` | Class context | List; created context | F06 | planned |
| `GET`, `PATCH`, `DELETE` | `/class-contexts/:id` | Changed fields | Context | F06 | planned |
| `GET`, `POST` | `/plans` | Context, time, theme, material | List; draft plan | F06 | planned |
| `GET`, `PATCH`, `DELETE` | `/plans/:id` | Fields edited by the author, with the revision number | Plan, pending suggestions and findings | F06 | planned |
| `PUT` | `/plans/:id/identification` | School, teacher, class, date and shift | Plan | F06 | planned |
| `POST`, `DELETE` | `/plans/:id/skills`, `/plans/:id/skills/:code` | Skill code | Plan; a code outside the catalog is refused | F06 | planned |
| `POST` | `/plans/:id/reopen` | — | Plan back to draft, in the next version | F06 | planned |
| `POST` | `/plans/:id/duplicate` | — | New draft plan | F06 | planned |
| `POST` | `/plans/:id/suggestions/:sid/decision` | `accept`, `edit` (with the text) or `reject` | Updated plan | F06 | planned |
| `POST` | `/plans/:id/approve` | — | Approved plan and the version; `409` with the blocking findings | F06 | planned |
| `POST` | `/plans/:id/generations` | — | `202` with the generation number; the same generation on a second click | F07 | planned |
| `GET` | `/plans/:id/generations/:gid` | — | State, step and failure reason | F07 | planned |
| `POST` | `/plans/:id/exports` | Approved version and format | File to download; records the export event | F08 | planned |
| `GET` | `/me/ai-credit` | — | The month's balance | F11 | planned |
| several | `/admin/*` | Administrator role (2FA comes in before publishing, 0009 D10) | Invitations, usage, occurrences, settings | F12 | planned |

## Shared data

The request and response schemas live in `packages/contracts` (Zod) and apply to the API and to the screen. The plan fields are in [DOMAIN](DOMAIN.md), section 3, and the exact format in [CONTRACTS](CONTRACTS.md), section 4.
