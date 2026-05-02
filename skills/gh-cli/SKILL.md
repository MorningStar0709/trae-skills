---
name: gh-cli
description: Operate GitHub repositories, pull requests, issues, and releases via the `gh` CLI. Use when the user asks to push, create PR, manage issues, edit repo metadata, view repo, or any GitHub workflow operation, including Chinese requests such as “推送”“提 PR”“创建 issue”“改仓库描述”“查看仓库信息”.
---

# Gh CLI

Use `gh` CLI for GitHub operations. Verify `gh` is authenticated before running commands.

## Common Operations

### Repository

`gh repo edit <owner/repo>` — edit repository metadata. Common flags: `--description`, `--homepage`, `--add-topic`, `--remove-topic`, `--default-branch`.

`gh repo view <owner/repo>` — view repository details.

### Pull Requests

`gh pr create` — create a pull request.
`gh pr list` — list pull requests. Flags: `--state open/closed/merged`, `--author`, `--assignee`, `--draft`.
`gh pr view <number>` — view PR details.
`gh pr checkout <number>` — checkout a PR locally.

### Issues

`gh issue create` — create an issue.
`gh issue list` — list issues. Flags: `--state`, `--label`, `--author`, `--assignee`.
`gh issue view <number>` — view issue details.
`gh issue close <number>` — close an issue.

### General

`gh auth status` — verify authentication status.
`gh --help` — general help.
`gh <command> --help` — command-specific help.

## When Not Sure

Run `gh <command> --help` before searching elsewhere. The CLI's own help output is the most up-to-date reference.
