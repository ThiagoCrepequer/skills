# Estimation and task-focused descriptions

## Story-point scale: 1 / 3 / 5 / 8

Use the requested scale as a relative estimate of delivering the agreed scope, including meaningful verification. Compare scope, coupling, risk, uncertainty, and validation demands. Do not convert points to hours, count changed lines/files, or treat title adjectives as evidence. Historical team estimates are calibration examples, not universal labels or grounds for mechanically repointing every task.

| Points | Typical scope | Conditions that distinguish it |
| --- | --- | --- |
| **1 — Very easy** | Small style correction, simple validation, short localized correction or bounded option change | Known behavior, one narrow surface, little coupling, low regression risk, straightforward verification |
| **3 — Low to medium** | A focused defect or improvement with some logic in a limited, less critical part of the system | Several branches/states or one contained workflow, understood dependencies, manageable regression coverage |
| **5 — Medium to high** | Medium CRUD/feature, a moderately complex new capability, or coordinated changes across multiple parts | Meaningful business rules, contracts/data relationships, more substantial validation, bounded but material dependencies |
| **8 — High** | Large or tightly coupled feature/change, critical business/data flow, integration or migration with broad effects | Significant cross-system coordination, critical invariants, uncertain behavior, compatibility/data-transition concerns, or substantial integration verification |

Assess the task as a whole, not just the last fix or the diff currently visible in one repository. A localized critical defect can warrant more points than its line count suggests, but criticality alone is not an automatic 8. Prefer the band best justified by the combined demands. If the task clearly exceeds the largest band, flag decomposition for planning rather than inventing 13 or creating tickets automatically.

A credible existing team estimate should normally remain. Reconcile a missing or clearly outdated estimate when evidence supports it, explaining material scope differences. Finishing more slowly than expected is not, by itself, a reason to rewrite the estimate. Do not inflate points to reward effort or lower points because difficult work has already been implemented.

## Calibrate from the actual project

Discover the applicable points field before searching. Use a small, bounded sample of recent comparable tasks in the issue's project/domain, preferably completed tasks with clear descriptions. Search each requested value using the field's actual JQL clause/ID, for example `project = <project> AND cf[<discovered-id>] = 3 ORDER BY updated DESC`. Paginate when needed to find useful comparables; do not scan the entire organization by default.

Read more than titles: outcome, scope, acceptance conditions, status, and relevant decisions. Start with a few candidates per relevant band and extend only if ambiguous. Missing descriptions, mixed unrelated work, inconsistent point fields, and purely technical delivery diaries reduce confidence. Record the issue keys and decisive similarities/differences in the private execution context; do not export internal Jira content into a public skill/repository.

If meaningful history is unavailable, use the requested scale and disclose that the estimate is rubric-based. If both the field and estimate remain materially ambiguous, leave the value unchanged and report the decision needed. Do not silently introduce a new scoring scale from historical outliers.

### Lessons from the authoring-time sample

A small sample of recent work supported these general distinctions:

- Small display/validation corrections and bounded option additions can fit **1** when their impact is genuinely narrow.
- A contained recurring-workflow rule or focused completion error can fit **3** even when several states need regression coverage.
- Changes coordinating a CRUD rule with integration behavior, or preventing duplicate imports while preserving validation/data representation, can fit **5**.
- A description that sounds like “small improvements” can still fit **8** when the actual scope spans financial behavior, exports, historical data changes, and multiple applications. Broad import/integration workflows or coordinated domain features also fit this band's demands.

These are anonymized patterns, not copied tickets or fixed calibration anchors. Revalidate against the active project's actual historical usage on each invocation.

