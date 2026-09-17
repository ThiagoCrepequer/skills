# Compact evidence record

Use this structure in the user's work log or handoff; omit irrelevant sections. Keep sensitive SQL/binds in a private approved artifact, not in public tickets.

## Scope and contract

- Query fingerprint/report period and affected flow:
- Requested stage and authorized environments/actions:
- Repository call chain and observable invariants:
- Explicitly out of scope:

## Baseline and hypothesis

- Database/ORM versions, data size, relevant indexes/settings:
- Exact query variant and parameter provenance:
- Scenarios, permissions, result counts, cache/I/O conditions:
- Bottleneck evidence and expected plan change:
- Candidate alternatives and reason for selection/rejection:

## Validation ledger

| Gate | Evidence | Status / limitation |
| --- | --- | --- |
| Behavioral regressions | Test names, command, result | Not run / passed / failed |
| Equivalence | Both difference directions; values/order/pages | Scope and snapshot |
| ORM SQL | Capture location and structural checks | Captured / reconstructed |
| Plans | Baseline/candidate execution, buffers, reads, loops | Environment and repetitions |
| Review/build | Focused commands and findings | Full suite not implied |

## Decision and handoff

- Implemented changes versus proposals:
- Observed result, expected benefit, and confidence limits:
- Deployment prerequisites, index effects, recovery/rollback:
- Remaining risks and the single next useful action:

Avoid “no behavior change guaranteed.” State which invariants were tested and which production scenarios were measured. Keep measured database gains separate from expected overall application impact.
