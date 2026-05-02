---
name: gh-cli
description: Operate GitHub repositories, pull requests, issues, and releases via the `gh` CLI. Use when the user asks to push, create PR, manage issues, edit repo metadata, view repo, or any GitHub workflow operation, including Chinese requests such as “推送”“提 PR”“创建 issue”“改仓库描述”“查看仓库信息”.
---

# Gh CLI

Use `gh` CLI for GitHub operations. Verify `gh` is authenticated before running commands.

## Core Patterns

These patterns apply to **every** `gh` invocation. Read this section first.

### Structured output

Human output is column-formatted and unreliable for agents. Always prefer:
- `--json field1,field2,...` for structured JSON output.
- Run with `--json` and **no field list** to see available fields first.
- `--jq '<expr>'` for filtering inline, no separate `jq` pipe.
- `--template '<go-template>'` for shaped text output (but check `--help` first — `-T` collides with body template on `pr create` / `issue create`).

### Pagination

List commands cap results. Default is usually 30.
- Pass `-L N` (`--limit N`) to control count.
- `gh issue list` / `gh pr list` do not expose `totalCount` via `--json`. For a true total, use `gh api graphql`.
- For raw API pagination: `gh api --paginate <path>` + `--jq` + optionally `--slurp`.

### Repo targeting

`gh` infers repo from cwd's git remotes. Override with `--repo OWNER/REPO` (`-R`).

### Search vs list

- `gh search issues|prs|code|repos|commits|users` — cross-repo, full search syntax (`is:open`, `author:`, `label:`, `repo:owner/name`, `in:title`). Prefer for cross-repo or author/label filtering.
- `gh issue list --search "..."` / `gh pr list --search "..."` — same syntax but scoped to one repo.

### Fall back to `gh api`

When typed commands don't expose the data you need:
- `gh api repos/{owner}/{repo}/pulls/{n}/comments` — review-thread comments (not covered by `gh pr view --comments`).
- `gh api graphql -f query='...' -F var=value` — arbitrary GraphQL.
- `gh api repos/{owner}/{repo}/...` — REST shortcuts. `{owner}/{repo}` placeholders auto-filled from repo remotes.

### Authentication

`gh auth status` — basic check. `gh auth status --json` for structured output with host, user, and env var info.

### Interactivity

In non-TTY contexts `gh` already skips the pager, strips ANSI, and errors with clear messages instead of prompting. No need to set `GH_PAGER` or `--no-pager`.

## Common Operations

### Repository

`gh repo edit <owner/repo>` — edit metadata. Flags: `--description`, `--homepage`, `--add-topic`, `--remove-topic`, `--default-branch`.
`gh repo view <owner/repo>` — view details.

### Pull Requests

`gh pr create` — create. Flags: `--title`, `--body`, `--draft`, `--base`.
`gh pr list` — list. Flags: `--state open/closed/merged`, `--author`, `--assignee`, `--draft`, `-L`.
`gh pr view <number>` — view details. `--comments` for issue-level comments.
`gh pr diff <number>` — read diff without switching branches.
`gh pr checkout <number>` — switch to PR branch locally.

### Issues

`gh issue create` — create. Flags: `--title`, `--body`, `--label`, `--assignee`.
`gh issue list` — list. Flags: `--state`, `--label`, `--author`, `--assignee`, `-L`, `--search`.
`gh issue view <number>` — view details.
`gh issue close <number>` — close.
`gh issue reopen <number>` — reopen.

## When Not Sure

Run `gh <command> --help` before searching elsewhere. The CLI's own help output is the most up-to-date reference.
