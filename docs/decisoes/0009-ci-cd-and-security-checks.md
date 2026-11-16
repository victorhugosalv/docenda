# 0009: CI/CD and security checks

- **Date and status:** 16/11/2026, approved by Victor.
- **Context:** RNF-09 asks for TDD, with unit, integration and e2e tests running on every change. RNF-06 asks for ASVS 5.0 level 1 before publishing. Since 16/11, every change reaches `main` only through a pull request: a ruleset requires a PR (0 approvals, single author) and blocks force pushes and deletion. There is no code yet, so this ADR sets the pipeline that comes with the first code. The repository settings were checked through the GitHub API on 16/11/2026. Secret scanning, push protection and branch auto-delete were already on. Dependabot alerts were off and were turned on with this decision. Versions and docs checked on 16/11/2026.
- **Decision:**
  - **D1. CI on every pull request** (GitHub Actions, one `ci.yml`, on `pull_request` and on `push` to `main`):
    1. `npm ci`, so installs follow the lockfile exactly;
    2. types with `tsc --noEmit`;
    3. lint with ESLint 10 and typescript-eslint 8;
    4. module boundaries with dependency-cruiser (0001);
    5. unit and integration tests with Vitest 5;
    6. e2e tests with Playwright 1.65, from the first screen on;
    7. `docker build` of the image (0007 D2), without publishing.
  - **D2. TypeScript pinned to 6.0.x.** TypeScript 7.0.2 (the native compiler) is the npm `latest`, but typescript-eslint 8.72.2 accepts only TypeScript `>=4.8.4 <6.1.0`. Revisit when typescript-eslint supports 7.
  - **D3. Database in integration tests:** a GitHub Actions service container with the same `postgres:18.7-trixie` image as Compose (0007 D1), and Compose locally. Each test cleans up its own data.
  - **D4. Static analysis (SAST):** CodeQL default setup (free for public repositories, no workflow to maintain), with results in the repository's Security tab.
  - **D5. Dependencies:** Dependabot alerts and security updates on from now. A `.github/dependabot.yml` arrives with the first code, with:
    - weekly version updates for `npm`, `github-actions`, `docker` and `docker-compose`;
    - grouped minor and patch updates;
    - a 7-day cooldown, because hijacked npm packages are usually found within days. Security updates skip the cooldown.
  - **D6. Secrets:**
    - GitHub secret scanning and push protection stay on. They catch known provider formats, such as an AWS key.
    - Gitleaks 8.30 runs in CI, to catch generic secrets that the GitHub scan misses, such as a password inside `DATABASE_URL`.
    - A secret that reaches a commit is replaced at once. Removing it from history is not enough.
  - **D7. A hardened pipeline** (GitHub Actions secure use guide):
    - `permissions: contents: read` at the top, and more only in the job that needs it;
    - actions pinned to the full commit SHA, with the version in a comment. Dependabot keeps them updated (to confirm in its docs with the first workflow);
    - no `pull_request_target`;
    - untrusted values (PR titles, branch names) reach scripts only through environment variables.

    The `actions.lock` file (public preview since 14/10/2026) does the same job for the whole dependency tree. It is revisited once it is generally available.
  - **D8. Protected branch:** the existing `main` ruleset also requires the CI checks to pass (required status checks), added once the first workflow has run. Approvals stay at 0.
  - **D9. Delivery (CD):** after a merge to `main`, a job builds the image and publishes it to GitHub Container Registry (GHCR) with the `GITHUB_TOKEN`, so no other secret is needed. Only that job gets `packages: write`. The image is tagged `sha-<commit>`, so any version can be restored exactly. Deploying to AWS, with OIDC instead of fixed keys, belongs to topic 7.
  - **D10. Before publishing:**
    - **Threat model (STRIDE):** a one-page `docs/AMEACAS.md` (public). It has the data flow and a table of threat, current defense (ADR) and what is missing, using the six STRIDE categories (spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege). Open risks are described in general terms, without a step-by-step path to exploit them.
    - **2FA for the administrator only:** Better Auth's `twoFactor` plugin (TOTP and single-use backup codes). The plugin has no rule per role, so our own code refuses admin routes while the administrator has no 2FA, with tests. Whether the TOTP secret is encrypted at rest is checked in the plugin source. 2FA for everyone stays as RF-09 (later).
    - **DAST with OWASP ZAP:** the baseline scan (ZAP action-baseline v0.15.0, image `ghcr.io/zaproxy/zaproxy:stable`, ZAP 2.17) runs in a workflow started by hand and once a week, against the app container in CI. It only browses and watches the answers. A full scan, which really attacks, runs once locally against our own container before publishing.
- **Alternatives considered:**
  - Testcontainers 12 for integration tests: each run gets its own database, but it needs the Docker socket, which 0007 keeps out of Claude's dev container.
  - TypeScript 7 without typescript-eslint: faster type checks, but no lint.
  - Semgrep CE instead of CodeQL: good rules, but one more workflow and configuration to maintain.
  - Gitleaks also as a local pre-commit hook: it catches a secret earlier, but it is one more tool to install. Push protection already blocks known formats at push time.
  - Dependabot alerts only, without version updates: less noise, but the dependencies fall behind and every upgrade becomes a big jump.
  - No publishing until topic 7: simpler, but the "build once, ship the same image" path gets tested only at the end.
  - ZAP only before publishing: fewer runs, but a regression in headers or cookies is found late.
- **Consequences:**
  - Pros: no change reaches `main` without types, tests and security checks; leaked secrets and vulnerable dependencies are flagged on their own; the pipeline itself has minimal permissions; every image can be traced to a commit.
  - Cons: a few minutes per PR; Dependabot PRs to review each week; ZAP and CodeQL can raise false positives that need triage (a ZAP rules file records the accepted ones and why).
  - Changes elsewhere: none in the other ADRs. 0002 does not pin TypeScript; D2 sets it here.
  - Cost: none. GitHub Actions, CodeQL, secret scanning, Dependabot and GHCR are free for public repositories.
- **Sources (checked on 16/11/2026):** [GitHub Actions secure use](https://docs.github.com/en/actions/reference/security/secure-use), [workflow lockfiles (preview)](https://github.blog/changelog/2026-10-14-github-actions-workflow-lockfiles-in-public-preview/), [secret scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning), [push protection](https://docs.github.com/en/code-security/concepts/secret-security/push-protection), [Dependabot options (cooldown, groups, ecosystems)](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference), [CodeQL default setup](https://docs.github.com/en/code-security/code-scanning/enabling-code-scanning/configuring-default-setup-for-code-scanning), [Better Auth two-factor plugin](https://www.better-auth.com/docs/plugins/2fa), [ZAP baseline action](https://github.com/zaproxy/action-baseline), [OWASP Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html), npm registry (`typescript` 7.0.2 and 6.0.3, `typescript-eslint` 8.72.2, `eslint` 10.15.0, `vitest` 5.1.0, `@playwright/test` 1.65.1, `testcontainers` 12.2.1), GitHub releases (`gitleaks` v8.30.1, `zaproxy` v2.17.0).
