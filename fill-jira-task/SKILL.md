---
name: fill-jira-task
description: Fill or reconcile Jira issue metadata using the available MCP, authenticated CLI, or configured API. Select an existing epic for the system area the task touches, fill an empty assignee with the verified acting user, calibrate story points on the 1/3/5/8 scale, preserve and improve task-focused descriptions, and record supplied review findings. Proactively enrich thin tasks: research gaps in the issue, linked and similar issues, the epic, and other connected sources, then ask a short grill-style interview (at most a few questions, each with a recommended answer) only for decisions that materially change scope or estimate, marking unconfirmed assumptions. Use for Jira task enrichment directly or as the Jira specialist called by finish-task. Does not perform Git operations, open PRs, review code, or transition issue status.
license: MIT
---

# Fill Jira Task

Own Jira context retrieval, task enrichment, and task-field reconciliation. Work directly from a user's request or from a caller such as `finish-task`; no repository, PR, or code review is required for a standalone metadata request. Preserve existing reports and requirements, and verify actual persisted changes.

Follow the issue/user's language. Read the available Jira connection's schema and any applicable project guidance. Load [references/jira-updates.md](references/jira-updates.md) for connection discovery, supported fields, hierarchy, identity, concurrency, and comments. Load [references/estimation-and-description.md](references/estimation-and-description.md) when assessing points or descriptions. Load [references/enrichment-and-interview.md](references/enrichment-and-interview.md) before building a Reconcile plan or when asked to enrich a task. Load [references/epicos-ns.md](references/epicos-ns.md) when choosing an epic; it is one project's area mapping, not a universal catalog. These references belong to this skill; callers should use this entrypoint rather than reproduce the field policies.

## Resolve the issue and choose the operation

Ask for the Jira key if neither the user nor the caller has supplied an unambiguous key. Reuse a supplied key and context; do not ask again as part of a handoff. Resolve an issue URL/numeric ID through the available connection when necessary. Fetch the current issue, project/type, original description, related decisions, parent/epic, assignee, points, and relevant comments/links/attachments. Never guess credentials, field IDs, tool names, or the acting Jira identity.

Use one of these conceptual operations; they are instruction modes, not CLI flags:

| Operation | Purpose | Permitted Jira writes |
| --- | --- | --- |
| **Context** | Return the issue and original requirements to a caller preparing a review, or inspect requested information | None |
| **Reconcile** | Fill/correct the supported requested metadata and record supplied pending findings | Only justified task-field changes and supplied task-related findings comments |
| **Findings only** | Record the supplied review verdict/pendings while normal finalization is blocked or incomplete | Only the matching pending-findings comment |

For a direct request to fill Jira information, use Reconcile unless the user requests inspection/proposals only. Enrichment research and interview questions are not Jira writes: they may run within Context or Reconcile, and they never widen the permitted writes in the table. Respect a request limited to particular fields. Do not manufacture a code review verdict for a standalone request; determine corrections from the issue, approved scope, supplied decisions, and trustworthy evidence. Investigating or fixing application code is outside this skill.

## Contract with finish-task or another caller

The caller loads and follows this skill in its existing task context; using a companion skill does not require a new chat, a subagent, or another user prompt.

Accept the issue key, selected operation, requested field scope, original requirements snapshot/version, approved decisions, available implementation/validation evidence, and, when this is finalization work, all task PR URLs and reviewed/current head/base SHAs, review verdict, findings, and explicit human acceptances. Missing optional evidence is not invented. A finalization caller must supply a usable gate before Reconcile.

For a `finish-task` handoff:

- **Context:** return the issue and original/approved requirements without mutating it. Preserve rich-text/source information and available version/updated metadata so the caller can detect later edits.
- **PASS** or applicable **ACCEPTED WITH PENDING FINDINGS:** Reconcile is allowed when the issue requirements and all task PR revisions still match the reviewed snapshot. Preserve accepted findings as unresolved and retain the human's specific decision.
- **BLOCKED**, **INCOMPLETE**, missing verdict, or stale head/base/requirements: do not reconcile fields. Use Findings only for supplied relevant pendings; return the rest as pending and tell the caller what review/context needs refreshing.

Re-read relevant issue content before writing. If requirements changed, or refreshed PR evidence shows a different head/base, return control to the caller for review; do not repair the verdict or reinterpret code. If current PR revision evidence cannot be obtained through the available connection/caller, report the finalization gate as unverified instead of reusing a stale verdict. Standalone operation must not be used to evade a known active finalization gate for the same task; that gate resumes after reviewed fixes or explicit acceptance of the identified findings.

