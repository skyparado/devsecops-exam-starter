# Validation record

Validated locally on 2026-09-14.

| Check | Result |
| --- | --- |
| Jest using Node 24.21.0 | Passed: 1 suite, 1 test |
| Start server.js with a temporary PORT under Node 24.21.0 | Passed |
| Live GET /health | HTTP 200 and exact expected JSON |
| Main package: npm audit --audit-level=high | Exit 0; zero vulnerabilities |
| Production: npm audit --omit=dev --audit-level=high | Exit 0; zero vulnerabilities |
| Isolated demo audit | Exit 1; high-severity lodash finding, as intended |
| Workflow and Compose YAML parsing | Passed with js-yaml |
| git diff --check | Passed |

Node 24 was invoked through npm exec without replacing the system Node 26
installation. The live server process was stopped after verification.

Audit reports are saved as audit-clean.txt, audit-production.txt, and
audit-demo-isolated.txt. audit-demo.txt preserves the earlier finding before
the demonstration dependency was removed from the application manifest.
Reports reflect the advisory database at execution time.

The main workflow audits the application; the separate manual demonstration
workflow audits security-demo/package-lock.json and intentionally fails.
The demonstration has no runtime code and is excluded from the image.

## Checks still requiring another environment or account actions

- Docker image build, non-root execution, and image health check: Docker is not installed.
- Compose startup and service health: prepared as a separate CI job, not executed locally.
- Hosted GitHub Actions: no workflow run or screenshot has been produced.
- GitHub fork: public repository metadata confirms skyparado/DevSecOps is not a fork.
- Branch protection and application submission: not configured or performed.

Parsing YAML does not establish that a Docker image builds or a hosted workflow
passes. The remaining container checks need Docker or a GitHub Actions runner.
