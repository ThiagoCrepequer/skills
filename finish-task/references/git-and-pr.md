# Git isolation, atomic commits, and PRs

## Establish the base and scope

Read repository guidance before choosing a workflow. Inspect repository status, branch/upstream, remotes, worktrees, staged/unstaged changes, untracked files, and the commit graph. Useful read-only commands include:

```bash
git status --short --branch
git branch --show-current
git worktree list
git diff --stat
git diff --cached --stat
git log --oneline --decorate -10
git symbolic-ref --quiet refs/remotes/origin/HEAD
```

Find the intended PR base from an existing task PR, explicit user instruction, or the remote default branch. Do not assume `main` when the repository uses `master` or a documented release/integration branch. Check that the base ref resolves and fetch it when possible. Record a failed fetch and use a known local base only with an explicit freshness limitation.

Compare the whole branch against that base:

```text
git log <base>..HEAD --oneline
git diff <base>...HEAD
```

Classify changes as task-related, unrelated, or uncertain using the issue, session evidence, callers, and repository context. Include already staged files in this assessment. Ask a focused clarification only when uncertain ownership would change what is committed or published; continue independent reads. Never treat all dirty files or all branch commits as belonging to the task.

If task commits accidentally already exist on the local base, branch from the current task head and review those commits against the remote target. Do not reset the user's base to make it clean. If the task is already merged remotely and there are no new changes, report the existing PR rather than manufacture a new diff.

## Choose isolation without losing work

Reuse a branch/worktree only when it belongs to this task and contains no unrelated commits that would enter the PR. A branch different from `main`/`master` can still be the wrong branch. Detached HEAD also needs a named task branch before publication.

For task changes on the base branch, create a new branch at the current HEAD before staging/committing. Dirty working files stay in that checkout. Use the repository's naming rules or `codex/<issue-key>-<short-slug>` when no rule exists. Resolve name collisions by checking ownership, not by overwriting the branch.

Use a new worktree when simultaneous work, existing unrelated commits, or repository instructions make it useful. Use managed worktree tools if available. A worktree starts from a commit and does not automatically include staged, unstaged, or untracked task changes. If transferring work, preserve an explicit patch/file inventory, use non-destructive copying/applying, and compare the resulting diff and untracked files before continuing. Do not drop stashes, reset the source, or remove source files merely to simulate a move. Do not expose sensitive patches outside the workspace.

If the current branch has unrelated commits, isolate the task from the real base using selected task commits and task changes; preserve the original branch and dirty work. Do not cherry-pick an inseparable mixed commit without inspecting and isolating its task hunks. Stop dependent publication if safe separation is uncertain.

## Make each commit coherent

Atomic commits represent one independently understandable purpose. A fix and its regression tests usually belong together. Keep prerequisite/schema changes ordered before consumers when they can form valid separate changes; otherwise keep coupled changes in one buildable commit. Do not split mechanically by file, layer, or a required commit count.

Stage named files or selected hunks. Inspect `git diff --cached` before each commit, including any files the user had already staged. Preserve unrelated index contents: if mixed staging prevents a task-only commit, record the original index state and use a safe index-isolation technique supported by the environment; verify unrelated staging is restored. Never blindly `git add .`, commit the entire index, or unstage someone else's files without preserving their state.

Follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/):

```text
<type>[optional scope][!]: <concise description>

[optional explanation of why]

Refs: PROJ-123
[BREAKING CHANGE: explanation when applicable]
```

`feat` means new behavior, `fix` means a bug fix. Other types (`docs`, `test`, `refactor`, `perf`, `build`, `ci`, `chore`, `style`) follow local conventions. Do not classify a product behavior change as `style` solely because it affects UI. A Jira prefix by itself is not a Conventional Commit type; put the key in a compatible description or footer unless project rules specify a compatible alternative.

Run relevant repository-mandated checks. Do not bypass commit hooks with `--no-verify`, or declare interrupted/empty test runs passing. If a check fails, record the result and keep the workflow review-pending. Avoid empty commits, automated history rewriting, and committing secrets or unrelated artifacts.

## Reuse or create the PR

Search PRs by repository, exact head/base, and task key/links. An identical title in another repository or from a different branch is not sufficient. If several candidates overlap ambiguously, resolve ownership before updating one. Reuse a relevant open PR and preserve human-authored context. A merged PR belongs to completed history; subsequent work needs a new task branch/PR. A deliberately closed unmerged PR should not be silently reopened or replaced without resolving the closure reason.

Push the task branch normally. A push rejection is a reason to inspect remote state, not to force-push. Use a PR template when present, adapting its sections to the task; avoid leaving empty template scaffolding.

Title:

```text
PROJ-123 - Prevent duplicate appointment submissions
```

Suggested body, adjustable to the repository and task:

```markdown
## Context

[Jira task link]. Explain the reported problem, affected flow, and intended outcome.

## Changes

Describe the resulting behavior and the relevant implementation choices. Mention material boundaries, compatibility, or cross-repository dependencies when useful.

## Validation

Record commands and outcomes actually completed, the behaviors they prove, and material checks that failed or were not run. Separate local tests from CI and production evidence.

## Remaining work or constraints

Include actionable findings, accepted risks, rollout prerequisites, or an integration order only when they exist.
```

Drafts make a pending review explicit. Preserve an existing PR's human-selected state; do not convert it gratuitously. For a new PR without draft support, add review-pending text and update that text when the gate completes.

With a CLI, place multiline bodies in a temporary file and use `--body-file`; use structured arguments with MCP tools. Protect literal backticks and shell metacharacters. Attach created/reused task PRs through the environment's artifact tool when available. Do not post separate PR reviews/comments or request reviewers unless the task or a deliberately invoked skill authorizes them.

For multiple repositories, document every PR and integration dependency. Review each diff and the contract between them before the Jira gate can pass. Do not merge, deploy, or close the Jira issue as part of this workflow.
