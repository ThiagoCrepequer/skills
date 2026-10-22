# Fill Jira Task

Fill and reconcile Jira task information through the available MCP, authenticated CLI, or configured API connection. This skill owns epic selection by system area, empty-assignee resolution, story-point calibration, task-focused descriptions, and supplied pending-findings comments. The parent epic is the area the task touches, never a generic bucket or a domain concept. The NS list in the references is an example; list the project's open epics with JQL (`issuetype = Epic`) when that list may be outdated.

```bash
npx skills add ThiagoCrepequer/skills --skill fill-jira-task
```

```text
$fill-jira-task Fill the supported metadata for PROJ-123, preserving its reports and requirements.
```

Before filling, it maps what the task is missing, researches the issue's comments, epic, linked and similar issues, and other connected sources, and asks only a short grill-style round (each question with a recommended answer) for decisions it cannot look up. Unconfirmed assumptions are labeled. Callers such as finish-task and headless runs get no questions; the open questions are returned instead.

It can run on its own without Git or a PR. It discovers the actual Jira fields and authenticated actor, compares points using the 1/3/5/8 scale and project history, and changes only incomplete or inappropriate metadata. Complete descriptions, fitting parents, and existing assignees are preserved.

[finish-task](../finish-task/SKILL.md) uses this specialist for read-only issue context, eligible field reconciliation, and findings-only comments when its review is blocked. The specialist respects the caller's review verdict, original requirements, reviewed revisions, and human acceptance; it does not perform the code review itself.

To install the composed workflow:

```bash
npx skills add ThiagoCrepequer/skills --skill finish-task fill-jira-task
```

The entrypoint is [SKILL.md](SKILL.md). Supporting references cover [Jira operations and comments](references/jira-updates.md), [estimation and descriptions](references/estimation-and-description.md), [enrichment and the short interview](references/enrichment-and-interview.md), and an [example area-epic mapping](references/epicos-ns.md).

Licensed under [MIT](LICENSE).
