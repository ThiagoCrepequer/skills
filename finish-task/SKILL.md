---
name: finish-task
description: Finalize an implemented Jira task by isolating its Git changes, creating atomic Conventional Commits, opening or updating its pull request, reviewing against the original task and project standards, and reconciling Jira metadata. Use when asked to finish a task, prepare its handoff, or close out implementation work with Jira and a PR. Does not merge, deploy, or transition Jira status by default.
license: MIT
---

# Finish Task

Turn implemented work into a reviewable handoff, with an honest review verdict and current Jira metadata. Follow the user's language for questions, Jira text, and summaries; follow repository conventions for commits and PRs.

## Start with the task

Ask for the Jira issue key before changing Git or remote state, unless the user already supplied an unambiguous key for this execution. Do not silently select a ticket from the branch name or conversation history. Normalize the supplied key and resolve numeric IDs or issue URLs through Jira when needed.

Read applicable `AGENTS.md`, contribution guidance, PR templates, test commands, and project architecture/standards. Discover the available Jira MCP, authenticated CLI, or configured API client; prefer an existing connection. Never invent tool names, field IDs, CLI flags, credentials, or an authenticated identity.

Fetch the issue's summary, original description, issue type, project, parent/epic, assignee, points, and relevant comments, attachments, and linked decisions. Keep the original requirements as the review baseline, before editing the description. If the ticket cannot be retrieved, request the missing access/context and stop dependent publication/reconciliation; do not claim spec review passed.

Identify every repository needed to deliver this task. Record a compact execution ledger: issue key, repository and base, branch/worktree, initial staged/unstaged/untracked changes, task commits, PR URL, reviewed head/base SHAs, validation, review findings/acceptances, and Jira fields actually changed. This supports safe retries rather than duplicating work.

## 1. Isolate and commit relevant work

Read [references/git-and-pr.md](references/git-and-pr.md).

Verify that each repository uses a task-specific branch separate from `main`, `master`, and its actual target/default branch. An unrelated feature branch is not sufficient isolation. Reuse a suitable task branch/worktree; otherwise create a branch using repository conventions, such as `codex/PROJ-123-short-description`. A new branch from the current checkout preserves dirty files and is usually sufficient. If a worktree is needed, transfer only identified task changes and verify them; worktree creation does not copy uncommitted work.

Inspect staged, unstaged, untracked, and already committed changes against the intended base. Preserve unrelated work. Partition relevant changes into coherent, buildable commits by purpose, including dependent tests and documentation; atomic does not mean one commit per file. Inspect each staged diff and use Conventional Commits 1.0.0:

```text
fix(scheduling): prevent duplicate submissions

Refs: PROJ-123
```

Use `feat` for new behavior and `fix` for bug fixes; select other types by repository convention. Mark actual breaking changes with `!` or a `BREAKING CHANGE:` footer. Do not create empty commits, amend or rewrite existing history without authorization, discard work, or bypass failing hooks/checks. Run proportionate required validation and report failures exactly.

## 2. Open or update the PR

Find an existing PR for the task's actual repository/head/base, including task links; reuse it instead of opening duplicates. If an earlier PR is already merged, use a fresh task branch for follow-up changes. Publish the task branch with a normal push, not a direct push to the base or a force push. Never push unrelated commits merely because the branch is separate.

Use the title `<JIRA-KEY> - <task summary>`, for example `PROJ-123 - Prevent duplicate appointment submissions`. Derive the summary from Jira and the approved task scope. Use repository templates and the flexible PR structure in [references/git-and-pr.md](references/git-and-pr.md): context, resulting behavior/implementation, validation, and relevant limits or dependencies.

Create a draft when supported while review/validation is pending. If drafts are unavailable, state that review is pending in the body. Preserve a reused PR's human-authored context and review state. Attach every created/reused task PR to the current chat when the environment supports attachment. For tasks spanning repositories, prepare each PR and record integration order.

## 3. Review and enforce the gate

Read [references/review-gate.md](references/review-gate.md). Fetch the original issue/approved decisions again if needed and pin the actual PR head and target base. Review the complete task diff, not just the most recent uncommitted changes, for:

- fidelity to original requirements and accepted scope changes;
- implementation correctness, regressions, and meaningful test evidence;
- conformity to documented project architecture and practices.

Inspect available review/test-quality skills and use relevant guidance when available. The review remains self-contained when those skills are absent. Do not install tooling or require a particular third-party skill to finish the workflow.

Use a precise verdict: **PASS**, **BLOCKED** (actionable findings), **INCOMPLETE** (insufficient access/evidence), or **ACCEPTED WITH PENDING FINDINGS** (the human explicitly accepted identified risks). Missing requirements must not be made to disappear by rewriting Jira. New commits, a changed base, or changed requirements invalidate the previous verdict and require review of the affected scope.

If review is blocked or incomplete, stop the normal finalization steps. Keep the PR review-pending and report what remains. **The only Jira write allowed at this gate is the requested pending-findings comment**, described in [references/jira-updates.md](references/jira-updates.md); do not update parent, assignee, points, or description. Do not automatically fix findings or continue after a timeout. Resume only after fixes are reviewed or the human explicitly accepts the specified outstanding findings; acceptance is not a passing test or review.

## 4. Reconcile Jira after the gate

Read [references/jira-updates.md](references/jira-updates.md) and [references/estimation-and-description.md](references/estimation-and-description.md).

With PASS or a recorded, applicable human acceptance, prepare and apply only supported, relevant field changes:

- **Parent/epic:** inspect existing epics and choose a defensible match. Never create an epic. Preserve a fitting parent and valid subtask hierarchy; report ambiguity instead of inventing a category.
- **Assignee:** only when empty, assign the authenticated Jira actor (or the explicitly identified acting user). Verify identity and assignability; do not infer them from Git author, reporter, or display name.
- **Story points:** discover the applicable field and compare similar historical tasks. Use the user's scale **1 / 3 / 5 / 8**, considering scope, coupling, risk, uncertainty, and verification. Preserve a justified estimate; reconcile a missing/outdated one with a short rationale.
- **Description:** leave a complete, accurate description unchanged. Otherwise create or narrowly correct a task-focused account of context, intended outcome, behavior, and relevant acceptance conditions. Preserve reports, decisions, important details, and links. Keep deep implementation/test logs in the PR. Never erase an unimplemented promise to match the code.
- **Pending findings:** add or update a concise Jira comment for unresolved findings, including human acceptance when applicable. Avoid duplicate comments for the same review.

Re-read edited fields immediately before writing to detect concurrent changes, then verify every mutation with a fresh read. Continue independent supported updates when one field is unavailable and report partial completion. Before marking a newly created draft ready, confirm its current head/base still match the accepted review and required checks; never promote a blocked or incomplete review.

## 5. Report the handoff

Summarize the Jira key, branch/worktree and atomic commits, PR links, review verdict and exact validated scope, Jira fields changed/retained/unavailable with relevant rationale, and any pending findings or next actions. Distinguish completed actions from proposed changes and failed operations.

Invocation authorizes the described task branch/commits, PR publication, Jira metadata reconciliation, and requested pending-findings comments within the task's scope. Honor already supplied authorization without extra approval rounds. Merging PRs, deployment, deleting branches, creating epics/issues, and moving Jira to Done require a separate user instruction. A published PR is a handoff, not evidence of deployment or Jira resolution.