## Enrich before reconciling

Do not just fill fields with whatever the issue already says. Before building the change plan, follow [references/enrichment-and-interview.md](references/enrichment-and-interview.md):

1. **Map the gaps** in context, objective, expected behavior, acceptance conditions, scope, dependencies, data/compatibility, risk, estimate inputs, and area, scaled to the task's size.
2. **Research first.** Close gaps from the issue, its comments and history, the epic and siblings, linked/similar/duplicate issues, comparable estimated issues, and other connected sources such as the repository, Sentry, or documentation. Research is read-only and bounded; report duplicates or overlaps instead of editing other issues.
3. **Ask only what remains and matters.** A gap becomes a question only when it is the user's decision, changes scope, acceptance, the estimate band, or the epic, and has no safe default. Ask in grill format: numbered, impact-ordered, each with a recommended answer worded so "yes" accepts it; at most 5 questions per task (ideally 3) in at most 2 rounds. Zero questions is a normal outcome. "Go ahead" accepts every recommendation as an assumption.
4. **Mark what is unconfirmed.** Never present an assumption as a requirement. Label unconfirmed assumptions that shape the description, and return all assumptions and open questions in the result.

Skip the questions, but still research and mark assumptions, when a caller such as `finish-task` invoked the skill, the run is headless or scheduled, or the user asked for no questions. In a batch with a live user, ask one consolidated round for the whole batch instead of one per issue. Enrichment feeds the field rules below; it does not override them, and a complete, accurate field still needs no update.

## Reconcile the supported requested fields

Build a compact current/proposed/reason/editability plan and apply only changes justified by the actual issue context, enrichment research, and the user's answers. A complete field may need no update.

- **Parent / epic:** the parent epic is the system area the task touches. Pick the existing epic for the affected area, never a generic bucket or a domain concept. Keep a fitting parent and valid subtask hierarchy. Report unresolved ambiguity. **Never create an epic** or edit a parent issue merely to categorize a subtask. List the project's open epics with JQL (`issuetype = Epic`) when a saved list may be outdated. See [references/epicos-ns.md](references/epicos-ns.md).
- **Assignee:** preserve any existing assignee. Only fill an empty field with the verified authenticated actor or explicitly identified acting user, after checking assignability. Do not infer a human from Git author, reporter, or token ownership guesses.
- **Story points:** discover the applicable numeric field for this project/type/board. Use **1 / 3 / 5 / 8**, comparing actual historical task scope, coupling, criticality, uncertainty, and validation. Keep a justified existing estimate; explain any supported correction. Do not invent field IDs, write every similarly named field, or convert points to hours.
- **Description:** leave complete, accurate text unchanged. Create or narrowly correct incomplete/inaccurate text using flexible, task-focused context, goals, behavior, and acceptance conditions. A description missing material context, scope, or acceptance conditions that enrichment established is incomplete; add them with their sources and keep unconfirmed assumptions labeled. Preserve reports, relevant details, decisions, links, and original requirements. Deep implementation/test logs belong in supplied PRs or existing technical records. Missing implementation cannot be hidden by removing a requirement.
- **Pending findings:** add/update a task-related comment only when findings or material finalization limitations are supplied. Follow the reference's identity, visibility, retry, and acceptance rules; do not invent findings or claim that accepted defects were fixed.

Handle stale/conflicting descriptions as requirements/context decisions, not an opportunity to overwrite another user. Send only intended changed fields and verify them with a fresh read. Continue independent supported updates when one field is absent/non-editable or fails after an eligible gate; report partial completion and unknown write results accurately.

## Return the result

Return the issue key/link, operation, and a concise per-field result: changed with previous/new values and relevant rationale; retained because already appropriate; unavailable/non-editable; ambiguous; or failed/pending. Include the enrichment sources used, the questions asked and their answers, assumptions still unconfirmed, open questions (with recommendations when the questions were skipped), and estimate confidence. Include the findings comment link/ID and review identity when written, any preserved acceptance, and unresolved actions. Return enough refreshed issue/revision context for the caller to detect invalidation; never report review, merge, deployment, or Jira resolution from metadata updates.

Invocation authorizes the described updates and supplied task-related findings comments within the user's field scope and selected operation. Honor inherited authorization from a valid caller without another approval round. Interview answers settle task content; they do not expand this authorization. This skill does not authorize Git commits, PR publication, new issues/epics/fields, links or edits on related issues, issue-type changes, project-configuration changes, status transitions, or unrelated messages, including posting interview questions as Jira comments.
