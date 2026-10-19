# Enrichment and short interview

Turn a thin issue into a richer, evidence-backed task without turning the run into a questionnaire. The method adapts the grilling interview behind [grill-me](https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me): finding facts is the agent's job, decisions belong to the user, every question carries a recommended answer, and dependent questions wait for a later round. Unlike an open-ended grilling session, this interview is bounded: it exists to fill one task, not to settle a whole design.

Run it after retrieving the issue context and before building the change plan in [the skill entrypoint](../SKILL.md). It never writes to Jira by itself. Everything learned feeds the normal Reconcile plan, field rules, and authorization.

## 1. Map the gaps

Read the issue as a PM/PO, tester, and developer would. For each dimension below, classify it as **covered**, **closable by research**, **needs a user decision**, or **not relevant to this task**. Scale the bar to the task: a 1-point correction needs a problem, expected behavior, and one or two acceptance conditions, not every row.

| Dimension | What a gap looks like |
| --- | --- |
| Context / report | Who is affected, where, since when, and the original report or reproduction are missing or vague |
| Objective / value | The title states a solution but not the problem or outcome it serves |
| Expected behavior and business rules | Desired behavior, states, or rules are implicit or contradictory |
| Acceptance conditions | No observable way to tell the task is done |
| Scope and out of scope | Boundaries are unclear, e.g. which screens, apps, units, integrations, or historical data are included |
| Affected surfaces | Unknown whether web, app, backend, reports, or integrations change |
| Dependencies and links | Blockers, predecessor tasks, external parties, or related issues are not identified |
| Data and compatibility | Migration, backfill, correction of historical data, or contract changes are possible but unstated |
| Risk and criticality | Financial, health, billing, access-control, or data-integrity impact is not acknowledged |
| Estimate inputs | Uncertainty, coupling, or validation demands are unknown enough to move the estimate across a band |
| Area / epic | The system area the task touches cannot be determined |
| Classification | The type looks wrong (e.g. a defect filed as an improvement) or the task looks like an umbrella that should be decomposed |

Type and decomposition problems are reported, not fixed: changing issue type, creating subtasks, or splitting issues is outside this skill.

## 2. Research before asking

Close every gap you can from the environment first. Ask the user only for what the environment cannot answer. Use a bounded ladder and stop as soon as the gap is closed:

1. **The issue itself:** full description, comments, attachments, links, and change history when the connection exposes it.
2. **Hierarchy:** parent epic, sibling tasks under the same epic or parent, and subtasks.
3. **Linked and similar issues:** explicit links first; then a small JQL text search in the same project on the decisive terms of the summary (for example `project = <project> AND text ~ "<term>" ORDER BY updated DESC`), reading a handful of results rather than scanning the site. Look for duplicates, predecessors, earlier versions of the same feature (`pt.2`, `v1`), and reports of the same symptom.
4. **Comparable estimated issues:** the calibration sample from [estimation-and-description.md](estimation-and-description.md) doubles as evidence for scope and acceptance patterns.
5. **Other connected sources, when available and relevant:** repository code or PRs to locate the affected surfaces (read-only; this is not a code review), Sentry for an error a bug report describes, Confluence or other documentation for business rules. Skip sources that are not connected; do not ask the user to connect them.

Rules for research results:

- A fact found in a source can enter the description as a fact. Keep its reference (issue key, link) when it helps the reader.
- An inference from similar issues, code structure, or naming is an **assumption**, not a fact. Treat it as in section 5.
- Summarize sensitive material instead of copying it: no credentials, tokens, access-bearing URLs, personal or health data, or raw stack traces in the description. Follow the ticket's existing practice for customer or tenant names.
- Research is read-only. Do not link, comment on, edit, or close related issues; report a likely duplicate or overlap instead.

## 3. Choose the questions

A remaining gap becomes a question only when **all** of these hold:

