# perf-lint action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-perf--lint-2ea44f?logo=github)](https://github.com/marketplace/actions/perf-lint)
[![Release](https://img.shields.io/github/v/release/markslilley/perf-lint-action?sort=semver&logo=github)](https://github.com/markslilley/perf-lint-action/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Rules](https://img.shields.io/badge/rules-53-orange)
![Tools](https://img.shields.io/badge/JMeter%20%C2%B7%20k6%20%C2%B7%20Gatling-supported-blueviolet)

> **The only dedicated linter for performance test scripts** — JMeter, k6, and Gatling —
> as a GitHub Action. Catch quality issues in your load tests *before* they reach the
> load environment, right inside CI.

Most teams lint their application code and never lint the load tests that decide whether
that code ships. A test with no think times, no assertions, or hardcoded data produces
confident-looking numbers that are quietly wrong. `perf-lint` is static analysis for the
test scripts themselves — **53 rules** across JMeter `.jmx`, k6 `.js`/`.ts`, and Gatling
`.scala`/`.kt`, running on every push and pull request.

Scripts never leave your runner. All rules run locally; nothing is uploaded unless you
opt in to the dashboard.

---

## What it catches

```text
$ perf-lint check tests/k6/ecommerce.js

  ecommerce.js (k6)  [C 70/100]

  W [K6001] MissingThinkTime    — No sleep() calls found. (line 1)             [fixable]
  W [K6004] MissingThresholds   — No thresholds defined in options. (line 1)   [fixable]
  W [K6007] MissingTeardown     — setup() exported but no teardown(). (line 3) [fixable]

  Violations    3  (0 errors, 3 warnings)
  Quality score C  70/100
```

Missing think times, absent thresholds/assertions, hardcoded values, missing cache/cookie
managers, unrealistic ramp patterns, correlation gaps, and 45+ more — each with a rule ID,
severity, and (where possible) an auto-fix. Every file gets a 0–100 quality score and an
A–F grade you can use as a CI gate.

---

## Quick start

```yaml
# .github/workflows/perf-lint.yml
name: perf-lint

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: markslilley/perf-lint-action@v1
        with:
          paths: tests/performance/
```

That's the whole setup. The step fails when warnings or errors are found; set
`fail_on_violations: 'false'` to report without blocking while you adopt it.

> ⭐ If this saves you a bad load test, **star the repo** — it helps other teams find it.

## Why perf-lint

- **Tool-agnostic** — one linter for JMeter, k6, *and* Gatling. No lock-in.
- **Private by default** — all 53 rules run on your runner; scripts are never uploaded.
- **Auto-fixable** — many rules insert the missing manager/threshold/think-time for you.
- **CI-native** — quality score as a gate, SARIF into the Security tab, step outputs for
  downstream jobs.
- **The only one of its kind** — no other dedicated linter covers performance test scripts
  across all three major tools.

---

## With dashboard (Pro / Team violations)

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/performance/
    api_key: ${{ secrets.PERF_LINT_API_KEY }}
```

The terminal always shows the **18 free-tier rules**. Pro and Team violations are counted
and hinted at; the full breakdown appears in your dashboard. Get an API key at
[perflint.martkos-it.co.uk](https://perflint.martkos-it.co.uk) and store it as a repository
secret — `secrets.PERF_LINT_API_KEY`.

## With GitHub Code Scanning (SARIF)

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - uses: markslilley/perf-lint-action@v1
        with:
          paths: tests/performance/
          upload_sarif: 'true'
          api_key: ${{ secrets.PERF_LINT_API_KEY }}
```

Violations appear inline on pull requests under the **Security** tab.

## Pull request comments

On `pull_request` events the action posts a single quality-score comment and updates it
in place on every push — no comment spam. Give the job permission to write PR comments:

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4

      - uses: markslilley/perf-lint-action@v1
        with:
          paths: tests/performance/
          min_score: '60'   # also fail the PR if the score drops below 60
```

The comment shows the overall grade/score, the violation count, and — when there are
higher-tier issues — how many are hidden. Set `pr_comment: 'false'` to disable it.

---

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `paths` | No | `.` | Files or directories to analyse (space-separated) |
| `version` | No | `latest` | perf-lint version to install |
| `severity` | No | `warning` | Minimum severity for non-zero exit: `error` \| `warning` \| `info` |
| `format` | No | `text` | Primary output format: `text` \| `json` \| `sarif` |
| `config` | No | — | Path to `.perf-lint.yml` config file |
| `ignore_rule` | No | — | Comma-separated rule IDs to ignore (e.g. `JMX001,K6003`) |
| `upload_sarif` | No | `false` | Upload SARIF to GitHub Code Scanning |
| `fail_on_violations` | No | `true` | Fail the step when violations are found |
| `min_score` | No | — | Fail when the overall quality score is below this (0–100) |
| `pr_comment` | No | `true` | On PRs, post/update a score comment (needs `pull-requests: write`) |
| `github_token` | No | `${{ github.token }}` | Token used to post the PR comment |
| `api_key` | No | — | perf-lint API key — store as a repository secret |

## Outputs

| Output | Description |
|--------|-------------|
| `violations` | Total free-tier violations found |
| `score` | Overall quality score (0–100) |
| `grade` | Overall quality grade (A–F) |
| `sarif_file` | Path to SARIF file (when `upload_sarif: true`) |

---

## Tier model

perf-lint runs **all 53 rules** locally — scripts never leave your machine.
The terminal output shows **18 free-tier rules**. Pro (22) and Team (13) violations
are counted and hinted at; the full picture is visible in your dashboard.

| Tier | Rules | What it covers |
|------|-------|---------------|
| Free | 18 | Basic hygiene: missing managers, think time, thresholds |
| Pro | 22 | All ERRORs + production-critical warnings |
| Team | 13 | Advanced patterns: correlation, memory, architecture |

See [perflint.martkos-it.co.uk](https://perflint.martkos-it.co.uk) for Pro and Team plans.

When higher-tier violations are present, the action adds a short summary to the run's
**job summary** (and a notice annotation) telling you how many Pro/Team issues are hidden —
so you can see what you're missing without leaving GitHub. It never fails the build and
stays silent when there's nothing hidden.

## Examples

### Fail only on errors

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    severity: error
```

### Pin a specific version

```yaml
- uses: markslilley/perf-lint-action@v1.0.0
  with:
    paths: tests/
```

### Report-only mode (never fail CI)

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    fail_on_violations: 'false'
```

### Ignore specific rules

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    ignore_rule: 'JMX013,GAT005'
```

### Use a config file

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    config: .perf-lint.yml
```

---

## Permissions

| Feature | Permission needed |
|---------|------------------|
| Basic lint | None (default) |
| SARIF upload | `security-events: write` |

---

## Part of the Martkos performance-testing ecosystem

perf-lint is the script-quality layer of a full performance-testing toolchain — data
generation, scripting, execution, migration, and reporting across JMeter, k6, and Gatling.
See [martkos-it.co.uk](https://martkos-it.co.uk) and the core CLI at
[markslilley/perf-lint](https://github.com/markslilley/perf-lint)
(`pip install perf-lint-tool`).

## License

MIT — see [LICENSE](LICENSE).
