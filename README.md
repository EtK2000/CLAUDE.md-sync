# CLAUDE.md-sync

> **Warning:** This project is vibecoded slop. Proceed with caution.

A GitHub Action that keeps `CLAUDE.md` in sync across repos. This repo holds the canonical
source-of-truth; consumer repos use the action to automatically open PRs when the file drifts.

## Quick Start

Add this workflow to any consumer repo at `.github/workflows/sync-claude-md.yml`:

```yaml
name: Sync CLAUDE.md
on:
  schedule:
    - cron: '0 9 * * 1'   # Every Monday at 9 AM UTC
  workflow_dispatch:        # Manual trigger

permissions:
  contents: write
  issues: write
  pull-requests: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: EtK2000/CLAUDE.md-sync@v1
        with:
          source-repo: EtK2000/CLAUDE.md
```

## Inputs

| Input           | Default                               | Description                                                      |
|-----------------|---------------------------------------|------------------------------------------------------------------|
| `source-repo`   | **required**                          | `owner/repo` holding the canonical file                          |
| `source-ref`    | `master`                              | Branch or tag to fetch from                                      |
| `source-path`   | `CLAUDE.md`                           | Path in the source repo                                          |
| `target-path`   | `CLAUDE.md`                           | Where to place the file locally                                  |
| `target-branch` | *(auto-detected)*                     | Branch the PR targets (defaults to repo default branch)          |
| `pr-title`      | `chore: sync CLAUDE.md from upstream` | PR title                                                         |
| `branch-name`   | `claude-md-sync/update`               | Branch used for the sync PR                                      |
| `pr-labels`     | `automated`                           | Comma-separated PR labels                                        |
| `token`         | `${{ github.token }}`                 | GitHub token (see [Required Permissions](#required-permissions)) |

## Required Permissions

The workflow `permissions` block needs:

- `contents: write` -- to push the sync branch
- `pull-requests: write` -- to create/comment on PRs
- `issues: write` -- to create PR labels (via `gh label create`)

The default `GITHUB_TOKEN` works if you set the `permissions` block as shown in the Quick Start.

### Using a custom token

If the default `GITHUB_TOKEN` doesn't work for your setup (e.g. organization policies, fine-grained
access control), you can pass a Personal Access Token instead:

```yaml
      - uses: EtK2000/CLAUDE.md-sync@v1
        with:
          source-repo: my-org/claude-configs
          token: ${{ secrets.SYNC_TOKEN }}
```

A fine-grained PAT scoped to the consumer repo needs **Contents**, **Pull requests**, and
**Issues** set to **Read and write**.

## Behavior

- **Files match** -- no-op, exits cleanly.
- **Files differ, no open PR** -- creates a new branch, commits the updated file, opens a PR.
- **Files differ, PR already open** -- force-pushes the branch and comments on the existing PR.
- **Source fetch fails** -- exits with an error.

## Setup (Consumer Repos)

### 1. Enable Pull Requests

The consumer repo must have Pull Requests enabled. This is on by default, but if it was disabled:

**Settings > General > Features > Pull Requests**

Without this, PR creation fails with `Resource not accessible by integration (createPullRequest)`.

### 2. Allow GitHub Actions to open PRs

In each consumer repo, go to **Settings > Actions > General > Workflow permissions** and enable:

- **"Read and write permissions"**
- **"Allow GitHub Actions to create and approve pull requests"**

Without the second checkbox, the action will fail when trying to open PRs.

## Troubleshooting

| Error                                                                  | Cause                                                                                                                                                       |
|------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Resource not accessible by integration (createPullRequest)`           | Pull Requests are disabled on the repo, or the "Allow GitHub Actions to create and approve pull requests" checkbox is unchecked. Check both settings above. |
| `Resource not accessible by personal access token (createPullRequest)` | The PAT is missing `Pull requests: write` permission, or Pull Requests are disabled on the repo.                                                            |
| `Failed to fetch source file (HTTP 404)`                               | The `source-repo`, `source-ref`, or `source-path` is wrong, or the source repo is private.                                                                  |

## Development

Run ShellCheck locally:

```bash
shellcheck sync.sh
```

### Security

This repo has **"Require actions to be pinned to a full-length commit SHA"** enabled. When adding or
updating action dependencies, always pin to a commit SHA:

```yaml
# Good
- uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4

# Bad
- uses: actions/checkout@v4
```