- It is a decision or a fact only the user or a stakeholder holds (priority between options, intended scope, a business rule, an unrecorded agreement with a customer).
- The answer materially changes the task: scope, acceptance conditions, the estimate band, or the epic.
- No safe default exists. When a reasonable default exists and a wrong guess is cheap to correct, assume it and mark it instead of asking.

Rank the qualifying questions by impact: scope boundaries and acceptance first, then estimate drivers, then everything else. Limits:

- **At most 5 questions per task in total, ideally 3 or fewer.** Drop the lowest-impact ones and record them as assumptions.
- **At most 2 rounds.** The first round asks the frontier, every qualifying question whose answer does not depend on another open question. A second round is allowed only for questions unblocked by the first answers, within the same total cap.
- **Zero questions is a normal outcome.** When nothing qualifies, say so in one line and continue.

Never ask:

- Anything already stated in the issue, its comments, links, or the conversation.
- Anything the research ladder could answer: which component, who reported it, whether an error appears in Sentry, what a linked issue says.
- For the story-point number itself. Estimating is this skill's job; ask at most for the scope decision that drives the estimate.
- Implementation choices that belong to the developer, or stylistic preferences about the description.

## 4. Ask in the grill format

Follow the user's language. Open with one line saying how many questions there are and that the user can answer by number or accept everything. For each question:

```text
❓ Q1 - <short title>: <one or two sentences: what research found, with a key or link, and the decision needed>
➡️ <recommended answer> (<effect: e.g. "adds acceptance condition X", "keeps 5 points", "8 points if not">)
```

- Word each question so that "yes" accepts the recommendation.
- Keep the body short; the detail lives in the research, not in the question.
- Offer a single acceptance phrase for the whole round (for example "pode seguir" / "go ahead").
- If the user prefers one question at a time, ask sequentially in impact order with the same cap.

Handling answers:

- **Yes / the recommendation:** settled as confirmed.
- **"Go ahead" / "pode seguir", or a request to skip:** every open recommendation becomes a marked assumption. Do not ask again in this run.
- **"I don't know":** keep it as an open question, lower the estimate's stated confidence when it is an estimate driver, and do not guess a requirement from it.
- **An answer that conflicts with existing requirements:** treat it as a scope decision. Preserve the original statement and record the decision, as the description rules require.
- **An answer that greatly expands scope:** flag decomposition for planning; do not create issues.
- **An answer that reveals a new decision:** it may enter round 2 only within the cap; otherwise record it as an open question.

The answered round is the confirmation. Proceed to the change plan under the invocation's existing authorization; do not add an extra approval round unless the user asked for a proposal only.

## 5. Record assumptions and open questions

Never present an assumption as a confirmed requirement, report, or decision.

- **In the description:** when an unconfirmed assumption shapes the scope or acceptance conditions you write, keep it in a short, clearly labeled section in the issue's language (for example "Premissas a confirmar" / "Assumptions to confirm"), one line each, with its source when there is one. Omit the section when there are no such assumptions.
- **In the result:** list every assumption, unanswered question, and estimate confidence, so the caller or user can confirm later.
- **On a later run:** when an assumption is confirmed, move it into the body as a fact and remove it from the labeled section; when it is rejected, correct the text and keep the decision where it matters.
- Open questions are not posted as Jira comments. The only comment this skill writes is the pending-findings comment defined in [jira-updates.md](jira-updates.md).

## Non-interactive and batch runs

Skip the questions entirely when any of these apply:

- A caller such as `finish-task` invoked the skill. The caller's approved decisions are authoritative; the run must not stall on user input.
- The run is headless or scheduled, or there is no live user to answer.
- The user asked for no questions, or already said "go ahead" for this run.

Still map gaps and research. Write marked assumptions under the normal description rules and return the questions that would have been asked, each with its recommendation, so the caller or user can raise them.

For a batch of issues with a live user, do not interview per issue. After researching all of them, ask one consolidated round of at most 5 questions across the whole batch, limited to the gaps with the largest effect on scope or estimates, keyed by issue. Record everything else as assumptions.
