# Incident response plan

> Decision: [0010](decisions/0010-logs-monitoring-and-incident-response.md). Based on: LGPD, art. 48, and CD/ANPD Resolution 15/2024 (checked on 16/11/2026). Rehearsed once before publishing.

A **security incident** is any event that compromises the confidentiality, integrity or availability of personal data. Examples: a key leaked in a commit, someone logging into an account that is not theirs or the database exposed.

What already limits the damage: Docenda stores no identifiable student data (section 7 of the requirements). A leak reaches, at most, the account data.

## The 7 steps

| # | Step | What to do | When |
|---|---|---|---|
| 1 | **Contain** | Turn off the AI (cap at zero) and suspend the accounts involved; take the application offline if the attack continues | Immediately |
| 2 | **Rotate the secrets** | `BETTER_AUTH_SECRET` (invalidates every cookie), database password, Anthropic key and e-mail credentials; a secret that went into a commit is rotated, not just removed from history | Immediately |
| 3 | **End the sessions** | Delete every session in the database; everyone logs in again | Immediately |
| 4 | **Assess the risk** | Which data, of how many people, since when. Can the incident affect the data subjects' rights, and does it involve any criterion of art. 5º (authentication credentials, sensitive data, large scale, children's data, confidentiality)? | Within 1 business day |
| 5 | **Notify** | If there is relevant risk or harm: the ANPD (form on its website) and each data subject by e-mail, in plain language, with the minimum content of art. 9º (below) | **Within 3 business days** of learning that personal data was affected |
| 6 | **Record** | Every incident, notified or not, in the private register: date, what happened, data affected, risk assessment, measures and notifications | Always; kept for **5 years** |
| 7 | **Learn** | Turn it into a test, an alert or an ADR, so it does not happen again | After it is closed |

## What the notice to data subjects must contain (art. 9º)
- What happened and which data was affected.
- The risks and the possible impacts.
- What has already been done and what the person can do (for example, change the password if it is reused on another site).
- The date the incident was discovered and, if any, the reason for the delay.
- A contact for questions.

## Where each thing lives
- **This plan:** public, in the repository.
- **Incident register:** private, outside the repository, kept for 5 years.
- **Logs and records used in the investigation:** technical log (no personal data), access records (6 months) and audit (6 months), according to [0010](decisions/0010-logs-monitoring-and-incident-response.md).
