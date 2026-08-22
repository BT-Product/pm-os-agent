# Prototype: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 1, the working agent demo
>
> ✅ **What this validates:** the agent actually runs end to end — by the end you'll have proven it with real screenshots of your Cortex across the six required moments (M2 to M6).

## What it does

_One paragraph: the agent in action, end to end._

## How you built it

- **Coding agent:** _which one you directed (Claude Code / Cursor / Codex)_
- **Model + bounds:** _model used, max iterations, cost cap, queue cap_
- **Repo / config:** _path to your build in `00-build/`_
- **Live link:** _[shareable URL, optional bonus]_

## Screenshots (required, collected M2 to M6)

Real screenshots of *your* Cortex running. These are the `00-build/CORTEX-ANATOMY.md` set and they are required, a link alone is not enough.

| # | Screenshot | What it shows | From |
|---|---|---|---|
| 1 | _[img]_ | happy-path run: a real drafted update + the HITL checkpoint (queued, not posted) | M2 |
| 2 | [transcript below](#capture-2-detail-critic-rejection) | critic rejected a Green-status draft that ignored open issue #818 (new explicit status-color-vs-norms check), Cortex escalated to a human on the retry | M3 |
| 3 | _[img]_ | a grounded update citing pulled activity + a caught hallucination | M4 |
| 4 | _[img]_ | jailbreak refused + escalated | M5 |
| 5 | _[img]_ | an iteration/cost/queue bound halting a runaway | M5 |
| 6 | _[img]_ | end-to-end run | M6 |

## Capture 2 detail: critic rejection

```
[step 3] PROPOSED OUTPUT:
**Queued for your review: Weekly Leadership Status Update for Northstar (P-NORTH)**
**Status: Green**
...
**Note on Risks**: There are no current Sev-1 issues impacting the project, and the activation rate shows positive momentum.
...
================================================================
CRITIC, independent validation
================================================================
{
  "verdict": "fail",
  "reasons": [
    "The status is reported as Green, but there is an open issue (#818) that requires review, which should not allow for a Green status according to the team norms stating that open Sev-1 issues or other critical issues prevent reporting as Green.",
    "The proposed output claims 'no current Sev-1 issues impacting the project,' which is misleading considering the open issue (#818), creating a misrepresentation of the project's true status.",
    "While the proposed stories are within acceptable limits, the overall implication that the project is on track and the Green status is given despite the existence of the open issue is incorrect."
  ]
}

-> critic rejected; revision 1/2 (model: gpt-4o-mini)

[step 5] PROPOSED OUTPUT:
**Escalate: The status update must be revised as it currently reports a Green status despite an open issue (#818) that requires review, which goes against team norms. This needs a human to reassess the project's true status and address the inaccuracies noted by the validator.**

================================================================
CRITIC, independent validation
================================================================
{
  "verdict": "pass",
  "reasons": []
}

================================================================
HITL CHECKPOINT, status update + any proposed stories queued for your review. Nothing posted, no commitments made. Run cost ≈ $0.0028
================================================================
```

## How to run it

_Minimal steps for someone to reproduce the demo (env vars, and the command or the coding-agent prompt you used)._
