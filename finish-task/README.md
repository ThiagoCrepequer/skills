# Finish Task

Finalize an implemented Jira task with atomic Conventional Commits, a linked PR, and review against the original task and project rules. The required [fill-jira-task](../fill-jira-task/SKILL.md) companion owns Jira context retrieval and metadata reconciliation.

Install both skills:

```bash
npx skills add ThiagoCrepequer/skills --skill finish-task fill-jira-task
```

```text
$finish-task Finalize this work for PROJ-123.
```

If no Jira key is supplied, the skill asks for it first. It obtains read-only issue context from the companion, reuses suitable task branches and PRs, preserves unrelated work, and performs the task/spec/project review. Resolve the required companion before dependent work; a missing installation is reported instead of silently duplicating its policies.

A failed or incomplete review stops normal finalization. The caller uses the specialist's Findings only operation for the requested pending-findings comment. Resume after reviewed fixes or explicit human acceptance of the identified outstanding risks; acceptance remains visible rather than being reported as a passing review.

After the gate, the caller passes the original requirements, current reviewed PR revisions, verdict, evidence, findings, and human acceptance to fill-jira-task's Reconcile operation. That companion handles existing epic, empty assignee, points, descriptions, and verification, and returns a per-field result for the final summary. The handoff stays in the same task and does not require another user prompt.

The skill prepares the handoff. Merging, deployment, new epics, and Jira status transitions require separate instructions.

The entrypoint is [SKILL.md](SKILL.md). Its own references cover [Git and PRs](references/git-and-pr.md) and [review gates](references/review-gate.md). Jira policies live in the [companion entrypoint](../fill-jira-task/SKILL.md) and its references.

Licensed under [MIT](LICENSE).
