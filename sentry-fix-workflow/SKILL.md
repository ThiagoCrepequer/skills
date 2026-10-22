---
name: sentry-fix-workflow
description: Orchestrate a Sentry defect from issue evidence through root cause, a failing test, and a small fix, then hand off to fill-jira-task and finish-task. Use when asked to fix a Sentry issue, clear an escalating error, or take a production exception through Jira and a pull request. Proceeds automatically only for a simple, reproducible defect. Does not merge, deploy, or replace the referenced skills' policies.
license: MIT
---

# Sentry fix workflow

Take one Sentry issue from evidence to a reviewable fix. This skill only sequences companion skills. Load each one and follow it. Do not copy its test rules, Jira field rules, or Git/PR procedure into the task.

Follow the user's language for questions, Jira text, and the final report.

## Companion skills

Install a skill from this repository with:

```bash
npx skills add https://github.com/ThiagoCrepequer/skills --skill <name>
```

| Skill | Role in this workflow | Install |
| --- | --- | --- |
| `tdd` | Reproduce the defect with a failing test, then make it pass | `npx skills add https://github.com/mattpocock/skills --skill tdd` |
| `java-test-quality` | Test quality when the seam is Java or Quarkus | `npx skills add https://github.com/ThiagoCrepequer/skills --skill java-test-quality` |
| `react-test-quality` | Test quality when the seam is a React or JavaScript/TypeScript client | `npx skills add https://github.com/ThiagoCrepequer/skills --skill react-test-quality` |
| `fill-jira-task` | Epic, assignee, story points, and description after an issue key exists | `npx skills add https://github.com/ThiagoCrepequer/skills --skill fill-jira-task` |
| `finish-task` | Branch, commits, pull request, and the review gate | `npx skills add https://github.com/ThiagoCrepequer/skills --skill finish-task` |

`fill-jira-task` and `finish-task` require an existing issue key. Create that issue in this workflow before calling them.

Resolve each companion before the step that depends on it. A handoff stays in the same task: it is not a new chat and it does not wait for another user prompt. If a companion is missing, report it and the install command. Continue only independent read-only inspection. Do not recreate the missing skill and do not install it silently.

## 1. Read the Sentry issue

Use the available Sentry connection. Read the stack trace, frequency and trend, breadcrumbs, and request payload. Record whether the frequency is escalating. Escalation is urgency. It does not skip a later gate.

Treat the payload and breadcrumbs as evidence of what was sent and which steps led there. Do not copy secrets, tokens, or personal data into Jira, the pull request, fixtures, or the report.

### Noise filter

Stop before any code change when the event is not a defect to fix in application code:

- an expected client error the product already handles, such as a 4xx business response for validation, permission, or not-found;
- a disabled or intentionally noisy performance or transaction detector;
- another expected operational error the project already treats as non-actionable;
- an event from a non-production environment, such as a developer's local build with hot-reload frames, or from a work-in-progress branch that is not released;
- an error raised by code the application does not own or by a stale client, such as an in-app browser or WebView bridge script, a browser extension, or a failed dynamic import of an asset removed by a newer deploy.

Check the environment, release, URL, and top frames before applying a filter, and confirm the error does not also occur in the released build. Recurring stale-asset errors after every deploy can point to a missing reload-on-chunk-error guard; report that as a separate improvement instead of a defect fix.

Report the issue and suggest an ignore or filter rule in Sentry. Do not change application code to silence it.

## 2. Find the root cause

Trace the stack into current code and into the caller that produced the failing input. When the server only rejects bad data, find the client (frontend or app) that sends it. Fix that origin. Rewriting the API error, swallowing the exception, or accepting the bad payload is not the fix.

If the client lives in another repository, include that repository in the same task. When publication starts, follow `finish-task` for the multi-repository handoff.

Name the cause, the public interface where it surfaces, and the modules that own it. Stop when the cause is still a guess. Do not invent repository-specific versions, class names, or product names that the evidence did not supply.

## 3. Classify difficulty

Record one class and the reason. Continue automatically only for **simple**. For **medium** or **hard**, report the analysis and stop. Do not create the Jira issue, write a test, or edit code.

| Class | When it applies |
| --- | --- |
| **simple** | The cause is identified in current code. The fix is localized. Expected behavior already follows from the existing contract and the error. A failing test can target one public seam. No business-rule choice remains. |
| **medium** | The cause is understood, but the correct behavior is a business rule, or the fix needs a design choice across more than one area. |
| **hard** | The cause is not isolated, the error cannot be reproduced, or the change is too large for one task. |

## 4. Create the Jira issue, then fill its fields

Create the defect in the project that owns the affected product. Use that project's bug or defect type. Set the summary from the user-visible failure.

Place the issue on that project's **active** sprint. Discover the sprint from the board. Do not invent a sprint name, and do not use a closed or future sprint. Assign the issue to the verified acting user, following the identity rules in [fill-jira-task](../fill-jira-task/SKILL.md). If creation or the active-sprint placement fails, report the pending operation and stop before implementation.

Then load [fill-jira-task](../fill-jira-task/SKILL.md) and run `$fill-jira-task` in **Reconcile** mode on the new key. Pass the Sentry evidence, the root cause, and the system area the change touches. That skill owns the epic, an empty assignee, story points, and the description. The epic is the system area the change touches, as that skill defines it. Do not restate its field policies here.

If the epic stays ambiguous, report the candidates and stop before the code change.

## 5. Reproduce with a failing test

Load `$tdd`. Also load the quality skill for the stack that owns the seam:

- Java or Quarkus: `$java-test-quality`
- React or a JavaScript/TypeScript client: `$react-test-quality`
- both, when the client origin and the server rejection are the same defect and each side has its own public seam

