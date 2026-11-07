# Secure Project Baseline

Reusable free-only security automation specification for web-based software projects.

## 1. Definition of “Free”

For this specification, **free** means:

> No paid tier required, no private-repository restriction, no required SaaS subscription, no commercial-use fee, and usable in private/commercial projects from local development and CI.

This specification deliberately avoids tools that are only free for open-source repositories, tools that require paid SaaS plans for private projects, or tools with unclear commercial-use limitations.

## 2. Purpose

Secure Project Baseline is a reusable security automation layer that can be added to web-based software projects to provide repeatable, automated checks for common security weaknesses during local development, pull requests, CI/CD, and scheduled scans.

The package must maximize security coverage using only tools that are free to use in private and commercial projects, with no dependency on paid SaaS features.

## 3. Goals

The package must:

1. Detect leaked secrets before they enter the repository.
2. Detect vulnerable dependencies.
3. Detect insecure source code patterns.
4. Detect insecure infrastructure-as-code and container configuration.
5. Detect vulnerable container images.
6. Detect missing or weak web security headers.
7. Detect TLS and HTTPS misconfiguration.
8. Run dynamic web application scans against staging environments.
9. Run API security checks from an OpenAPI specification where available.
10. Provide project-specific security regression tests for authentication, authorization, session handling, file uploads, and business logic.
11. Produce machine-readable and human-readable reports.
12. Be usable in GitHub Actions, GitLab CI, Jenkins, local terminal workflows, and pre-commit hooks.
13. Avoid any required paid service, commercial license, or private-repository limitation.

## 4. Non-Goals

The package will not:

1. Guarantee that an application is secure.
2. Replace manual penetration testing.
3. Replace secure architecture review.
4. Replace threat modeling.
5. Automatically fix every finding.
6. Require a specific hosting provider.
7. Require a specific programming language.
8. Require GitHub Advanced Security, Snyk, GitGuardian, commercial Semgrep, or any paid SaaS security scanner.

## 5. Required Free Tooling

### 5.1 Secrets Detection

Primary tool:

```text
detect-secrets
```

Purpose:

```text
Detect API keys, tokens, passwords, private keys, high-entropy strings, and accidental credential commits.
```

Rationale:

```text
detect-secrets can run locally and in CI, supports baselines, and does not require a paid service.
```

Use cases:

```text
pre-commit scan
pull request scan
full repository scan
baseline audit
```

Required configuration files:

```text
.secrets.baseline
.pre-commit-config.yaml
```

Optional alternative:

```text
Gitleaks CLI only, not Gitleaks GitHub Action
```

The baseline package must not depend on `gitleaks-action` because the GitHub Action has separate licensing behavior from the CLI and may not satisfy a strict free-only requirement for all private/commercial use cases.

Another suitable alternative is Trufflehog

### 5.2 Static Application Security Testing

Primary tool:

```text
Semgrep Community Edition
```

Purpose:

```text
Detect insecure source code patterns, OWASP Top 10 issues, framework-specific anti-patterns, unsafe API usage, injection risks, insecure cryptography, and hardcoded security bypasses.
```

Rationale:

```text
Semgrep Community Edition is an open-source static analysis engine that can run locally and in CI without requiring the hosted Semgrep AppSec Platform.
```

Required configuration:

```text
.semgrep.yml
```

Recommended rule sources:

```text
p/owasp-top-ten
p/security-audit
language-specific rules
project-specific custom rules
```

Project-specific rule examples:

```text
No raw SQL string concatenation
No disabled TLS verification
No use of eval
No insecure random generation for tokens
No hardcoded admin bypass
No unsafe deserialization
No direct use of user input in shell commands
No unsafe redirect from user-controlled input
```

### 5.3 Dependency Vulnerability Scanning

Primary tools:

```text
OSV-Scanner
OWASP Dependency-Check
npm audit / pnpm audit / yarn npm audit where applicable
```

Purpose:

```text
Detect known vulnerabilities in open-source dependencies.
```

Rationale:

```text
OSV-Scanner and OWASP Dependency-Check are free/open-source tools suitable for CI and local scanning.
```

Required behavior:

```text
Scan lockfiles where present.
Scan package manifests.
Fail build on critical vulnerabilities by default.
Warn on high vulnerabilities by default.
Allow temporary exceptions with explicit expiry dates.
```

