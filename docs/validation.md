# Validation and submission evidence

Verified on 2026-09-14 against the official starter fork:
[skyparado/devsecops-exam-starter](https://github.com/skyparado/devsecops-exam-starter).

## Hosted failure and remediation

| Revision | Run | Result |
| --- | --- | --- |
| A: `f06a0bc` | [Failed CI](https://github.com/skyparado/devsecops-exam-starter/actions/runs/34864508939) | Dependency audit detected deliberately added lodash; image scan also found real runtime vulnerabilities |
| B: `79e326c` | [Successful CI](https://github.com/skyparado/devsecops-exam-starter/actions/runs/34864755525) | All four jobs passed after remediation |

Both revisions ran the same CI workflow. The starter's server.js and
server.test.js are unchanged from upstream commit
`8e66095e2406e7ff6c85103a6e9c424a7482f74b`.

Commit A intentionally added unused development dependency lodash@4.17.20 to the
application package.json and lockfile. The normal Dependency security job failed
on the high-severity finding. Commit B removed it; npm audit passed.

The first Trivy image scan additionally identified two high-severity PCRE2
findings and four high-severity findings in package-manager dependencies.
Commit B upgraded the affected Debian library and removed runtime npm/Yarn,
then passed the same image scan. No finding was allowlisted to make this pass.

## Verified hosted checks

| Check | Result |
| --- | --- |
| Clean npm ci and Jest | Passed |
| Docker build | Passed |
| Container UID is not 0 | Passed |
| Live container /health response | Passed |
| Trivy runtime image gate | Passed: no fixable high/critical findings |
| Compose configuration and app/Redis health | Passed |
| Compose API HTTP request and Redis PONG | Passed |
| npm audit of application, including dev dependencies | Passed: zero findings |
| Trivy working-tree secret scan | Passed |

Image scanning intentionally blocks fixable high/critical findings;
a passing result does not mean the image has no lower-severity or unfixed
vulnerabilities. See [security design](security-design.md).

## Permanent evidence

- [Failed dependency audit](ci-before-dependency-audit.txt)
- [Failed image scan](ci-before-image-scan.txt)
- [Successful dependency audit](ci-after-dependency-audit.txt)
- [Successful image scan](ci-after-image-scan.txt)
- [Successful job and step results](ci-after-results.json)
- [Branch protection snapshot](branch-protection.json)

These are authentic downloaded CI artifacts and API results, not illustrative
screenshots. Reports also remain as workflow artifacts for 30 days; committed
copies retain the evidence after artifact expiration.

## Branch protection

Main requires pull requests and these checks:
Test and build, Compose smoke test, Dependency security, and Secret scan.
Checks must be up to date with main. Rules also apply to administrators;
force pushes and branch deletion are disabled.
The intentionally failing manual demonstration workflow is not required.

## Local checks

The preserved Jest test and live server startup/HTTP response passed under
Node 24.21.0, invoked through npm without replacing the system Node installation.
Main and production-only dependency audits reported zero findings.
The isolated demo audit returned 1 with the expected high-severity lodash finding.
YAML parsing and git diff --check passed.

Docker is not installed on the local generation machine. Docker and Compose
were successfully verified on GitHub's hosted runner instead.

## Remaining submission step

Review the code, explanations, and run evidence so you can explain the decisions.
Submit the fork URL through the exam application form. The application form has
not been submitted automatically.
