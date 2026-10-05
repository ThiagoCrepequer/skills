# Jira connection, field reconciliation, and pending findings

## Discover the actual connection

Prefer the available Jira MCP. Otherwise inspect the authenticated Jira CLI's help/schema or use a configured API client. Do not hardcode an organization, project, Jira URL, account, custom-field ID, or credentials. Use the issue's project and type as scope. A tool's field-write success response is not proof that Jira stored the value.

Examples of capabilities to discover, rather than mandatory tool names:

| Need | MCP/CLI capability | Jira Cloud REST concept |
| --- | --- | --- |
| Read task and decisions | Get issue, comments, attachments/links | Issue retrieval and relevant expansions |
| Search epics/history | JQL search with selected fields and pagination | Issue search |
| Discover fields and editability | Field search, issue/project metadata | Fields, field contexts, issue edit metadata |
| Resolve acting user | Current authenticated user / account info | `GET /rest/api/3/myself` |
| Confirm assignment | Search assignable users, assign issue | Assignable-user lookup and issue assignment |
| Write only changed fields | Update issue with supported schema | Issue field update |
| Record review findings | Add/read/edit comment | Issue comments |

Jira Cloud v3 commonly uses Atlassian Document Format for descriptions/comments; MCP and CLI wrappers may accept Markdown and perform conversion. Inspect the selected client's schema. Server/Data Center can use different identities, fields, and markup. Preserve existing rich text, links, attachment references, and useful formatting; do not flatten a description blindly.

Official references: [authenticated Jira user](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-myself/), [Jira issue operations](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/), and [Atlassian Document Format](https://developer.atlassian.com/cloud/jira/platform/apis/document/structure/). Consult current documentation only for the interface needed by this run.

## Prepare a minimal change plan

For each requested property, record current value, proposed value, reason, and whether it is supported/editable. A field being globally listed does not prove that it applies to this issue type/project or is writable on its screen. Avoid creating replacement custom fields or changing project configuration.

Only reconcile metadata after PASS or applicable, explicit human acceptance. On a blocked/incomplete review, the pending-findings comment below is the sole allowed Jira write.

Before each write, re-read the fields it could overwrite. If another user changed them, preserve that newer input and recompute the proposal; ask for resolution only when the conflict is material. Send only the intended changed fields, never a stale copy of the whole issue. After writing, read them again and compare actual persisted values.

## Parent / existing epic

Fetch the existing parent/epic and its type, then search existing epics primarily in the issue's project. A typical JQL query is `project = <actual-project> AND issuetype = Epic`, adapted to the actual issue-type name/ID and pagination. Read candidates' summaries, descriptions, status, and relevant related tasks; a keyword match alone is insufficient.

Choose by business outcome, domain/workflow, intended audience, and included scope. Preserve an existing fitting epic. Change a clearly inappropriate association only when a better existing candidate is defensible and the applicable hierarchy/field is editable. Prefer active fitting epics over closed ones unless the workflow deliberately uses a closed epic. If no epic fits or candidates are materially ambiguous, leave the field unchanged and report the candidates/reason instead of inventing a category. **Never create an epic.**

Cloud's `parent`, legacy Epic Link, and team-managed/company-managed hierarchies differ. Discover the actual relation and payload shape. For a subtask, the immediate parent is normally its task/story: preserve that hierarchy and inspect the ancestor's epic. Do not replace a subtask's parent with an epic or silently recategorize its parent issue. If doing so would be necessary, report the separate decision needed.

## Assignee only if empty

Keep any existing assignee, including one different from the Git author or current actor. Re-read immediately before assignment so a newly assigned user is not overwritten.

Identify the authenticated Jira actor through the chosen connection, ideally its current-user operation. Use the user's verified acting identity if explicitly supplied. Jira Cloud typically needs `accountId`; Server/DC may need username/key. Do not infer identity from ticket reporter, Git config, token filename, email fragments, or display name alone. The token can belong to a service account: do not silently replace it with a guessed human.

Confirm that the resolved actor is active and assignable to this issue. Use a dedicated assignment capability when available, because some generic update wrappers silently ignore assignee changes. If current identity or assignment rights cannot be established, keep the field unchanged and report that field as pending; continue other independent, supported updates.

## Story points and description

Read [estimation-and-description.md](estimation-and-description.md). Detect the actual numeric field by name, schema, project/type context, edit metadata, and usage on comparable issues. Several fields named Story Points / Story point estimate may coexist. Do not write both or pick the first global match; use the board/project configuration or actual comparable usage, and resolve material ambiguity before writing.

Do not clear a justified existing estimate. Record the previous value and rationale when reconciling an outdated value. Write a numeric value in the supported schema, then verify it persisted. If the field is absent/non-editable, report that result rather than creating a field or silently using a different one.

Classify the description as absent, incomplete/inaccurate, or complete/accurate. Only the first two need a write. Preserve important reports, decisions, contextual identifiers, examples, reproduction details, and links within the original issue's access boundary. Sensitive credentials or access-bearing URLs must not be copied to public PRs, repo documentation, or summaries. Do not remove historical evidence merely because it is inconvenient; retain or clearly contextualize obsolete details and resolve any proposed removal of sensitive original material separately.

## Pending-findings comment: the review-gate exception

The user requested a Jira comment whenever findings remain pending. This is authorized even when BLOCKED/INCOMPLETE prevents other finalization writes. Read recent comments first; reuse/update the comment created by this workflow for the same PR/review when the connection permits. Preserve other users' comments. After a timeout or unknown result, re-read before retrying so a successful write is not duplicated.

Use a stable identity such as task key + task PR URL(s) + reviewed head SHA(s), with a human-readable heading `Finalization review`. Include only actionable, task-related information:

```text
Finalization review — PROJ-123
PR: <task PR URL>, reviewed revision: <head SHA>
Verdict: BLOCKED / INCOMPLETE / ACCEPTED WITH PENDING FINDINGS

Pending:
- Finding, concrete user/technical consequence, relevant requirement/file,
  and the correction or decision needed.

Next steps:
- Review the adjustments, or obtain explicit acceptance of the identified risks.
- Parent, assignee, points, and description reconciliation remain pending.
```

Adapt the next steps to what is actually pending. For accepted findings, state who accepted which items in the current trusted session and keep their unresolved status visible; do not claim they were fixed. Relevant technical details belong in this review comment or the PR, not in the product description.

Respect the issue's comment visibility and the connector's service-desk internal/public semantics. Do not broaden visibility or mention additional users to create notifications. An invocation of this skill authorizes this requested Jira comment, not unrelated messaging or PR comments.

## Partial completion and retries

Independent supported fields can still be reconciled if another field fails after the review gate. Record each success/failure, re-read results, and do not claim all Jira updates completed. Stop retrying the same permission/schema failure after inspecting its cause; choose a supported route or report the precise pending operation. Do not roll back another user's edits or automatically undo successful independent changes.

Re-runs discover existing commits, PRs, unchanged metadata, and matching comments first. If the reviewed revision/spec changed, return to the review gate before applying new metadata. Transitioning issue status, moving it to Done, creating issues/epics, and editing unrelated tickets are outside this skill's default scope.