Supported ecosystems should include, where applicable:

```text
npm
pnpm
yarn
Maven
Gradle
Python/pip
Ruby
Go modules
.NET
Composer
```

### 5.4 Container and Filesystem Scanning

Primary tool:

```text
Trivy
```

Purpose:

```text
Scan container images, filesystems, lockfiles, IaC files, secrets, and dependency vulnerabilities.
```

Required scans:

```text
trivy fs .
trivy image <image-name>
trivy config .
```

Required outputs:

```text
SARIF
JSON
human-readable table
```

Default failure policy:

```text
Fail on CRITICAL vulnerabilities.
Warn on HIGH vulnerabilities.
Ignore LOW and MEDIUM unless explicitly configured otherwise.
```

### 5.5 Infrastructure-as-Code Scanning

Primary tools:

```text
Checkov
Trivy config
```

Purpose:

```text
Detect insecure Terraform, CloudFormation, Kubernetes, Docker, Helm, GitHub Actions, and other IaC configurations.
```

Required checks:

```text
public S3 buckets
unencrypted databases
open security groups
overly broad IAM permissions
missing logging
missing encryption at rest
public Kubernetes services
privileged containers
Docker containers running as root
unsafe GitHub Actions permissions
```

### 5.6 Dockerfile Security

Primary tool:

```text
Hadolint
```

Purpose:

```text
Lint Dockerfiles for insecure or poor-practice container build patterns.
```

Required checks:

```text
Avoid latest tags
Avoid root user where possible
Pin base image versions
Avoid curl | sh patterns
Minimize package installation
Remove package manager cache
Avoid secrets in build args
```

### 5.7 Dynamic Application Security Testing

Primary tool:

```text
OWASP ZAP
```

Purpose:

```text
Run automated security scans against a live staging deployment.
```

Required scan types:

```text
ZAP baseline scan
ZAP API scan where OpenAPI exists
ZAP full scan for scheduled or release-candidate scans
```

Required environments:

```text
staging
ephemeral preview environments where available
```

Prohibited environments:

```text
production active scans by default
```

Production-safe scans may include:

```text
passive scan
header checks
TLS checks
robots/sitemap checks
non-invasive GET requests
```

### 5.8 API Security Testing

Primary tools:

```text
Schemathesis
OWASP ZAP API scan
Dredd or Newman where already used
```

Purpose:

```text
Test REST APIs against OpenAPI definitions for schema conformance, malformed input handling, unexpected status codes, authorization enforcement, and server errors.
```

Required checks:

```text
unauthenticated access rejected
invalid methods rejected
invalid content types rejected
invalid payloads rejected
malformed IDs rejected
pagination limits enforced
unexpected 500 errors flagged
OpenAPI schema mismatches flagged
```

### 5.9 TLS and HTTPS Testing

Primary tools:

```text
testssl.sh
Mozilla Observatory
SSL Labs manual/scheduled verification
```

Purpose:

```text
Detect weak TLS configuration and missing HTTPS protections.
```

Required checks:

```text
HTTP redirects to HTTPS
TLS 1.0 disabled
TLS 1.1 disabled
weak ciphers disabled
valid certificate chain
HSTS present
no mixed content
secure cookies only over HTTPS
```

### 5.10 Security Header Testing

Primary tools:

```text
custom curl/fetch script
Mozilla Observatory
OWASP ZAP passive scan
```

Purpose:

```text
Detect missing or weak web security headers.
```

Required headers:

```http
Content-Security-Policy
Strict-Transport-Security
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
Cross-Origin-Opener-Policy
Cross-Origin-Resource-Policy
```

Recommended legacy/compatibility header:

```http
X-Frame-Options
```

The package should prefer CSP `frame-ancestors` over `X-Frame-Options` where practical.

## 6. Required Project Files

The package must add the following directory structure:

```text
/security
  /config
    semgrep.yml
    zap-baseline.conf
    zap-api.conf
    trivy.yaml
    checkov.yaml
    detect-secrets.filters
    security-thresholds.json
  /scripts
    security-local.sh
    security-ci.sh
    security-staging.sh
    check-headers.js
    check-cookies.js
    check-openapi.js
    check-sensitive-errors.js
    generate-security-report.js
  /tests
    auth.security.spec.ts
    authorization.security.spec.ts
    session.security.spec.ts
    headers.security.spec.ts
    upload.security.spec.ts
    api.security.spec.ts
    business-logic.security.spec.ts
  /reports
    .gitkeep
  /docs
    security-automation.md
    security-exceptions.md
    security-test-matrix.md
```

