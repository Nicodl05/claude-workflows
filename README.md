# claude-workflows

Reusable GitHub Actions workflows for automating issue resolution with Claude Code.

## What this repo does

This repository exposes a reusable workflow (`claude-issue.yml`) that, when called from a project repository, automatically:

1. Creates a branch named `feature/issue-{id}-{slug}` from main.
2. Runs Claude Code in headless mode with the issue context as prompt.
3. Commits any changes produced by Claude Code.
4. Opens a pull request referencing the issue with the label `awaiting-review`.
5. Posts a comment on the issue with the pull request URL.

Claude Code follows the quality rules defined in the global `CLAUDE.md`:
- Reads the Graphify graph of the target repository if available.
- Fetches up-to-date documentation via the context7 MCP server.
- Writes or updates tests for every change.
- Creates an ADR in `docs/adr/` when an architecture decision is made.
- Uses conventional commits and English-only identifiers.

## How to add the workflow to a new project

### 1. Copy the caller workflow

Copy `docs/caller-template.yml` from this repository into your project at:

```
.github/workflows/claude-issue.yml
```

The file content:

```yaml
name: Claude Issue Handler

on:
  issues:
    types: [labeled]

jobs:
  claude:
    if: github.event.label.name == 'claude'
    uses: Nicodl05/claude-workflows/.github/workflows/claude-issue.yml@main
    with:
      issue_number: ${{ github.event.issue.number }}
      issue_title: ${{ github.event.issue.title }}
      issue_body: ${{ github.event.issue.body || '' }}
      issue_labels: ${{ toJson(github.event.issue.labels.*.name) }}
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 2. Add the ANTHROPIC_API_KEY secret

In your project repository, go to **Settings > Secrets and variables > Actions** and create a secret:

| Name | Value |
|---|---|
| `ANTHROPIC_API_KEY` | Your Anthropic API key |

### 3. Create the `claude` label

In your project repository, go to **Issues > Labels** and create a label named exactly `claude`.

Once both are in place, adding the `claude` label to any open issue will trigger the workflow.

## Requirements

- The calling repository must grant the workflow `contents: write`, `pull-requests: write`, and `issues: write` permissions.
- The default branch must be named `main`.
- The `awaiting-review` label must exist in the calling repository if you want the pull request to be tagged automatically (the workflow will not fail if it is missing, the label will simply be skipped).
