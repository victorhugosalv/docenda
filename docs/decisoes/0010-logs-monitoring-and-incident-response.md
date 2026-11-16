# 0010: Logs, monitoring and incident response

- **Date and status:** 16/11/2026, approved by Victor.
- **Context:**
  - RNF-39 asks for logs without passwords, tokens or sensitive content, and for cost, response time and quality per model and operation.
  - REQ-22 lists the audit events (logins, failures, account changes, deletion, plan state changes, misuse and admin decisions), and RF-93 asks for admin decisions to be audited.
  - Section 7.1 of REQUISITOS already keeps access records (date, time, IP) for 6 months (Marco Civil, art. 15).
  - RF-72 asks for an alert at 80% of the AI budget, and 0008 left the queue load to be watched here.
  - OWASP ASVS 5.0 has no level 1 item on logging: chapter V16 starts at level 2. This ADR adopts the cheap level 2 items: 16.2.1 (when, where, who, what), 16.2.2 (UTC), 16.2.4 (a readable format), 16.2.5 (sensitive data rules), 16.3.1 to 16.3.4 (auth, authorization, security events and errors), 16.4.1 (no log injection) and 16.5.1 (generic errors).
  - Resolução CD/ANPD 15/2024 sets the incident rules. A controller tells the ANPD and the data subjects within 3 business days of learning that personal data was affected (arts. 6 and 9), and keeps a record of every incident, reported or not, for at least 5 years (art. 10). Small agents get twice the time. Whether an individual counts as a small agent (Resolução CD/ANPD 2/2022, art. 2) could not be confirmed in the official text, so the plan uses 3 business days anyway.
  - Versions and docs checked on 16/11/2026.
- **Decision:**
  - **D1. Three separate records,** each with its own purpose, place and retention:

    | Record | Purpose | Where | Retention |
    |---|---|---|---|
    | Technical log | Finding bugs | Process output (stdout) | Local: rotation; AWS: 30 days |
    | Access record | Legal duty (Marco Civil) | `access_record` table | 6 months |
    | Audit trail | Who did what (REQ-22) | `audit_event` table | 6 months |

  - **D2. Technical log:** pino 10 with pino-http 11, one JSON line per event on stdout, timestamps in UTC (ISO 8601).
    - Every request gets a request ID (UUID), returned in the `X-Request-Id` header and passed to queue jobs next to the other IDs (0008 D6). Error screens show only a generic message and this ID.
    - Levels: `info` in production, `debug` locally. pino-pretty only in local runs.
    - Log calls never build the message by joining user text: values go as fields (OWASP, log injection).
  - **D3. What never reaches the technical log** (RNF-39):
    - pino `redact` removes `req.headers.cookie`, `req.headers.authorization`, `res.headers["set-cookie"]` and any `password`, `token` or `secret` field;
    - request and response bodies are never logged, so the plan text and the class context never get there, only IDs;
    - the client IP is left out of the technical log. It lives only in the access record (D5);
    - a test sends a request with known decoy values (password, cookie, a fake student name) and fails if any of them shows up in the log output.
  - **D4. Where the technical log lives:** locally, the Docker `json-file` driver with rotation (`max-size: 10m`, `max-file: 3`) in Compose. On AWS, 30 days, set up in topic 7.
  - **D5. Access record:**
    - one row per successful login: account ID, IP and date and time in UTC;
    - kept confidential, readable only by the administrator, and handed over only by court order;
    - a daily scheduled pg-boss job deletes rows older than 6 months.
  - **D6. Audit trail:**
    - its own `audit_event` table with the event type, the actor ID, the target ID and the time. No content, only IDs;
    - append-only: the application's database user may insert and read, but not update or delete. Only the retention job, under a separate database role, deletes rows older than 6 months;
    - it lives in the reusable `audit` module (0001), behind its own small interface.
  - **D7. Monitoring in v1:**
    - `GET /healthz` (the process is alive) and `GET /readyz` (the database answers), used by the Compose healthcheck now and by AWS later;
    - an admin page with cost per operation and per model (RNF-14, RNF-39), read from the AI usage records (RF-74), and the queue state from pg-boss;
    - an e-mail to the administrator, through the existing e-mail port, in three cases: 80% of the monthly AI budget (RF-72), the AI circuit breaker opening (0008 D4), and a job reaching the dead letter queue (0008 D6);
    - no Sentry, OpenTelemetry or Prometheus in v1.
  - **D8. Incident response:**
    - a public plan in `docs/INCIDENTES.md`: contain, rotate secrets, end sessions, assess the risk (art. 5 criteria), notify the ANPD and the data subjects within 3 business days, record, and turn the lesson into a test or an ADR;
    - the incident register is private (outside the public repository) and kept for 5 years;
    - one tabletop exercise before publishing, timing each step.
  - **D9. Change elsewhere:** the V16 row in 0006 gets a note that V16 starts at level 2 and is covered here.
- **Alternatives considered:**
  - Winston instead of pino: more transports and formats, but slower and with more pieces to configure.
  - The IP in the technical log too: easier debugging, but personal data spread into a place with a different retention and wider access.
  - One access record row per request: much more personal data with no gain for the legal duty.
  - The audit trail inside the technical log: it is lost in the rotation, and anyone with access to the server can edit it.
  - Sentry 11 from v1: a good error dashboard, but it sends data abroad and needs every event scrubbed first. Revisited together with AWS (topic 7).
  - OpenTelemetry and Prometheus: the market standard for traces and metrics, but one more stack to run for a v1 with at most 10 people.
  - A private incident plan in the notes: it hides nothing worth hiding, and a public plan shows the work.
- **Consequences:**
  - Pros: no personal content in the technical log; every legal record has its own place and deadline; a request ID links a user's error report to the log without asking for any data; the three alerts cover the failures that cost money or lose work.
  - Cons: two scheduled deletion jobs to test; the admin page and alerts are our own code; without an external error tracker, errors are only seen in the log and in the alert e-mails.
  - Changes elsewhere: 0006 (note on V16), REQUISITOS section 7.1 and REQ-17 (audit trail kept for 6 months).
  - Cost: none.
- **Sources (checked on 16/11/2026):** [OWASP ASVS 5.0, V16](https://github.com/OWASP/ASVS/blob/v5.0.0/5.0/en/0x25-V16-Security-Logging-and-Error-Handling.md), [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html), [pino redaction](https://github.com/pinojs/pino/blob/main/docs/redaction.md), [Resolução CD/ANPD 15/2024 (full text)](https://www.legisweb.com.br/legislacao/?id=458235), [summary of Resolução 15 (Mattos Filho)](https://www.mattosfilho.com.br/unico/regulamento-incidente-seguranca/), [Marco Civil da Internet (Lei 12.965/2014)](https://www.planalto.gov.br/ccivil_03/_ato2011-2014/2014/lei/l12965.htm), npm registry (`pino` 10.4.0, `pino-http` 11.0.0, `pino-pretty` 13.2.0, `@sentry/node` 11.13.0).