## 7. Required NPM Scripts

For JavaScript/TypeScript projects, the package must add these scripts:

```json
{
  "scripts": {
    "security": "npm run security:secrets && npm run security:sast && npm run security:deps && npm run security:headers",
    "security:local": "bash security/scripts/security-local.sh",
    "security:ci": "bash security/scripts/security-ci.sh",
    "security:staging": "bash security/scripts/security-staging.sh",
    "security:secrets": "detect-secrets scan --baseline .secrets.baseline",
    "security:secrets:audit": "detect-secrets audit .secrets.baseline",
    "security:sast": "semgrep scan --config security/config/semgrep.yml --error",
    "security:deps": "osv-scanner scan source .",
    "security:deps:npm": "npm audit --audit-level=high",
    "security:container": "trivy image ${IMAGE_NAME}",
    "security:fs": "trivy fs .",
    "security:iac": "checkov -d . && trivy config .",
    "security:headers": "node security/scripts/check-headers.js",
    "security:cookies": "node security/scripts/check-cookies.js",
    "security:api": "schemathesis run ${OPENAPI_URL}",
    "security:zap:baseline": "docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t ${STAGING_URL} -r zap-report.html",
    "security:test": "playwright test security/tests"
  }
}
```

For non-Node projects, equivalent Makefile commands must be provided:

```makefile
security:
	$(MAKE) security-secrets
	$(MAKE) security-sast
	$(MAKE) security-deps
	$(MAKE) security-iac

security-secrets:
	detect-secrets scan --baseline .secrets.baseline

security-sast:
	semgrep scan --config security/config/semgrep.yml --error

security-deps:
	osv-scanner scan source .

security-iac:
	checkov -d .
	trivy config .

security-fs:
	trivy fs .

security-staging:
	zap-baseline.py -t $(STAGING_URL) -r security/reports/zap-report.html
```

## 8. Security Automation Levels

The package must define four automation levels.

### 8.1 Level 1: Local Developer Checks

Runs before commit or before opening a pull request.

Required checks:

```text
detect-secrets
Semgrep fast rules
dependency lockfile scan
Dockerfile lint if Dockerfile exists
```

Failure behavior:

```text
Block commit for confirmed secrets.
Warn for medium/low SAST findings.
Block for high-confidence critical SAST findings.
```

### 8.2 Level 2: Pull Request Checks

Runs on every pull request.

Required checks:

```text
secrets scan
SAST scan
dependency vulnerability scan
IaC scan
Dockerfile scan
unit security tests
authorization regression tests
header tests against local app where possible
```

Failure behavior:

```text
Fail on confirmed secrets.
Fail on critical dependency vulnerabilities.
Fail on critical/high IaC misconfigurations.
Fail on high-confidence SAST findings.
Fail on broken authorization tests.
```

### 8.3 Level 3: Staging Deployment Checks

Runs after deployment to staging.

Required checks:

```text
OWASP ZAP baseline scan
security header scan
cookie attribute scan
TLS scan
OpenAPI/API scan
authenticated route checks
file upload abuse checks
error leakage checks
```

Failure behavior:

```text
Fail release candidate on high or critical ZAP alerts unless explicitly excepted.
Fail on missing HSTS in production-like environment.
Fail on session cookie missing Secure or HttpOnly.
Fail on authorization bypass.
Fail on exposed stack trace.
```

### 8.4 Level 4: Scheduled Security Checks

Runs daily or weekly.

Required checks:

```text
full dependency scan
full container scan
ZAP full scan against staging
Nuclei scan against staging if adopted
testssl.sh scan
subdomain and exposed service checks where configured
```

Failure behavior:

```text
Create issue/ticket for high and critical findings.
Notify maintainers.
Do not automatically deploy or block existing production unless configured.
```

## 9. Security Test Matrix

The package must include a security test matrix based on OWASP WSTG categories.

