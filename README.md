# mifos-x-cla-check

[![CI](https://github.com/therajanmaurya/mifos-x-cla-check/actions/workflows/ci.yml/badge.svg)](https://github.com/therajanmaurya/mifos-x-cla-check/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/therajanmaurya/mifos-x-cla-check)](https://github.com/therajanmaurya/mifos-x-cla-check/releases)
[![License: MPL 2.0](https://img.shields.io/badge/License-MPL_2.0-brightgreen.svg)](LICENSE)

A reusable **Contributor License Agreement (CLA) check** for Mifos repositories.

On every pull request it checks whether the author belongs to an approved contributors team. If they do, the
check passes. If they don't, it posts the CLA signing steps on the pull request, adds a `cla-required` label, and
fails the check — so the PR cannot be merged until the CLA is on file.

It replaces the `cla-check.yml` script that was copied into every repository. One copy means one place to fix a
bug, change the wording, or add a team.

- **Runs as a GitHub App bot** — comments and labels appear as the App, never a personal account.
- **One comment per PR** — re-runs update the existing comment instead of posting a new one on every push.
- **Clears itself** — once the author joins an approved team, the label is removed and the comment says so.
- **Configurable** — organization, teams, CLA link and label are inputs; the Mifos values are the defaults.

---

## Contents

- [Quick start](#quick-start)
- [Keep the existing required check name](#keep-the-existing-required-check-name)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [Setup](#setup)
- [Migrating from a copied `cla-check.yml`](#migrating-from-a-copied-cla-checkyml)
- [Security](#security)
- [Versioning](#versioning)
- [Development](#development)

---

## Quick start

Create `.github/workflows/cla-check.yml` in the repository you want to protect:

```yaml
name: CLA contributor check

on:
  pull_request_target:
    types: [opened, synchronize, reopened]

jobs:
  check-contributor-membership:
    runs-on: ${{ vars.CI_RUNNER_LINUX || 'ubuntu-26.04' }}
    permissions:
      pull-requests: write
      statuses: write
    steps:
      - uses: therajanmaurya/mifos-x-cla-check@v1
        with:
          app-id: ${{ secrets.CLA_APP_ID }}
          private-key: ${{ secrets.CLA_APP_PRIVATE_KEY }}
```

That is the whole file. Org, teams, CLA link and label default to the Mifos values.

### Even shorter: the reusable workflow

```yaml
name: CLA contributor check

on:
  pull_request_target:
    types: [opened, synchronize, reopened]

jobs:
  cla:
    uses: therajanmaurya/mifos-x-cla-check/.github/workflows/cla-check.yml@v1
    secrets: inherit
```

> ⚠️ With the reusable workflow, GitHub names the check **`cla / check-contributor-membership`**. If your
> branch protection or ruleset requires a check named exactly `check-contributor-membership`, use the composite
> action above — see the next section.

---

## Keep the existing required check name

Mifos rulesets require a status check called **`check-contributor-membership`**. GitHub names a check after the
**job** that produces it:

| How you call it | Check name |
|---|---|
| Composite action inside a job with id `check-contributor-membership` (Quick start) | `check-contributor-membership` ✅ |
| Reusable workflow from a job with id `cla` | `cla / check-contributor-membership` ⚠️ |

So keep the job id `check-contributor-membership` and use the composite action, and no ruleset needs to change.
If you prefer the reusable workflow, update the required check name in the ruleset in the same change.

---

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `app-id` | **yes** | — | ID of the GitHub App that reads team membership and comments (e.g. `${{ secrets.CLA_APP_ID }}`). |
| `private-key` | **yes** | — | That App's private key (e.g. `${{ secrets.CLA_APP_PRIVATE_KEY }}`). |
| `org` | no | `openMF` | Organization whose teams hold the approved contributors. |
| `app-owner` | no | value of `org` | Owner the App installation token is minted for, if different from `org`. |
| `teams` | no | `contributors,contracted-contributors` | Comma-separated team slugs. Membership in **any** one clears the check. |
| `cla-url` | no | the Mifos contributor agreement | Link to the CLA form, used in the comment. |
| `label` | no | `cla-required` | Label added while the CLA is missing, removed once verified. Set `''` to disable labelling. |
| `skip-bots` | no | `true` | Skip the check for pull requests opened by bot accounts (Dependabot, Renovate, …). |

The reusable workflow accepts `org`, `teams`, `cla-url`, `label` and `runs-on`, and requires the secrets
`CLA_APP_ID` and `CLA_APP_PRIVATE_KEY` (`secrets: inherit` passes them through).

## Outputs

| Output | Description |
|---|---|
| `verified` | `true` when the author is in an approved team (or is a skipped bot), otherwise `false`. |
| `team` | The team slug that verified the author, if any. |

```yaml
      - id: cla
        uses: therajanmaurya/mifos-x-cla-check@v1
        with:
          app-id: ${{ secrets.CLA_APP_ID }}
          private-key: ${{ secrets.CLA_APP_PRIVATE_KEY }}
      - if: always() && steps.cla.outputs.verified == 'true'
        run: echo "Verified via ${{ steps.cla.outputs.team }}"
```

---

## Setup

1. **GitHub App.** Install a GitHub App on the organization with these permissions:
   - *Organization → Members: Read* — to look up team membership.
   - *Repository → Pull requests: Read & write* and *Issues: Read & write* — to comment and label.
2. **Secrets.** Store the App's ID and private key as `CLA_APP_ID` and `CLA_APP_PRIVATE_KEY` — at the
   **organization** level so every repository can use them.
3. **Teams.** Add contributors with a signed CLA to the approved teams (`contributors` or
   `contracted-contributors` by default).
4. **Required check (recommended).** Require `check-contributor-membership` in the branch ruleset so an unsigned PR
   cannot be merged.
5. **Runner (optional).** The examples run on `vars.CI_RUNNER_LINUX`, falling back to `ubuntu-26.04`. Set the
   organization variable `CI_RUNNER_LINUX` to move every repository to another image in one place.

The `cla-required` label is created automatically the first time it is applied.

---

## Migrating from a copied `cla-check.yml`

Most Mifos repositories carry an inline copy of this check. Replace the whole file with the
[Quick start](#quick-start) example:

- the job id stays `check-contributor-membership`, so **required checks keep working with no ruleset change**;
- the secrets stay `CLA_APP_ID` / `CLA_APP_PRIVATE_KEY`;
- behaviour is the same, except that re-runs now update one comment instead of adding a new one on every push, and
  a verified author's earlier comment is updated to say the check is clear.

---

## Security

The check runs on **`pull_request_target`** so it can comment on pull requests from forks. That event runs with
the base repository's permissions and secrets, so this action:

- **never checks out or executes code from the pull request** — it only reads the PR's metadata (author, number);
- uses a short-lived **GitHub App installation token** instead of a personal access token;
- requests only `pull-requests: write` and `statuses: write` on the job.

Keep it that way in the calling workflow: do not add a step that checks out the PR's head ref to the same job.

---

## Versioning

Releases follow [Semantic Versioning](https://semver.org). Pin the rolling major tag **`@v1`** to receive fixes
and backwards-compatible features automatically, or an exact tag (e.g. `@v1.0.0`) to control every update. A
breaking change ships as a new major (`@v2`), so `@v1` callers are never broken. See [CHANGELOG.md](CHANGELOG.md).

## Development

- `action.yml` — the composite action (the logic lives here).
- `.github/workflows/cla-check.yml` — the reusable-workflow wrapper around it.
- `.github/workflows/ci.yml` — lints both on every push and pull request.

Lint locally with [actionlint](https://github.com/rhysd/actionlint):

```bash
actionlint
```

## License

[Mozilla Public License 2.0](LICENSE).