[Atlassian's estimation guidance](https://www.atlassian.com/agile/project-management/estimation) treats story points as relative effort informed by complexity, risk, uncertainty, and amount of work. The four-value scale above is the user's chosen convention.

## Describe the task, not the implementation diary

A description should tell a PM/PO, support analyst, tester, and developer what problem matters, who experiences it, what outcome is expected, and what defines success. Do not force every task into a user-story sentence or a fixed set of headings. [Atlassian's user-story guidance](https://www.atlassian.com/agile/project-management/user-stories) emphasizes user value, clarification, and acceptance; bugs, technical debt, and operational tasks can express the same essentials in other forms.

Use as much structure as the task needs:

- **Small correction:** a concise problem statement, desired behavior, and one or two relevant acceptance conditions.
- **Bug:** preserve the original report and reproduction context; describe actual versus expected behavior, affected users/scope, and regression boundaries.
- **Feature/customization:** explain context and value, affected workflows, intended behavior, relevant business rules, and observable acceptance conditions.
- **Performance/technical debt:** express the service/operational problem, desired improvement, behavior/invariants to preserve, and justified measurable acceptance when supplied. Put SQL/classes/index mechanics, execution logs, and test commands in the PR.
- **Data/operational task:** preserve affected scope, period, authorized constraints, intended outcome, reconciliation/verification conditions, and relevant reports. Do not claim production execution from local scripts/tests.

Possible sections are **Context / report**, **Objective**, **Expected behavior and scope**, **Acceptance conditions**, and **Relevant constraints / decisions**. Choose, combine, or omit them; never leave empty boilerplate. Preserve the issue's language and useful existing organization. Observable technical requirements explicitly present in the issue remain requirements; simplifying language must not erase them.

## Decide whether to edit

| Existing description | Action |
| --- | --- |
| Absent | Create a concise, evidence-backed description from the task summary, approved decisions, and trustworthy user/operational outcomes when available. Do not invent reports, users, business rules, or product acceptance. |
| Incomplete/inaccurate | Make the narrow additions/corrections required to capture the approved scope and actual resulting behavior while preserving original evidence. |
| Complete/accurate | Leave it unchanged, including its formatting. Do not rewrite merely to fit the suggested headings. |

Before updating, compare the original contract with approved task decisions and the evidence available for the requested operation. In a finalization handoff, also use the caller's reviewed implementation; this specialist consumes that review rather than running one. Standalone filling does not require application code, a PR, or review evidence. If a proposed correction lacks support, retain the original statement and request only the facts needed for that correction. A missing implemented requirement identified by a finalization review is a finding, not an outdated description. Only reconcile it after an explicit scope decision/acceptance, retaining the prior promise and the decision where important. Code behavior alone cannot prove that a statement in Jira is wrong.

Important information includes customer/user reports, reproduction steps, dates/periods, affected populations, examples and counterexamples, business constraints, historical decisions, attachments, links, and unresolved items. Preserve these in the existing description or a clearly labeled historical/decision section within the ticket when restructuring. Do not remove reported symptoms just because they are fixed. Clearly distinguish the approved target behavior from superseded notes when both remain useful.

Deep implementation descriptions already present should not be silently deleted for stylistic conformity. Preserve important detail, or move it to a clearly referenced technical note/PR only when that retention is concrete and supported; the workflow does not authorize unrelated comments or public disclosure of sensitive material. Avoid making a complete task longer by appending test logs, commits, and code-review output.

Re-read immediately before writing. If another user edited the description, merge only the necessary additions into their current text; resolve a conflicting requirement instead of overwriting it. Use the connector's supported rich-text/Markdown format and check the persisted result for retained information, links, and formatting.

## Examples of flexible descriptions

### Small bug

> Users can submit the appointment form twice while the first request is still processing, creating duplicate appointments. A submission must create at most one appointment, show a pending state until the request finishes, and allow another attempt after a failure. Preserve the original report and reproduction steps below.

### Feature with several business rules

> Some organizations operate distinct units that share an identification number. They need to keep those units separate while registering the shared identifier. Creation and editing must accept this situation, preserve the other validation rules, and keep each unit's relationships independent. Integrations must consider every corresponding unit, avoid duplicate links on reprocessing, and preserve organization isolation.

Then list only material acceptance conditions and constraints supplied by the task. Details such as repository method names, schema scripts, or exact automated test counts belong in the PR.
