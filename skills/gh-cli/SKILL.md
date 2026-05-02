---
name: gh-cli
description: Operate GitHub repositories, pull requests, issues, and releases via the `gh` CLI. Use when the user asks to push, create PR, manage issues, edit repo metadata, view repo, or any GitHub workflow operation, including Chinese requests such as “推送”“提 PR”“创建 issue”“改仓库描述”“查看仓库信息”.
---

# Gh CLI

Use `gh` CLI for GitHub operations.

## Use This Skill

- Push, commit, or any git remote operation needing GitHub interaction
- Create / list / view / close issues or pull requests
- Edit repository metadata (description, topics, homepage)
- Search across repos for issues, PRs, code, or users
- View repository details or authentication status
- "推送" / "提 PR" / "创建 issue" / "改仓库描述" / "查看仓库信息"

## Do Not Use

- Operations that do not involve GitHub (local git, other forges)
- Raw `git` operations that do not need GitHub API interaction
- Managing Agent Skills (use `gh skill` subcommand, not this skill)

## Input Contract

**Required Inputs:**
- A GitHub operation intent (create PR, list issues, edit repo, etc.)

**Optional Inputs:**
- Target repo (`owner/repo`) — if omitted, inferred from cwd's git remotes
- Issue or PR number
- Flags specific to the operation

**Missing input handling:**
- **No specific repo**: Use git remote from cwd. If no remote found, ask user.
- **No PR/issue number for view**: List recent items first, then ask which one.
- **Ambiguous intent**: Run `gh <command> --help` to clarify options.

## Execution Protocol

1. **Verify authentication**: `gh auth status`. If unauthenticated, report and stop.
2. **Choose the right command path**:
   - Typed commands (`gh pr`, `gh issue`, `gh repo`) for standard operations.
   - `gh search` for cross-repo queries.
   - `gh api` for anything typed commands do not expose.
3. **Use structured output**: Always prefer `--json` + `--jq` over parsing human-readable text.
4. **Handle pagination**: Pass `-L N` to control result count. For `totalCount`, use `gh api graphql`.
5. **Target repo**: Override with `--repo OWNER/REPO` (`-R`) when not operating on cwd's repo.
6. **Execute and validate**: Run the command. If it fails, interpret the error and retry with adjusted parameters.

### Core Patterns

#### Structured output

- `--json field1,field2,...` for structured JSON.
- Run with `--json` and **no field list** to see available fields first.
- `--jq '<expr>'` for filtering inline, no separate `jq` pipe.
- `--template '<go-template>'` for shaped text output.

#### Pagination

List commands cap results. Default is usually 30. Pass `-L N` (`--limit N`) to control count. `gh issue list` / `gh pr list` do not expose `totalCount` via `--json`. For a true total, use `gh api graphql`. For raw API pagination: `gh api --paginate <path>` + `--jq` + optionally `--slurp`.

#### Repo targeting

`gh` infers repo from cwd's git remotes. Override with `--repo OWNER/REPO` (`-R`).

#### Search vs list

- `gh search issues|prs|code|repos|commits|users` — cross-repo, full search syntax. Prefer for cross-repo or author/label filtering.
- `gh issue list --search "..."` / `gh pr list --search "..."` — same syntax but scoped to one repo.

#### Fall back to `gh api`

When typed commands don't expose the data you need. `{owner}/{repo}` placeholders auto-filled from repo remotes.

#### Interactivity

In non-TTY contexts `gh` already skips the pager, strips ANSI, and errors with clear messages instead of prompting.

When not sure about flags or available fields, run `gh <command> --help` before searching elsewhere.

### Common Operations

`gh repo edit <owner/repo>` — edit metadata. Flags: `--description`, `--homepage`, `--add-topic`, `--remove-topic`, `--default-branch`.
`gh repo view <owner/repo>` — view details.

`gh pr create` — create. Flags: `--title`, `--body`, `--draft`, `--base`.
`gh pr list` — list. Flags: `--state open/closed/merged`, `--author`, `--assignee`, `--draft`, `-L`.
`gh pr view <number>` — view details. `--comments` for issue-level comments.
`gh pr diff <number>` — read diff without switching branches.
`gh pr checkout <number>` — switch to PR branch locally.

`gh issue create` — create. Flags: `--title`, `--body`, `--label`, `--assignee`.
`gh issue list` — list. Flags: `--state`, `--label`, `--author`, `--assignee`, `-L`, `--search`.
`gh issue view <number>` — view details.
`gh issue close <number>` — close.
`gh issue reopen <number>` — reopen.

## Failure Handling

- **Not authenticated**: `gh auth status` fails. Report to user and ask them to run `gh auth login`.
- **404 / not found**: Repo or resource does not exist. Verify `owner/repo` spelling and user's access.
- **API rate limited**: Wait and retry, or inform user if limit is exceeded.
- **Command not found**: `gh` not installed. Ask user to install GitHub CLI.
- **Network error**: Report the exact error. Do not guess the output.
- **Ambiguous result from no field list**: Re-run with explicit fields.

## Output Contract

When using this skill, include in your response:
- What command was executed (or attempted)
- The structured result (key fields, not raw JSON dump)
- If failed, the exact error and suggested next step

## Integration

- `git-commit`: Upstream — commit happens before `gh pr create`.
- `finishing-a-development-branch`: Upstream — branch wrap-up may involve `gh pr create` or `gh pr merge`.
- `chinese-git-workflow`: Adjacent — use when the remote is on a domestic platform.