| WSTG Area | Automated Check | Required Tool |
|---|---|---|
| Information Gathering | Exposed files, backup files, admin paths | ZAP, custom script |
| Configuration | Headers, methods, debug modes | ZAP, custom script |
| Identity | Account enumeration checks | Playwright/API tests |
| Authentication | Login, reset, lockout, MFA where used | Playwright/API tests |
| Authorization | IDOR, tenant isolation, privilege escalation | Playwright/API tests |
| Session Management | Cookie flags, logout, session rotation | Playwright/API tests |
| Input Validation | XSS, SQLi-like payloads, SSRF guards | ZAP, Semgrep, API tests |
| Error Handling | Stack traces, SQL errors, framework errors | Custom API tests |
| Cryptography | TLS, weak ciphers, HTTPS redirects | testssl.sh |
| Business Logic | Payment, workflow, quota, inventory abuse | Project-specific tests |
| Client-Side | Storage, CORS, open redirects, CSP | Playwright, ZAP |
| API | OpenAPI fuzzing, schema mismatch, auth | Schemathesis, ZAP |

## 10. Required Project-Specific Security Regression Tests

The package must include templates for these tests. Each project must implement the ones that apply.

### 10.1 Authentication Tests

Required scenarios:

```text
anonymous users cannot access protected pages
anonymous users cannot access protected API endpoints
invalid login does not reveal whether email exists
password reset token expires
password reset token is single-use
logout invalidates session
old session cannot access protected endpoint after logout
```

### 10.2 Authorization Tests

Required scenarios:

```text
normal user cannot access admin endpoints
user cannot read another user's records
user cannot update another user's records
user cannot delete another user's records
tenant A cannot access tenant B data
deactivated user cannot access protected resources
role changes take effect without requiring a new deployment
```

### 10.3 Session Tests

Required scenarios:

```text
session cookie has HttpOnly
session cookie has Secure
session cookie has SameSite=Lax or SameSite=Strict
session ID changes after login
session ID changes after privilege elevation
session expires after configured timeout
session token is not present in URL
sensitive token is not stored in localStorage
```

### 10.4 Input Validation Tests

Required scenarios:

```text
XSS payloads are escaped or rejected
SQL injection-like payloads do not trigger 500 errors
path traversal payloads are rejected
template injection payloads are rejected
command injection payloads are rejected
SSRF payloads to localhost and cloud metadata IPs are rejected
unexpected content types are rejected
oversized payloads are rejected
```

### 10.5 File Upload Tests

Required scenarios:

```text
disallowed extensions are rejected
MIME type is validated server-side
file magic bytes are validated where appropriate
oversized files are rejected
SVG files are sanitized or rejected
uploaded files are not executable
uploaded files are served from isolated storage
uploaded file URLs do not expose local filesystem paths
```

### 10.6 Error Leakage Tests

Required scenarios:

```text
invalid IDs do not expose stack traces
database errors are not exposed
ORM errors are not exposed
absolute server paths are not exposed
environment variables are not exposed
internal hostnames are not exposed
debug pages are disabled outside local development
```

### 10.7 Business Logic Tests

Each project must define business-specific security abuse cases.

For an event/ticketing platform, required examples would include:

```text
attendee cannot receive ticket without successful payment
ticket inventory cannot go below zero
ticket cannot be checked in twice unless explicitly permitted
refund cannot exceed original payment
discount code cannot be reused past limit
organizer cannot modify another organizer's event
organizer cannot alter payout destination without reauthentication
webhook replay does not duplicate fulfillment
client-side price tampering is rejected server-side
```

For a SaaS application, required examples would include:

```text
free-plan user cannot access paid-plan endpoints
trial-expired user cannot access paid features
user cannot invite members beyond plan limit
user cannot change subscription price client-side
deleted workspace cannot be accessed
API key from one workspace cannot access another workspace
```

## 11. Default Security Thresholds

The package must include `security/config/security-thresholds.json`:

```json
{
  "secrets": {
    "confirmed": "fail",
    "suspected": "warn"
  },
  "sast": {
    "critical": "fail",
    "high": "fail",
    "medium": "warn",
    "low": "ignore"
  },
  "dependencies": {
    "critical": "fail",
    "high": "warn",
    "medium": "warn",
    "low": "ignore"
  },
  "containers": {
    "critical": "fail",
    "high": "warn",
    "medium": "warn",
    "low": "ignore"
  },
  "iac": {
    "critical": "fail",
    "high": "fail",
    "medium": "warn",
    "low": "ignore"
  },
  "dast": {
    "critical": "fail",
    "high": "fail",
    "medium": "warn",
    "low": "ignore"
  },
  "headers": {
    "missing_hsts": "fail_in_production",
    "missing_csp": "warn",
    "missing_x_content_type_options": "fail",
    "missing_referrer_policy": "warn",
    "missing_permissions_policy": "warn"
  }
}
```

