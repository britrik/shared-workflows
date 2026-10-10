# shared-workflows

Reusable GitHub Actions workflows for britrik repos.

## Gitleaks secret scan

`.github/workflows/gitleaks.yml` scans for committed secrets with the [gitleaks](https://github.com/gitleaks/gitleaks) CLI (version pinned and SHA-256 verified).

| Trigger | What is scanned |
|---|---|
| `pull_request` | the PR's commits (base..head) |
| `push` | the pushed commits (before..after); full history on a new branch |
| `schedule`, `workflow_dispatch` | the full history |

Merge commits are scanned with `--diff-merges=remerge`, so a secret added while resolving a conflict is caught.

Findings are redacted; the job summary lists rule, file, line, commit and fingerprint only. The job fails when anything is found.

Call it from a repo's `.github/workflows/gitleaks.yml`:

```yaml
name: Secret scan
on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: "17 5 * * 1"
  workflow_dispatch:
permissions:
  contents: read
concurrency:
  group: gitleaks-${{ github.event_name }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  gitleaks:
    uses: britrik/shared-workflows/.github/workflows/gitleaks.yml@main
```

A `.gitleaks.toml` at the repo root is picked up automatically (extend the default rules there). To accept a confirmed false positive, add its fingerprint to `.gitleaksignore`. If a real secret is found, rotate it first.

Inputs: `gitleaks-version` and `gitleaks-sha256` (the linux_x64 tarball checksum) to move to another release.
