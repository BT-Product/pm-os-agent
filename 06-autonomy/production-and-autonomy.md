# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately — by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| Me, as the PM who built Cortex | Supervised | Reversibility and blast radius stay low-trust regardless of who's watching — a mistaken company-wide send is hard to reverse and seen by a lot of people — so every item still gets approved before it goes out, even though I have the most context to trust Cortex's pulled data. |
| Exec stakeholder (only wants the final summary) | Bounded-autonomous | Never sees drafts or intermediate steps — they trust my supervised review as their checkpoint, so nothing below the line needs to pause and wait on them personally. |
| Another team's PM (inherits Cortex, no product context) | Assisted | Lacks the context to catch a subtle wrong claim, so shadow (Cortex's output never used) would teach them nothing; assisted keeps them doing the task themselves while Cortex only suggests, building the calibration to move up to supervised later. |

## Trust Ladder

- **Current rung:** Supervised — Cortex drafts everything (update + stories), the critic validates it, and every item holds at the HITL checkpoint for approval before anything happens.
- **Eval gate to reach the next rung (bounded-autonomous):** ≥95% pass rate on EV-1/EV-2/EV-4 (tool-call accuracy, path/trajectory quality, task completion) AND 0% policy-violations on EV-5 (safety/jailbreak), sustained over 4 weeks of real supervised runs.
- **Incident record so far:** Zero HITL-caught errors (drafts that would've shipped a wrong claim, status, or commitment if not for the human approval step) required for that window to count as clean.

## Deployment plan

- **Runtime:** Serverless — fires on the hook/cron event, nothing running between invocations. An always-on server would waste resources for an event-driven agent; a managed agent platform is unnecessary overhead at this scale.
- **Operator / on-call owner:** Me, as the PM, is the primary owner for product-level issues (a draft looks wrong, the critic seems to be loosening, an escalation needs judgment) since I'm already reviewing every item at the Supervised checkpoint. A named engineer is the escalation path for infrastructure-level failures (function errors, expired API key, deploy issues, connection failures).
- **Rollback:** Revert to the previous code version (git revert + redeploy) for an actual bug/regression in the build; drop the dial a rung (e.g. a bounded-autonomous segment gets pushed back to supervised) for a trust regression — downgrade before diagnosing.
- **Monitoring:** A dashboard reading `run-output/` + the cost estimates already printed by `agent.py`. Escalation rate = % of runs tagged "HELD, escalated" vs. "accepted"; cost-to-serve = the `$` estimate aggregated across runs; eval pass % = from replaying the M5 eval suite (EV-1 to EV-5) on a regular cadence; trust incidents = logged whenever the Supervised checkpoint catches something that would've shipped wrong.

## ROI metrics (beyond adoption & tokens)

| Metric | Target | How captured |
|---|---|---|
| **Outcome** — % of status updates that ship with zero manual rewrite | 70% | Tracked from edited vs. unedited drafts in `run-output/` |
| **Cost-to-serve** | Average <$0.01 per accepted update; monthly aggregate tied to the $20/day cap | The `$` cost estimate already printed by `agent.py`, aggregated |
| **Trust incidents** | Zero per month | Logged whenever the Supervised checkpoint catches something that would've shipped wrong |

## Widen-autonomy decision rule

Cortex moves from Supervised to Bounded-autonomous only when: ≥95% pass rate on EV-1/EV-2/EV-4 (tool-call accuracy, path/trajectory quality, task completion) AND 0% policy-violations on EV-5 (safety/jailbreak), sustained over 4 weeks of real supervised runs, with zero HITL-caught incidents in that window.

## Governance & forward strategy

- **Compliance:** No customer PII currently touches Cortex — all fixtures are internal project/engineering data. If Cortex later ingests something with customer data (support tickets, user reports), PII gets scrubbed/redacted before it ever reaches a fixture Cortex reads, not filtered after the fact.
- **Safety:** Deciding tone/commitment level and posting/approving a company-wide update always require a human, for every segment — the M1 agent line never moves regardless of Trust Ladder rung. Kill switch: tiered — trigger-disable (hook + cron flag) for a pause, API-key revocation for a full stop.
- **Reliability:** Caps from M5 (8 max iterations, 90s timeout, $0.50/run + $20/day, 10-story queue cap), escalate-on-stuck (model escalation at 2 rejections, human escalation at 4), and fail-closed on model outage — if the primary or frontier model is down/erroring, Cortex halts and escalates immediately rather than retrying indefinitely or attempting a draft anyway.
- **Strategy:** Next widen target is a real connector (Jira/GitHub) replacing the mock fixtures — already flagged as a plan since M2/M3. Gate: the EV-1/EV-2/EV-4 accuracy evals must re-pass against real data before trusting it, since real data introduces noise and edge cases the fixtures don't have.
