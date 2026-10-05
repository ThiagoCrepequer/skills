# Finish Task

Finalize an implemented Jira task with atomic Conventional Commits, a linked PR, review against the original task and project rules, and supported Jira metadata updates.

```bash
npx skills add ThiagoCrepequer/skills --skill finish-task
```

```text
$finish-task Finalize this work for PROJ-123.
```

If no Jira key is supplied, the skill asks for it first. It reuses suitable task branches and PRs, preserves unrelated work, and discovers the available Jira MCP/CLI/API connection and actual project fields.

A failed or incomplete review stops normal finalization. The requested pending-findings Jira comment is the sole write exception at that gate. Resume after reviewed fixes or explicit human acceptance of the identified outstanding risks; acceptance remains visible rather than being reported as a passing review.

After the gate, the skill chooses an existing fitting epic, fills an empty assignee using the verified acting user, calibrates points using the 1/3/5/8 scale and project history, and updates only incomplete or inaccurate task descriptions. It preserves reports, important decisions, and original requirements.

The skill prepares the handoff. Merging, deployment, new epics, and Jira status transitions require separate instructions.

The entrypoint is [SKILL.md](SKILL.md). Its supporting references cover [Git and PRs](references/git-and-pr.md), [review gates](references/review-gate.md), [Jira updates](references/jira-updates.md), and [estimation and descriptions](references/estimation-and-description.md).

Licensed under [MIT](LICENSE).
