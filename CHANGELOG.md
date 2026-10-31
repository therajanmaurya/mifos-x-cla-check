# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-10-31

### Added

- Composite action (`action.yml`) that checks whether a pull request's author is an active member of an approved
  contributors team, using a GitHub App installation token.
- Reusable workflow (`.github/workflows/cla-check.yml`) wrapping the action for the shortest caller setup.
- Inputs for organization, teams, CLA link, label, App owner and bot skipping; Mifos values as defaults.
- Outputs `verified` and `team`.
- Runner resolved from the `CI_RUNNER_LINUX` organization variable, falling back to `ubuntu-26.04`.

### Changed (compared with the inline `cla-check.yml` copies it replaces)

- Re-runs update a single marked comment instead of posting a new comment on every push.
- When an author becomes verified, their earlier comment is updated to say the check is clear.
- Team-lookup errors other than "not a member" (404) now fail loudly instead of being swallowed.

[1.0.0]: https://github.com/therajanmaurya/mifos-x-cla-check/releases/tag/v1.0.0
