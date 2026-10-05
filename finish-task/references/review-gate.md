# Review against the task and project

## Pin what is being reviewed

Record the repository, target branch/base SHA, merge base, and actual PR head SHA. Refresh the issue's original requirements and authoritative scope decisions; retain the pre-reconciliation baseline. Read comments and relevant attachments when they clarify reports or acceptance conditions. Do not let the current implementation define its own specification.

Review the complete task diff against the target base, including existing commits. Inspect surrounding callers, permissions, data flow, API contracts, migrations, and test setup only as relevant to the change. For a multi-repository task, review all task PRs and their integration order/contracts.

Use three complementary lenses:

| Lens | Questions | Evidence |
| --- | --- | --- |
| Task fidelity | Are promised outcomes, exceptions, and accepted decisions implemented? Are requirements missing or contradicted? | Original issue/decisions mapped to observable behavior and the diff |
| Correctness | Could the change produce a regression, wrong result, race, data leak, partial write, or incompatible contract? Do tests detect plausible failures? | Relevant code paths, callers, regression tests, required checks |
| Project practices | Does it follow applicable architecture, API, state, error-handling, style, security, and testing rules? | Specific rule in `AGENTS.md`, contribution/architecture docs, or established owning-source convention |

A compact requirement-to-evidence matrix helps with a complex task. A small fix can be reviewed in prose. Do not require every generic risk or every test layer for all tasks.

## Use available skills selectively

Inspect the skill catalog and read only relevant guidance. Examples, when installed:

- `code-review` for spec and standards review;
- `react-test-quality` or `java-test-quality` for the reliability of changed tests;
- a repository-specific architecture, framework, database, or verification skill for the affected area;
- a strict maintainability audit only when requested or the change warrants that scope.

Follow a used skill's applicable prerequisites and available delegation tools. Independent review can use subagents when authorized; provide the original spec and pinned diff rather than the implementer's preferred conclusion. If delegation is unavailable, perform the same review directly and disclose that limit. This workflow supplies its own review baseline and does not depend on another skill, a missing setup command, or an issue-tracker configuration file.

Do not install dependencies, add CI gates, or demand mutation/browser/full-suite runs solely because a reference lists them. Run required project checks and the lowest layer faithful to the actual risk. Separate pre-existing failures from new failures using evidence; an unexplained failure cannot be silently labeled baseline.

## Findings and verdict

An actionable finding needs a concrete trigger, observable consequence, supporting file/line or requirement/rule, and the needed correction or decision. Label severity and distinguish substantiated defects from uncertain questions and optional suggestions. Cosmetic preferences and hypothetical risks without evidence do not automatically block.

Use these verdicts:

- **PASS:** the task is coherent with the original/approved spec and project standards, relevant required validation completed, and no actionable finding remains unresolved.
- **BLOCKED:** a material requirement gap, correctness defect, documented-standard violation, or relevant failing validation remains.
- **INCOMPLETE:** unavailable Jira/spec/diff, required checks still pending, or another missing essential observation prevents an honest verdict.
- **ACCEPTED WITH PENDING FINDINGS:** the human explicitly accepted identified findings/limitations for this task and reviewed revision. Preserve the unresolved items and the acceptance; do not rename the verdict PASS.

A reported implementation/spec mismatch stays a finding. Rewording the issue to match code cannot clear it. Product scope changes must be backed by actual human decisions. A bug report with no formal acceptance section can still provide a concrete contract; a completely empty/unclear issue may need a targeted scope clarification before review can pass.

## Stop and resume

When BLOCKED or INCOMPLETE, halt normal Jira metadata/description updates and readiness promotion. Summarize the findings and all pending steps. Leave the existing commits and PR available for review. Do not repair the implementation automatically in a finalization run after finding a defect.

The pending-findings Jira comment is an explicit exception to this stop: record the review and what prevents continuation, if Jira access permits. Delegate the supplied review context to [fill-jira-task](../../fill-jira-task/SKILL.md) in **Findings only** mode; that specialist owns comment identity and visibility. A failure to post the comment leaves that operation pending; it never converts the verdict to PASS.

Continue only after the human fixes/authorizes fixes and the affected changes are reviewed, or explicitly accepts the specified unresolved findings. A generic acknowledgement, silence, or acceptance from another agent is insufficient. An existing acceptance applies only to the identified findings, scope, and reviewed revision; do not extend it to newly introduced problems. Be precise about failed, waived, pending, and unrun checks.

Before Jira reconciliation and PR readiness, compare current PR head/base and issue requirements with the reviewed snapshot. If they changed, review the affected scope and refresh validation rather than reuse a stale verdict. After a blocked run, reuse the existing PR and completed commits; commit only new relevant adjustments and update the review record.
