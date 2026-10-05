# Skills

Reusable agent skills for evidence-led software engineering.

## Available skills

- [query-performance-workflow](query-performance-workflow/SKILL.md): investigate expensive application queries, preserve behavior with regression tests, capture ORM SQL, and validate optimization with representative execution plans.
- [react-test-quality](react-test-quality/SKILL.md): design, write, strengthen, and review trustworthy React and JavaScript/TypeScript tests.
- [java-test-quality](java-test-quality/SKILL.md): design, write, strengthen, and review trustworthy Java and Quarkus tests.
- [finish-task](finish-task/SKILL.md): finalize task commits, PRs, and spec/standards review, delegating Jira operations to `fill-jira-task`.
- [fill-jira-task](fill-jira-task/SKILL.md): fill Jira epic, assignee, story points, and task descriptions, preserve reports/requirements, and record supplied review findings.

Each skill keeps its instructions and supporting references in its own directory. Add the directory to your agent's supported skill location and invoke it by name, for example `$query-performance-workflow` in Codex. Supporting references are loaded only when relevant. `finish-task` composes with the required `fill-jira-task` companion; install both for the complete finalization workflow. `fill-jira-task` can also run independently.

## Installation

Select skills from this repository:

```bash
npx skills add ThiagoCrepequer/skills
```

Install a specific skill:

```bash
npx skills add ThiagoCrepequer/skills --skill react-test-quality
npx skills add ThiagoCrepequer/skills --skill java-test-quality
npx skills add ThiagoCrepequer/skills --skill fill-jira-task
```

Install the complete finalization workflow:

```bash
npx skills add ThiagoCrepequer/skills --skill finish-task fill-jira-task
```

See the individual skill READMEs for usage examples. The migrated test-quality skills include their original MIT licenses in their respective directories.

## Migration provenance

The test-quality skills were migrated from their standalone repositories. Their original commits are retained in this repository's Git history.

| Skill | Source repository | Imported source commit |
| --- | --- | --- |
| React Test Quality | [ThiagoCrepequer/react-test-quality](https://github.com/ThiagoCrepequer/react-test-quality) | `ccf44e7` |
| Java Test Quality | [ThiagoCrepequer/java-test-quality](https://github.com/ThiagoCrepequer/java-test-quality) | `9d79341` |
