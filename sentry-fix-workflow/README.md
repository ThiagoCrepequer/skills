# Sentry Fix Workflow

Orchestrate one Sentry defect: read the issue, fix a simple cause at its origin, and hand off through Jira and a pull request. Companion skills own tests, Jira fields, and Git publication. This skill does not repeat them.

Install this skill:

```bash
npx skills add https://github.com/ThiagoCrepequer/skills --skill sentry-fix-workflow
```

Install the companions it loads:

```bash
npx skills add https://github.com/mattpocock/skills --skill tdd
npx skills add https://github.com/ThiagoCrepequer/skills --skill java-test-quality
npx skills add https://github.com/ThiagoCrepequer/skills --skill react-test-quality
npx skills add https://github.com/ThiagoCrepequer/skills --skill fill-jira-task
npx skills add https://github.com/ThiagoCrepequer/skills --skill finish-task
```

```text
$sentry-fix-workflow Investigate this Sentry issue and, when the fix is simple, take it through a failing test, Jira, and a pull request.
```

The workflow proceeds on its own only for a simple, reproducible defect. Medium and hard issues stop after the analysis. Expected 4xx responses and noisy detectors become filter suggestions, not code changes.

The entrypoint is [SKILL.md](SKILL.md). Test design stays in `tdd`, [java-test-quality](../java-test-quality/SKILL.md), and [react-test-quality](../react-test-quality/SKILL.md). Jira fields stay in [fill-jira-task](../fill-jira-task/SKILL.md). Branch, commits, pull request, and the review gate stay in [finish-task](../finish-task/SKILL.md).

The task branch must not start with `fix/`. Merging, deployment, and moving Jira to Done are out of scope. After a passing handoff, the workflow moves the issue to the project's ready-for-release status.

Licensed under [MIT](LICENSE).
