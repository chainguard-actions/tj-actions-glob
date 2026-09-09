<!-- markdownlint-disable -->

# Hardening Report: tj-actions--glob/v22.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--glob/v22.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags or branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks.

.github/workflows/codacy-analysis.yml:
  - codacy/codacy-analysis-cli-action@v4.4.5
  - github/codeql-action/upload-sarif@v3

.github/workflows/codeql.yml:
  - github/codeql-action/init@v3
  - github/codeql-action/autobuild@v3
  - github/codeql-action/analyze@v3

.github/workflows/sync-release-version.yml:
  - tj-actions/release-tagger@v4
  - tj-actions/sync-release-version@v13
  - tj-actions/git-cliff@v1
  - tj-actions/semver-diff@v3
  - actions/setup-node@v4
  - peter-evans/create-pull-request@v7.0.8

.github/workflows/test.yml:
  - actions/setup-node@v4.2.0
  - tj-actions/eslint-changed-files@v25
  - tj-actions/verify-changed-files@v20
  - ad-m/github-push-action@master  (branch ref!)
  - actions/upload-artifact@v4
  - codacy/codacy-coverage-reporter-action@v1
  - actions/download-artifact@v4

.github/workflows/update-readme.yml:
  - tj-actions/auto-doc@v3
  - tj-actions/remark@v3
  - tj-actions/verify-changed-files@v20
  - peter-evans/create-pull-request@v7

Locations:

- `.github/workflows/codacy-analysis.yml:35`
- `.github/workflows/codacy-analysis.yml:46`
- `.github/workflows/codeql.yml:38`
- `.github/workflows/codeql.yml:47`
- `.github/workflows/codeql.yml:55`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:16`
- `.github/workflows/sync-release-version.yml:23`
- `.github/workflows/sync-release-version.yml:26`
- `.github/workflows/sync-release-version.yml:28`
- `.github/workflows/sync-release-version.yml:44`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:47`
- `.github/workflows/test.yml:67`
- `.github/workflows/test.yml:72`
- `.github/workflows/test.yml:80`
- `.github/workflows/test.yml:85`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:18`
- `.github/workflows/update-readme.yml:21`
- `.github/workflows/update-readme.yml:32`

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings, allowing injection of arbitrary shell content.

1. In sync-release-version.yml: `run: yarn publish --${{ steps.semver-diff.outputs.release_type }} --no-git-tag-version` — the step output `release_type` is interpolated directly into the shell command without quoting or env-var indirection.

2. In test.yml: Multiple 'Show output' steps interpolate `steps.*.outputs.paths` and `steps.*.outputs.paths-output-file` directly into echo and cat commands, e.g.:
   `echo "${{ steps.glob-all-files.outputs.paths }}"`
   `cat "${{ steps.glob-all-files.outputs.paths-output-file }}"`
   `echo "${{ steps.glob-invalid.outputs.paths-output-file }}"`
These step outputs originate from the action under test and could contain shell metacharacters.

Locations:

- `.github/workflows/sync-release-version.yml:34`
- `.github/workflows/test.yml:100`
- `.github/workflows/test.yml:101`
- `.github/workflows/test.yml:112`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:124`
- `.github/workflows/test.yml:125`
- `.github/workflows/test.yml:135`
- `.github/workflows/test.yml:136`
- `.github/workflows/test.yml:147`
- `.github/workflows/test.yml:148`
- `.github/workflows/test.yml:159`
- `.github/workflows/test.yml:160`
- `.github/workflows/test.yml:171`
- `.github/workflows/test.yml:172`
- `.github/workflows/test.yml:183`
- `.github/workflows/test.yml:184`
- `.github/workflows/test.yml:193`
- `.github/workflows/test.yml:194`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows run with the default (potentially write) token permissions, violating the principle of least privilege.

- codacy-analysis.yml: no permissions at top level or job level
- sync-release-version.yml: no permissions at top level or job level
- test.yml: no permissions at top level or job level (build and test jobs both lack permissions)
- update-readme.yml: no permissions at top level or job level

Only codeql.yml has job-level permissions defined.

Locations:

- `.github/workflows/codacy-analysis.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types across 5 workflow files:

1. unpinned-uses: Pinned all 22 action references to full 40-char commit SHAs with original tags as comments. Used lookup_action_sha for each: codacy/codacy-analysis-cli-action@97bf5df, github/codeql-action/*@6f5948d, tj-actions/release-tagger@1a9264b, tj-actions/sync-release-version@2a7ef0d, tj-actions/git-cliff@75599f7, tj-actions/semver-diff@398e876, actions/setup-node@49933ea (v4) and @1d0ff46 (v4.2.0), peter-evans/create-pull-request@271a8d0 (v7.0.8) and @22a9089 (v7), tj-actions/eslint-changed-files@536c35c, tj-actions/verify-changed-files@a1c6ace, ad-m/github-push-action@881a632, actions/upload-artifact@ea165f8, codacy/codacy-coverage-reporter-action@89d6c85, actions/download-artifact@d3f86a1, tj-actions/auto-doc@b10ceed, tj-actions/remark@10fc407.

2. script-injection: Moved all ${{ }} expressions from run: shell strings to env: blocks. In sync-release-version.yml, the release_type step output is now in RELEASE_TYPE env var. In test.yml, all 9 'Show output' steps and the 'Verify' step now use PATHS and PATHS_OUTPUT_FILE env vars instead of inline expressions.

3. missing-permissions: Added top-level permissions blocks to codacy-analysis.yml (contents:read, security-events:write), sync-release-version.yml (contents:write, pull-requests:write, packages:write), test.yml (contents:write, pull-requests:read), and update-readme.yml (contents:write, pull-requests:write). codeql.yml already had job-level permissions.