## 12. Exception Handling

The package must support exceptions, but exceptions must be explicit, reviewed, and time-limited.

Each exception must include:

```text
finding ID
tool name
affected file/URL/package
reason
risk acceptance owner
expiration date
compensating control
link to ticket
```

Example:

```yaml
exceptions:
  - id: "OSV-2025-1234"
    tool: "osv-scanner"
    package: "example-package"
    reason: "No fixed version available"
    owner: "security-owner@example.com"
    expires: "2026-07-01"
    compensating_control: "Vulnerable function is not reachable"
    ticket: "SEC-123"
```

Rules:

```text
No permanent exceptions.
No exception may exceed 90 days without reapproval.
Critical findings require explicit owner approval.
Secrets may not be excepted if confirmed valid.
Authorization bypasses may not be excepted.
```

The same process covers code quality thresholds. See §29.2 for the fields a quality exception records.

## 13. Required CI/CD Behavior

### 13.1 Pull Request Workflow

Every pull request must run:

```text
detect-secrets
Semgrep CE
OSV-Scanner
Trivy filesystem scan
Checkov if IaC files exist
Hadolint if Dockerfile exists
security regression tests
```

### 13.2 Main Branch Workflow

Every merge to main must run:

```text
all pull request checks
container build
Trivy image scan
artifact/report generation
```

### 13.3 Staging Workflow

Every staging deployment must run:

```text
ZAP baseline scan
header checks
cookie checks
API security tests
authorization regression tests
error leakage tests
```

### 13.4 Scheduled Workflow

Weekly scheduled workflow must run:

```text
full dependency scan
full container scan
ZAP full scan against staging
TLS scan
IaC scan
security report generation
```

## 14. Example GitHub Actions Workflow

```yaml
name: security

on:
  pull_request:
  push:
    branches:
      - main
  schedule:
    - cron: "0 6 * * 1"

permissions:
  contents: read
  security-events: write
  actions: read

jobs:
  static-security:
    name: Static Security Checks
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Install Python security tools
        run: |
          pip install detect-secrets semgrep checkov

      - name: Secrets scan
        run: |
          detect-secrets scan --baseline .secrets.baseline
          detect-secrets audit .secrets.baseline --report

      - name: Semgrep scan
        run: |
          semgrep scan --config security/config/semgrep.yml --error

      - name: Install OSV-Scanner
        run: |
          curl -L https://github.com/google/osv-scanner/releases/latest/download/osv-scanner_linux_amd64 -o osv-scanner
          chmod +x osv-scanner
          sudo mv osv-scanner /usr/local/bin/osv-scanner

      - name: Dependency scan
        run: |
          osv-scanner scan source .

      - name: Install Trivy
        run: |
          sudo apt-get update
          sudo apt-get install -y wget apt-transport-https gnupg lsb-release
          wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg
          echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
          sudo apt-get update
          sudo apt-get install -y trivy

      - name: Filesystem scan
        run: |
          trivy fs --severity HIGH,CRITICAL --exit-code 1 .

      - name: IaC scan
        run: |
          checkov -d . --quiet
          trivy config --severity HIGH,CRITICAL --exit-code 1 .

  node-security-tests:
    name: Node/Web Security Tests
    runs-on: ubuntu-latest
    needs: static-security

    if: hashFiles('package.json') != ''

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm ci

      - name: npm audit
        run: npm audit --audit-level=high

      - name: Run security tests
        run: npm run security:test

  docker-security:
    name: Docker Security
    runs-on: ubuntu-latest

    if: hashFiles('Dockerfile') != ''

    steps:
      - uses: actions/checkout@v4

      - name: Hadolint Dockerfile
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile

      - name: Build image
        run: docker build -t local/security-target:${{ github.sha }} .

      - name: Scan image with Trivy
        run: |
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy image \
            --severity HIGH,CRITICAL \
            --exit-code 1 \
            local/security-target:${{ github.sha }}

  staging-dast:
    name: Staging DAST
    runs-on: ubuntu-latest
    needs:
      - static-security

    if: github.ref == 'refs/heads/main' && vars.STAGING_URL != ''

    steps:
      - uses: actions/checkout@v4

      - name: ZAP Baseline Scan
        run: |
          docker run --rm \
            -v ${{ github.workspace }}/security/reports:/zap/wrk \
            ghcr.io/zaproxy/zaproxy:stable \
            zap-baseline.py \
            -t "${{ vars.STAGING_URL }}" \
            -r zap-baseline-report.html

      - name: Security Header Checks
        run: |
          node security/scripts/check-headers.js "${{ vars.STAGING_URL }}"
```

