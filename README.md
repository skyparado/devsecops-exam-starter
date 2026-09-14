# Macky Merch API - secure delivery pipeline

Based on [the LSCS starter](https://github.com/dlsu-lscs/devsecops-exam-starter) at commit `8e66095e2406e7ff6c85103a6e9c424a7482f74b`. The starter's server and test are preserved. This workspace is an existing repository; importing starter files does **not** create a GitHub fork.

## Local setup

Install Node.js 24 LTS, then run:

```sh
npm ci --ignore-scripts
npm test -- --runInBand
npm start
```

Open http://localhost:3000/health. Expected response:

```json
{"status":"OK","message":"Macky Merch API is running smoothly."}
```

## Docker

Install Docker with Compose support and start its engine.

```sh
docker build --pull -t macky-merch:local .
docker run --rm --init -p 127.0.0.1:3000:3000 --name macky-merch macky-merch:local
```

In another terminal, verify the endpoint and non-root identity:

```sh
curl http://localhost:3000/health
docker exec macky-merch id
```

The user should be `node` (UID 1000). Stop the foreground container with Ctrl+C.

Optional API and dummy Redis:

```sh
docker compose up --build -d
docker compose ps
docker compose down
```

Both services join the same Compose network. Redis has no published host port and stores no persistent data. It is a dummy service for the networking bonus; the unchanged starter API does not query it.

## Architecture

```mermaid
flowchart TD
  A[Push or pull request to main] --> B[GitHub Actions]
  B --> C[npm ci and Jest]
  C --> D[Build image]
  D --> E[Non-root and HTTP checks]
  E --> F[Trivy image scan]
  B --> G[npm audit]
  B --> H[Trivy secret scan]
  B --> I[Compose app and Redis checks]
  F --> J[Required status checks]
  G --> J
  H --> J
  I --> J
```

See [security design](docs/security-design.md) for threat coverage, scan thresholds,
and limitations. Security audit and image reports are retained as CI artifacts
for 30 days. No deployment credentials are used.


The Dockerfile uses `node:24-bookworm-slim`: Node 24 is an LTS line, and Debian slim provides a small, familiar glibc environment. Unlike `latest`, the tag fixes the Node major and OS family. The Dockerfile also pins the multi-platform image digest. Dependabot proposes explicit updates instead of silently changing the base image. See the [Node release schedule](https://github.com/nodejs/Release) and [official Node image documentation](https://github.com/nodejs/docker-node).

The dependency stage installs only locked production packages. The runtime stage applies the available PCRE2 security update and removes npm/Yarn, which are unnecessary to execute the API. OS security updates are resolved from Debian repositories at build time, so the full build is not hermetic despite the pinned base image. The final stage copies production modules and the two necessary application files, runs as the built-in `node` user, and uses Node itself for its health check. Jest, Supertest, the demonstration dependency, Git metadata, and environment files are excluded. The allowlist in `.dockerignore` minimizes build context.

The workflow triggers on pushes and pull requests targeting `main`, plus manual runs. Separate jobs run tests/build/container checks, Compose startup checks, and dependency scanning. `npm ci` replaces the exam's `npm install` command for reproducibility and fails if the manifest and lockfile disagree. The container smoke test checks the UID and actual HTTP response. Workflow permissions are read-only, with time limits and cancellation of superseded runs. This is CI validation; no deployment destination was specified.

## Security demonstration

The application manifest has been remediated. Its full audit should now pass.
The separate private package in `security-demo/` deliberately pins the unused
`lodash@4.17.20` dependency. It is excluded from the Docker build context and is
never imported, installed by the root package, or deployed.

The main CI workflow audits all application dependencies with
`npm audit --audit-level=high`. A separate, manually triggered **Security
demonstration (expected failure)** workflow audits the fixture and must fail.
This preserves a repeatable negative example without breaking required main CI
checks. npm audit uses the npm advisory database and needs registry access;
network errors also fail the job. See the
[npm audit documentation](https://docs.npmjs.com/cli/v11/commands/npm-audit).

Run both checks locally:

```sh
npm audit --audit-level=high
npm --prefix security-demo audit --audit-level=high
```

The first should exit 0; the second should exit 1 and identify lodash.
The fixture has its own lockfile, so it can be audited without installing its
vulnerable dependency. See `docs/audit-demo.txt` for the original local finding,
`docs/audit-demo-isolated.txt` for the isolated fixture, and
`docs/audit-clean.txt` for the remediated application's report.

After publishing the files, open Actions, select **Security demonstration
(expected failure)**, and choose **Run workflow**. Capture the failed audit step
and its run URL. Then capture a passing **CI** run. Do not require the deliberately
failing demonstration workflow in branch protection. No hosted run is claimed
by these local reports.

## Challenge encountered

The starter's dependency graph contained incidental moderate findings in addition
to the deliberate lodash demonstration. Compatible dependency updates removed
those findings without changing the starter's API or test. The remaining
deliberate finding was captured before removing it from the application manifest.
This illustrates why fixing a scan requires reviewing the affected dependency
chain and rerunning both tests and the audit.

A second challenge was reproducibility: passing tests on the installed Node 26
did not verify the CI runtime. The same test and a live HTTP check were rerun
under Node 24.21.0. Docker is unavailable locally, so image and Compose behavior
must be verified on the hosted runner.

## Repository and hosted evidence

The submission fork is
[skyparado/devsecops-exam-starter](https://github.com/skyparado/devsecops-exam-starter),
forked from the official LSCS starter. This local workspace retains its original
remote. Hosted run links and branch protection status will be recorded in
[the validation record](docs/validation.md) after execution.

Required checks are **Test and build**, **Compose smoke test**,
**Dependency security**, and **Secret scan**. The separate manual demonstration
workflow is intentionally failing and must not be a required check.

## Local validation results

See `docs/validation.md` for executed checks and their limitations.
