# Secure Project Baseline

A reusable, **free-only** security automation layer for web-based software projects.

Secure Project Baseline gives any web project a repeatable set of automated security checks that run during local development, on pull requests, in CI/CD, after staging deployments, and on a schedule — using **only tools that are free to use in private and commercial projects**, with no dependency on paid SaaS tiers, private-repo restrictions, or commercial licenses.

> 📄 The full specification lives in **[`spec.md`](./spec.md)**. This README is the summary; the spec is the source of truth.

## What "free" means here

No paid tier required, no private-repository restriction, no required SaaS subscription, no commercial-use fee, and usable in private/commercial projects from local development through CI. The baseline deliberately avoids tools that are only free for open-source repos or that gate private-project use behind paid plans.

## Why use it

- **Catch problems early** — leaked secrets, vulnerable dependencies, and insecure code patterns are flagged before they merge.
- **One baseline, many projects** — drop the same `security/` layer into any web app regardless of language or host.
- **Defense in depth** — combines static analysis, dependency scanning, container/IaC scanning, dynamic scanning, and project-specific regression tests.
- **No vendor lock-in, no bill** — every required tool is genuinely free for commercial use.

## What it covers

| Security area | Tool |
| --- | --- |
| Secrets | detect-secrets (Gitleaks CLI / TruffleHog optional) |
| SAST | Semgrep Community Edition |
| Dependencies | OSV-Scanner, OWASP Dependency-Check, native `audit` |
| Filesystem / container | Trivy |
| Infrastructure-as-Code | Checkov, `trivy config` |
| Dockerfile | Hadolint |
| Dynamic scanning (DAST) | OWASP ZAP |
| API fuzzing | Schemathesis |
| TLS / HTTPS | testssl.sh, Mozilla Observatory |
| Security headers & cookies | custom scripts, ZAP passive scan |
| Browser / regression tests | Playwright |
| Reporting | generated Markdown / JSON / SARIF artifacts |

## Automation levels

The baseline defines four levels, each running at a different point in the lifecycle:

1. **Local developer checks** — pre-commit/pre-PR: secrets, fast Semgrep rules, lockfile scan, Dockerfile lint. Blocks confirmed secrets.
2. **Pull request checks** — full secrets/SAST/dependency/IaC/Dockerfile scans plus security regression tests. Fails on confirmed secrets, critical dependency vulns, critical/high IaC issues, and broken authorization tests.
3. **Staging deployment checks** — ZAP baseline scan, header/cookie/TLS checks, API security tests, error-leakage checks against a deployed staging environment.
4. **Scheduled checks** — daily/weekly full dependency, container, and ZAP full scans; opens issues for high/critical findings.

## Project layout the baseline adds

```text
security/
  config/      # semgrep.yml, zap-*.conf, trivy.yaml, checkov.yaml,
               # detect-secrets.filters, security-thresholds.json
  scripts/     # security-local.sh, security-ci.sh, security-staging.sh,
               # check-headers.js, check-cookies.js, generate-security-report.js, …
  tests/       # auth, authorization, session, headers, upload, api,
               # business-logic security specs
  reports/     # generated scan artifacts
  docs/        # security-automation.md, security-exceptions.md, security-test-matrix.md
```

## Getting started

For a Node/TypeScript project, the baseline adds npm scripts so the full suite runs with one command:

```bash
# Run the core static checks locally
npm run security

# Individual stages
npm run security:secrets    # detect-secrets
npm run security:sast       # Semgrep CE
npm run security:deps       # OSV-Scanner
npm run security:headers    # custom header check
npm run security:test       # Playwright security regression tests
```

For non-Node projects, equivalent `make` targets (`make security`, `make security-sast`, …) are provided. See [`spec.md` §7](./spec.md) for the full script and Makefile definitions, and §14 for a ready-to-use GitHub Actions workflow.

## Exceptions

Findings can be excepted, but every exception must be explicit, reviewed, and time-limited — finding ID, tool, affected target, reason, risk owner, expiration date, compensating control, and ticket link. No permanent exceptions; none may exceed 90 days without reapproval. Confirmed valid secrets and authorization bypasses can never be excepted. See [`spec.md` §12](./spec.md).

## Adoption checklist

A project has adopted the baseline only when all of these are true:

- [ ] detect-secrets baseline exists
- [ ] Semgrep scan runs in CI
- [ ] Dependency scan runs in CI
- [ ] Container scan runs if Docker is used
- [ ] IaC scan runs if IaC exists
- [ ] Security header check runs against staging
- [ ] ZAP baseline scan runs against staging
- [ ] Authorization regression tests exist
- [ ] Session cookie tests exist
- [ ] Error leakage tests exist
- [ ] Security report artifact is generated
- [ ] Exception process exists

## What this is *not*

This baseline does **not** guarantee an application is secure, and does not replace manual penetration testing, secure architecture review, or threat modeling. Automated scanners catch a lot, but the highest-risk web vulnerabilities are usually logic and authorization flaws that only project-specific tests can reliably catch — so each adopting project should still add tests for its own most important security boundaries (authorization, tenant isolation, session invalidation, file uploads, payment/business-logic abuse, token scope).

## License & status

Released under the [MIT License](./LICENSE). This repository currently contains the specification — see [`spec.md`](./spec.md) for the complete, authoritative requirements.
