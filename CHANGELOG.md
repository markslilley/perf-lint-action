# Changelog

All notable changes to the perf-lint GitHub Action are documented here.

## [1.2.0] — 2026-06-21

### Added
- **Automated PR comments.** On `pull_request` events the action posts a single
  quality-score comment (grade, score, violation count, hidden higher-tier count) and
  updates it in place on subsequent pushes. Controlled by `pr_comment` (default `true`)
  and `github_token`; requires `permissions: pull-requests: write`. Best-effort — a
  missing permission logs a hint and never fails the build.
- **`min_score` quality gate.** Fail the workflow when the overall quality score drops
  below a threshold (0–100), independent of the violation-based gate.

### Changed
- Failure is now enforced in a final step, so the PR comment and SARIF upload still run
  when a gate trips (the comment posts precisely when the build is red).

## [1.1.0] — 2026-06-21

### Added
- In-CI Pro/Team upgrade prompt. When perf-lint reports higher-tier violations
  that the free tier hides, the action now re-surfaces that count as a native
  GitHub `::notice::` annotation and a job-summary block linking to
  perflint.martkos-it.co.uk (UTM-tagged for click attribution). Best-effort —
  never fails the step, and stays silent when there are no hidden violations.

## [1.0.1] — 2026-02-28

### Fixed
- Install from `perf-lint-tool` on PyPI (PyPI normalises `perf-lint` and
  `perflint` to the same namespace; distribution renamed accordingly)

## [1.0.0] — 2026-02-28

### Added
- Initial release of the perf-lint GitHub Action
- Composite action wrapping `perf-lint check`
- Inputs: `paths`, `version`, `severity`, `format`, `config`,
  `ignore_rule`, `upload_sarif`, `fail_on_violations`, `api_key`
- Outputs: `violations`, `score`, `grade`, `sarif_file`
- SARIF upload to GitHub Code Scanning via `github/codeql-action/upload-sarif`
- Silent POST of full results to dashboard when `api_key` is set
- Major version tag (`v1`) updated automatically on each release
