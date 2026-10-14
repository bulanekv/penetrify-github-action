# Penetrify GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Penetrify-blue?style=flat&logo=github)](https://github.com/marketplace/actions/penetrify-security-scan)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**AI-driven autonomous penetration testing in your CI/CD pipeline.**

Penetrify scans your application for real-world vulnerabilities — SQL injection, XSS, authentication flaws, API security issues, and more — and reports findings directly in GitHub's Security tab.

---

## Quick Start

```yaml
- name: Run Penetrify Scan
  uses: bulanekv/penetrify-github-action@v1
  with:
    api-key: ${{ secrets.PENETRIFY_API_KEY }}
    application-id: 'app_xxxxxxxxxxxx'   # from the Penetrify dashboard
    target: 'https://staging.your-app.com'
    authorization-confirmed: true        # you own / are authorized to test the target
```

That's it. Results appear in your repository's **Security → Code scanning** tab.

## How It Works

1. **Triggers a scan** against your target via the Penetrify API
2. **Polls for results** until the scan completes (or times out)
3. **Generates a SARIF report** and uploads it to GitHub Security
4. **Fails the build** if vulnerabilities exceed your severity threshold

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | ✅ | — | Your Penetrify API key (store as a GitHub Secret) |
| `application-id` | ✅ | — | Penetrify application ID (dashboard → application → ID) |
| `authorization-confirmed` | ✅ | `false` | Must be `true`: you own or are explicitly authorized to test the target |
| `target` | ❌ | — | URL to scan; overrides the application's default target |
| `scan-type` | ❌ | `standard` | `quick` (0.5 credits), `standard` (1), `deep` (2) |
| `api-base` | ❌ | `https://api.penetrify.cloud/api/v1` | API base URL (staging / self-hosted only) |
| `severity-threshold` | ❌ | `high` | Fail build on this severity or above: `critical`, `high`, `medium`, `low` |
| `wait-for-results` | ❌ | `true` | Wait for completion (`true`) or fire-and-forget (`false`) |
| `timeout` | ❌ | `1800` | Max wait time in seconds |
| `upload-sarif` | ❌ | `true` | Upload results to GitHub Security tab |
| `config-file` | ❌ | — | Path to a Penetrify config JSON file |

## Outputs

| Output | Description |
|--------|-------------|
| `test-id` | Unique scan (test) identifier |
| `status` | Final status: `completed`, `failed`, `timeout` |
| `findings-count` | Total number of findings |
| `critical-count` | Number of critical findings |
| `high-count` | Number of high findings |
| `report-url` | Link to the full report on penetrify.cloud |
| `sarif-file` | Path to the generated SARIF file |

## Examples

### Basic: Scan on push to main

```yaml
name: Security Scan
on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: bulanekv/penetrify-github-action@v1
        with:
          api-key: ${{ secrets.PENETRIFY_API_KEY }}
          application-id: 'app_xxxxxxxxxxxx'
          authorization-confirmed: true
          target: 'https://staging.myapp.com'
          severity-threshold: 'high'
```

### Quick scan on PRs, full scan on deploy

```yaml
name: Security
on:
  pull_request:
    branches: [main]

jobs:
  quick-scan:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: bulanekv/penetrify-github-action@v1
        with:
          api-key: ${{ secrets.PENETRIFY_API_KEY }}
          application-id: 'app_xxxxxxxxxxxx'
          authorization-confirmed: true
          target: 'https://preview-${{ github.event.pull_request.number }}.myapp.com'
          scan-type: 'quick'
          severity-threshold: 'critical'
```

### Scheduled weekly scan

```yaml
name: Weekly Security Audit
on:
  schedule:
    - cron: '0 6 * * 1'

jobs:
  audit:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: bulanekv/penetrify-github-action@v1
        with:
          api-key: ${{ secrets.PENETRIFY_API_KEY }}
          application-id: 'app_xxxxxxxxxxxx'
          authorization-confirmed: true
          target: 'https://app.mycompany.com'
          scan-type: 'standard'
          severity-threshold: 'medium'
          timeout: '3600'
```

### Fire-and-forget (async)

```yaml
- uses: bulanekv/penetrify-github-action@v1
  with:
    api-key: ${{ secrets.PENETRIFY_API_KEY }}
    target: 'https://staging.myapp.com'
    wait-for-results: 'false'
```

## GitHub Security Integration

When `upload-sarif` is enabled (default), scan findings appear natively in GitHub:

- **Security tab → Code scanning alerts** — browse, filter, and manage findings
- **Pull request checks** — see new vulnerabilities introduced by a PR
- **Alert management** — dismiss false positives, assign to team members, track resolution

> **Required permission:** Your workflow must include `permissions: security-events: write` for SARIF upload to work.

## Getting Your API Key

1. Sign up at [penetrify.cloud](https://penetrify.cloud)
2. Navigate to **Settings → API Keys**
3. Create a new key and add it as a GitHub Secret named `PENETRIFY_API_KEY`

## Support

- Documentation: [docs.penetrify.cloud](https://docs.penetrify.cloud)
- Issues: [GitHub Issues](https://github.com/bulanekv/penetrify-github-action/issues)
- Email: support@penetrify.cloud

## License

MIT — see [LICENSE](LICENSE) for details.