## 15. Required Header Check Script Behavior

The header check script must:

1. Accept a target URL.
2. Send an HTTPS request.
3. Fail if the response is not HTTPS in production mode.
4. Check required headers.
5. Return non-zero exit code on blocking failures.
6. Print actionable remediation messages.

Minimum required logic:

```js
const REQUIRED_HEADERS = [
  "x-content-type-options",
  "strict-transport-security",
  "referrer-policy",
  "permissions-policy"
];

const RECOMMENDED_HEADERS = [
  "content-security-policy",
  "cross-origin-opener-policy",
  "cross-origin-resource-policy"
];

async function checkHeaders(url) {
  const response = await fetch(url, { redirect: "manual" });
  const failures = [];
  const warnings = [];

  for (const header of REQUIRED_HEADERS) {
    if (!response.headers.get(header)) {
      failures.push(`Missing required header: ${header}`);
    }
  }

  for (const header of RECOMMENDED_HEADERS) {
    if (!response.headers.get(header)) {
      warnings.push(`Missing recommended header: ${header}`);
    }
  }

  const xcto = response.headers.get("x-content-type-options");
  if (xcto && xcto.toLowerCase() !== "nosniff") {
    failures.push("X-Content-Type-Options must be nosniff");
  }

  return { failures, warnings };
}
```

## 16. Required Cookie Check Behavior

The cookie check script must validate:

```text
session cookies use Secure
session cookies use HttpOnly
session cookies use SameSite=Lax or Strict
auth tokens are not stored in URL
auth tokens are not stored in localStorage unless explicitly accepted
auth tokens are not stored in sessionStorage unless explicitly accepted
```

The script must support project-specific session cookie names:

```json
{
  "sessionCookieNames": [
    "sid",
    "session",
    "connect.sid",
    "__Host-session",
    "__Secure-session"
  ]
}
```

## 17. Required Semgrep Rules

The package must include custom Semgrep rules for common web app risks.

Required categories:

```text
unsafe eval
child_process with user input
raw SQL concatenation
disabled TLS verification
weak crypto
Math.random for security token generation
hardcoded JWT secrets
hardcoded API keys
open redirect patterns
dangerous CORS wildcard with credentials
Express error leakage
missing authorization middleware on protected routes
```

Example rule:

```yaml
rules:
  - id: node-disable-tls-verification
    message: Do not disable TLS certificate verification.
    severity: ERROR
    languages:
      - javascript
      - typescript
    patterns:
      - pattern: process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0"
```

Example Express route guard rule:

```yaml
rules:
  - id: express-route-without-auth-middleware
    message: Route may be missing authentication/authorization middleware.
    severity: WARNING
    languages:
      - javascript
      - typescript
    patterns:
      - pattern: app.$METHOD($ROUTE, $HANDLER)
      - pattern-not: app.$METHOD($ROUTE, $AUTH, $HANDLER)
```

## 18. Required Playwright Security Tests

The package must include Playwright templates for:

```text
anonymous route access
authorization boundary checks
session cookie checks
logout invalidation
browser storage checks
open redirect checks
file upload abuse
error leakage
```

Example:

```ts
import { test, expect } from "@playwright/test";

test("protected API rejects anonymous requests", async ({ request }) => {
  const response = await request.get(`${process.env.API_URL}/me`);
  expect([401, 403]).toContain(response.status());
});

test("invalid ID does not expose stack trace", async ({ request }) => {
  const response = await request.get(`${process.env.API_URL}/records/not-a-real-id`);
  const body = await response.text();

  expect(body).not.toMatch(/stack|trace|sequelize|typeorm|prisma|node_modules|syntaxerror|referenceerror/i);
});
```