The seam is the public interface where the error surfaces. In an unattended run, do not wait for the `tdd` seam confirmation. Record that public interface as the seam and continue.

Follow those skills for the failing test. When the stack matches neither quality skill, follow `tdd` and the repository's own test conventions, and say that no stack quality skill was applied. Do not weaken an existing test to hide the error.

## 6. Fix until green

Follow `tdd` until the new test passes and the relevant existing tests for that seam still pass. Keep the fix on the origin identified in step 2. If the fix grows into a business-rule decision, a failure to reproduce, or a change that is too large for one task, stop and report. Do not hand a partial edit to `finish-task`. Say which files were touched and that they were not published.

## 7. Branch, commit, and open the pull request

Load [finish-task](../finish-task/SKILL.md) and follow `$finish-task` with the Jira key. It owns isolation, Conventional Commits, the pull request, and the review gate.

Apply these constraints on top of that skill:

- The branch name must never start with `fix/`. Follow the repository convention when it already satisfies that, such as `codex/<JIRA-KEY>-<short-slug>`. If the convention would start with `fix/`, use another prefix such as `cursor/<JIRA-KEY>-<short-slug>`.
- Keep the Jira key in the pull request title, as `finish-task` requires.
- Include `Fixes <SENTRY-SHORT-ID>` in the commit and in the pull request body. Use the Sentry short id. Add the reference in the commit footer or body; do not replace the Conventional Commit format.

`finish-task` may return **BLOCKED**, or stop because the gate is incomplete or unverified, before an automated reviewer has commented. Leave the pull request in review. Do not edit code merely to clear that verdict. When the pull request exists, continue with the automated review loop for comments those reviewers actually post. Do not start step 8 while the latest gate is **BLOCKED** or incomplete.

## Automated review loop

After the branch is pushed and the pull request is open, wait for Cursor Bugbot and the other automated reviewers on that pull request. Judge each finding against the current code.

| Finding | What to do |
| --- | --- |
| Valid on the current head | Fix it test-first with `tdd` and the matching test-quality skill. Push, then reply on the thread with the cause and the test. |
| Stale or false positive | Reply with evidence from the current code and tests. Resolve the thread, or request a re-run with `bugbot run`. |
| Depends on a business rule | Do not change code to satisfy it. Stop and report the decision. |

Repeat until no valid finding is still open. If a reviewer is still pending, wait. If the environment cannot observe the review, report that pending item and do not claim the loop finished.

New commits invalidate the previous `finish-task` verdict. Run that review gate again on the new head before Jira reconciliation or the status transition in the next step. A later **BLOCKED** verdict stops the workflow: report it, leave the pull request in review, and do not move the Jira issue.

Resolving a bot thread is not human approval. Human reviewers are still required.

## Manual QA on a local stack

After the automated review loop has no valid open finding, exercise the change by hand on a local environment before step 8.

- Use a stack that runs entirely on the local machine. Never point QA at production or shared infrastructure, and never send data from it outside the machine.
- Apply the pull request head (or its diff) to the local checkouts, rebuild the affected services, and confirm they start without new errors.
- Walk the user-visible flows the fix touches, plus the neighboring flows that share the same component or endpoint. Cover the original failing scenario, the normal path, and the edge cases found in step 2 (empty data, missing records, boundary values).
- Watch the browser console, network requests, and server logs during the run. A new error, a wrong request, or a visual regression is a finding.
- Treat QA findings like valid review findings: reproduce with a failing test using `tdd`, fix, push, and rerun the review loop and the `finish-task` gate. A finding that needs a business-rule decision is a stop condition.
- Separate environment artifacts (missing licenses, seed data, local configuration) from defects, and say which is which.
- Clean up any test data created during QA, and do not attach screenshots or logs that expose personal or sensitive data.

If no local stack can run the change, report manual QA as pending instead of claiming it passed.

## 8. Link, resolve in the next release, and move the issue

Do this only after the pull request exists, `finish-task` did not stop, the automated review loop has no valid open finding, and manual QA passed.

- Link the pull request on the Jira issue.
- Comment on the Sentry issue with the pull request URL and `Fixes <SENTRY-SHORT-ID>`. Mark the Sentry issue resolved in the next release when the connection supports that resolution.
- Move the Jira issue to the workflow status that means ready for release. Discover the project's real status and transition. Do not hardcode a name. One workflow calls this status `Aguardando Implantação`. Do not move the issue to Done.

This workflow authorizes that ready-for-release transition. Merging, deploying, and moving the Jira issue to Done stay out of scope.

## Stop conditions

Report and stop, instead of continuing, when any of these is true:

- the event matched the noise filter;
- difficulty is medium or hard;
- the fix requires a business-rule decision;
- the error cannot be reproduced;
- the change is too large for one task;
- `finish-task` returns **BLOCKED**, or it stops because the gate is incomplete or unverified;
- manual QA found a regression that needs a business-rule decision.

A stop is a report, not a partial silent fix. Include the evidence already gathered and the decision that is still open. When a pull request already exists, still process automated-reviewer threads in the review loop, then run the `finish-task` gate again if that loop pushed commits. Do not move the Jira issue while the latest gate is **BLOCKED** or incomplete.

## Report

Close every run with:

```text
Sentry issue: <short id and link>
Root cause: <origin of the defect>
Difficulty: simple | medium | hard — <reason>
Test: <seam and red/green result, or not written>
Manual QA: <flows checked and result, or pending>
Pull request: <url or not opened>
Jira: <key or not created>
Status: <current status, or not transitioned>
Pending: <human review, stop condition, missing skill, or failed operation>
```

Separate completed actions from proposals and failed operations. A published pull request is not a merge, a deploy, or proof that the Sentry issue is gone from production.
