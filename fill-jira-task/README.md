# Fill Jira Task

Fill and reconcile Jira task information through the available MCP, authenticated CLI, or configured API connection. This skill owns epic selection, empty-assignee resolution, story-point calibration, task-focused descriptions, and supplied pending-findings comments.

```bash
npx skills add ThiagoCrepequer/skills --skill fill-jira-task
```

```text
$fill-jira-task Fill the supported metadata for PROJ-123, preserving its reports and requirements.
```

It can run on its own without Git or a PR. It discovers the actual Jira fields and authenticated actor, compares points using the 1/3/5/8 scale and project history, and changes only incomplete or inappropriate metadata. Complete descriptions, fitting parents, and existing assignees are preserved.

[finish-task](../finish-task/SKILL.md) uses this specialist for read-only issue context, eligible field reconciliation, and findings-only comments when its review is blocked. The specialist respects the caller's review verdict, original requirements, reviewed revisions, and human acceptance; it does not perform the code review itself.

To install the composed workflow:

```bash
npx skills add ThiagoCrepequer/skills --skill finish-task fill-jira-task
```

The entrypoint is [SKILL.md](SKILL.md). Supporting references cover [Jira operations and comments](references/jira-updates.md) and [estimation and descriptions](references/estimation-and-description.md).

Licensed under [MIT](LICENSE).