## 19. Required ZAP Configuration

The ZAP baseline scan must:

```text
run against staging only
exclude logout URLs
exclude destructive endpoints
exclude payment-processing endpoints unless explicitly configured
produce HTML and JSON reports
fail on high-confidence high-risk findings
warn on medium findings
```

Required ZAP files:

```text
security/config/zap-baseline.conf
security/config/zap-api.conf
```

Required exclusions:

```text
/logout
/delete
/remove
/payment
/stripe
/webhook
/admin/destructive-action
```

These exclusions must be configurable.

## 20. Required OpenAPI Security Requirements

If a project exposes an OpenAPI spec, the package must validate:

```text
all protected endpoints declare security requirements
no undocumented public endpoints exist
request bodies have schemas
responses have schemas
error responses are documented
pagination parameters have maximum limits
file upload endpoints declare allowed media types
admin endpoints require admin security scheme
```

Recommended OpenAPI security schemes:

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

## 21. Required Reports

The package must generate:

```text
security-summary.md
security-summary.json
zap-report.html
semgrep.sarif
trivy-results.json
osv-results.json
checkov-results.json
```

The Markdown summary must include:

```text
scan date
commit SHA
branch
tool versions
target URL
total findings by severity
blocking findings
warnings
exceptions used
expired exceptions
recommended next actions
```

## 22. Required Documentation

The package must include `security/docs/security-automation.md` with:

```text
how to run local checks
how to update the secrets baseline
how to audit suspected secrets
how to add an exception
how to configure staging scan target
how to add project-specific authorization tests
how to interpret reports
how to handle false positives
how to update tools
```

It must include `security/docs/security-exceptions.md` with:

```text
exception format
approval rules
expiration rules
examples
prohibited exceptions
```

It must include `security/docs/security-test-matrix.md` with:

```text
WSTG category
automated test
tool
location
owner
frequency
failure behavior
```

## 23. Minimum Adoption Checklist

A project has adopted the baseline only when all of the following are true:

```text
detect-secrets baseline exists
Semgrep scan runs in CI
dependency scan runs in CI
container scan runs if Docker is used
IaC scan runs if IaC exists
security header check runs against staging
ZAP baseline scan runs against staging
authorization regression tests exist
session cookie tests exist
error leakage tests exist
security report artifact is generated
exception process exists
```

## 24. Recommended Initial Implementation Order

### 24.1 Phase 1: Immediate Baseline

```text
detect-secrets
Semgrep CE
OSV-Scanner
Trivy filesystem scan
security headers script
basic auth/authorization tests
```

### 24.2 Phase 2: Deployment Security

```text
ZAP baseline scan
cookie/session tests
error leakage tests
TLS scan
file upload tests
```

### 24.3 Phase 3: Advanced Coverage

```text
Checkov IaC scan
Trivy container image scan
Schemathesis OpenAPI fuzzing
business logic abuse tests
weekly scheduled ZAP full scan
exception lifecycle management
```

## 25. Tools Explicitly Excluded From the Required Baseline

The following must not be required because they involve paid tiers, private-repo restrictions, usage limits, SaaS dependency, or licensing ambiguity for this package’s goals:

```text
GitHub Advanced Security
GitGuardian
Snyk
Semgrep AppSec Platform paid features
Gitleaks GitHub Action
Burp Suite Professional
StackHawk paid plans
Mend/WhiteSource paid plans
Veracode
Checkmarx commercial products
SonarQube commercial security features
```

Allowed optional tools:

```text
Burp Suite Community for manual local testing
Gitleaks CLI for local supplementary scanning
GitHub Dependabot where available
GitHub CodeQL where the repository/license situation allows it
```

## 26. Success Criteria

The package is successful when:

```text
new secrets are blocked before merge
critical vulnerable dependencies block release
critical container vulnerabilities block release
dangerous IaC misconfigurations block release
missing production security headers are detected
session cookies are verified automatically
authorization boundaries are tested automatically
staging deployments receive DAST coverage
security findings are visible in CI artifacts
exceptions are time-limited and auditable
projects can adopt the baseline without buying a tool
```

## 27. Final Recommended Tool Stack

