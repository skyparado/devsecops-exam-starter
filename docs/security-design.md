# Security design

| Risk | Control | Limit |
| --- | --- | --- |
| Known vulnerable application packages | npm audit includes development and production dependencies; high/critical findings fail CI | Advisory coverage is incomplete; a clean scan does not prove safety |
| Vulnerable OS or bundled image libraries | Trivy scans the built runtime image; fixable high/critical findings fail CI | Unfixed findings do not block under the chosen policy; review upstream advisories separately |
| Accidentally committed credentials | Trivy secret scan checks working-tree files | Pattern detection can miss secrets and does not scan deleted Git history |
| Compromised container process | Non-root node user; Compose drops capabilities and uses a read-only filesystem | This is defense in depth, not a sandbox guarantee |
| Accidental inclusion of local secrets/tools | Docker context allowlist and explicit runtime COPY | Review the allowlist when adding application files |
| Dependency/image/action drift | Lockfile, npm ci, digest-pinned Node image, commit-pinned Actions; Dependabot proposes updates | Pinned versions still require review and regular updates |
| Untrusted pull requests | Read-only workflow token, no deployment secrets, no pull_request_target workflow | Do not add privileged credentials to PR execution |

## Why these checks

npm audit is a small, native dependency gate with no separate scanner account.
Trivy adds coverage of the actual image, including OS packages, and repository
secret patterns. Both return a failing status rather than allowing errors to
be ignored. Separate jobs make failures easier to diagnose.

The build job validates runtime identity and HTTP behavior before scanning the
same image. Compose checks cover the extra service/network configuration.
The main CI workflow must pass before a pull request can merge once branch
protection is enabled. Nothing deploys the application automatically.

Sources: [Trivy Action](https://github.com/aquasecurity/trivy-action),
[npm audit](https://docs.npmjs.com/cli/v11/commands/npm-audit).

## Safe demonstration

The intentionally vulnerable lodash package is unused and development-only.
Commit A adds it to the application's manifest and lockfile so the normal
dependency job fails. Commit B removes it and reruns the same workflow.
The isolated security-demo fixture remains as a manual reproduction tool and
never enters the container. Audit detection requires no exploitation.