For the strict free-only baseline, use:

| Security Area | Required Tool |
|---|---|
| Secrets | detect-secrets |
| SAST | Semgrep Community Edition |
| Dependencies | OSV-Scanner, OWASP Dependency-Check where useful |
| Filesystem/container scan | Trivy |
| IaC | Checkov, Trivy config |
| Dockerfile | Hadolint |
| DAST | OWASP ZAP |
| API fuzzing | Schemathesis |
| TLS | testssl.sh |
| Headers | custom script, ZAP passive scan |
| Browser/security regression | Playwright |
| Reporting | generated Markdown/JSON/SARIF artifacts |

## 28. Notes on Adopting This Specification

This specification is intended to be used as a reusable baseline. Each project should still add application-specific tests for the most important security boundaries in that application.

For most web applications, the highest-value project-specific additions are:

```text
authorization boundary tests
tenant isolation tests
session invalidation tests
file upload abuse tests
payment/business-logic abuse tests
API key and token scope tests
```

Automated scanners are useful, but the highest-risk web application security failures are often logic and authorization flaws that only project-specific tests can reliably catch.

## 29. Default Code Quality Thresholds

This section sets one fleet-wide floor for duplication and coverage checks, in the same way §11 sets default security thresholds. It replaces per-repository numbers that drifted without a decision.

The package must include `quality/config/quality-thresholds.json`:

```json
{
  "mode": "floor",
  "duplication": {
    "tool": "jscpd",
    "threshold": 1,
    "minTokens": 50,
    "ignore": [
      "test and spec files",
      "test directories",
      "generated code",
      "build output",
      "coverage reports"
    ]
  },
  "coverage": {
    "lines": 80,
    "statements": 80,
    "branches": 70,
    "functions": 75
  }
}
```

The `ignore` entries name categories. Each project maps them to its own globs (for example `**/*.test.*`, `**/__tests__/**`, `**/dist/**`). Test code is excluded because repeated arrange/act/assert structure is a pattern, not debt. Generated code is excluded because nobody edits it by hand.

### 29.1 Floor Semantics

These numbers are a floor, not a target. They may only make a project stricter.

```text
A project that is already stricter than the floor keeps its stricter number.
A project that is below the floor records a §12 exception at its current number.
A project's number may move toward stricter, never toward looser.
```

What "stricter" means for each value:

| Value | Floor | Stricter means |
| --- | --- | --- |
| jscpd `threshold` | `1` | lower (`0` is stricter than `1`) |
| jscpd `minTokens` | `50` | lower (smaller clones are detected) |
| jscpd ignore list | test and generated code | fewer exclusions (scanning test code too is allowed) |
| coverage `lines` | `80` | higher |
| coverage `statements` | `80` | higher |
| coverage `branches` | `70` | higher |
| coverage `functions` | `75` | higher |

Examples:

```text
jscpd threshold 0 stays 0. Adopting this floor does not raise it to 1.
coverage branches 80, 85, 90 or 100 stays where it is. Adopting this floor does not lower it to 70.
jscpd minTokens 150 is below the floor. It needs a §12 exception or a change to 50 or lower.
coverage lines 45 is below the floor. It needs a §12 exception at 45 and must ratchet upward.
```

`minTokens` is part of the floor, not a tuning knob. A raised `minTokens` finds less duplication while the project reports the same `threshold`, so a project may not meet the `threshold` floor by raising `minTokens`.

### 29.2 Exceptions

A project below the floor on any value must record a dated exception under §12. The §12 rules apply unchanged: no permanent exceptions, and no exception may exceed 90 days without reapproval.

Each quality exception must include:

```text
tool name
metric
floor value
current value
reason
owner
expiration date
next target value
link to ticket
```

Example:

```yaml
exceptions:
  - id: "QUALITY-coverage-branches"
    tool: "jest"
    metric: "coverage.branches"
    floor: 70
    current: 45
    reason: "Legacy request handlers predate the test suite"
    owner: "repo-owner@example.com"
    expires: "2027-02-01"
    next_target: 55
    ticket: "PROJ-123"
```

Rules:

```text
The current value is the number the project measures today, not a lower number that is easier to pass.
On reapproval, current must be at least the previous current value.
A quality exception may never lower a number that already meets the floor.
The exception is removed once the project meets the floor.
```
