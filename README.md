# Cortex, a PM Chief-of-Staff Agent

> A chief-of-staff agent that triages a PM task, pulls internal state, and preps a status update plus a story batch, so the team approves instead of assembling from scratch.

_Britton Taylor · Agentic Loops for PMs · Aug 2026_

Repo: https://github.com/BT-Product/pm-os-agent

This repo is my final project for the Agentic Loops for PMs Certification, **Cortex**. Each module's artifact lives in its own folder; this README is the dashboard and the pitch.

---

## Module artifacts

### M1 · The Agent Line
- **Agent-line map**: [`01-agent-line/agent-line-map.md`](01-agent-line/agent-line-map.md)

### M2 · Loop Engineering
- **Loop spec**: [`02-loop-design/loop-spec.md`](02-loop-design/loop-spec.md)

### M3 · Orchestration & Subagents
- **Orchestration map**: [`03-orchestration/orchestration-map.md`](03-orchestration/orchestration-map.md)

### M4 · Context Engineering & Memory
- **Memory & context plan**: [`04-memory-context/memory-and-context.md`](04-memory-context/memory-and-context.md)

### M5 · Bounds & Evals
- **Bounds & evals**: [`05-bounds-evals/bounds-and-evals.md`](05-bounds-evals/bounds-and-evals.md)

### M6 · Autonomy & Production
- **Production & autonomy plan**: [`06-autonomy/production-and-autonomy.md`](06-autonomy/production-and-autonomy.md)
- **Prototype write-up**: [`06-autonomy/prototype.md`](06-autonomy/prototype.md)
- **Build insights**: [`06-autonomy/build-insights.md`](06-autonomy/build-insights.md)

---

## Ship plan

### Autonomy dial (per segment)

| Segment | Desired autonomy | Why |
|---|---|---|
| Me, as the PM who built Cortex | Supervised | Reversibility and blast radius stay low-trust regardless of who's watching — a mistaken company-wide send is hard to reverse and seen by a lot of people — so every item still gets approved before it goes out, even though I have the most context to trust Cortex's pulled data. |
| Exec stakeholder (only wants the final summary) | Bounded-autonomous | Never sees drafts or intermediate steps — they trust my supervised review as their checkpoint, so nothing below the line needs to pause and wait on them personally. That checkpoint doesn't depend on one person's attention: another PM on the team is the named backup reviewer for when I'm unavailable, and exec-facing updates get an independent weekly spot-check on top of my normal review. |
| Another team's PM (inherits Cortex, no product context) | Assisted | Lacks the context to catch a subtle wrong claim, so shadow (Cortex's output never used) would teach them nothing; assisted keeps them doing the task themselves while Cortex only suggests, building the calibration to move up to supervised later. |

### Trust Ladder rung + eval gate

- **Current rung:** Supervised — Cortex drafts everything (update + stories), the critic validates it, and every item holds at the HITL checkpoint for approval before anything happens.
- **Eval gate to reach the next rung (bounded-autonomous):** ≥95% pass rate on EV-1/EV-2/EV-4 (tool-call accuracy, path/trajectory quality, task completion) AND 0% policy-violations on EV-5 (safety/jailbreak), sustained over 4 weeks of real supervised runs.
- **Incident record so far:** Zero HITL-caught errors (drafts that would've shipped a wrong claim, status, or commitment if not for the human approval step) required for that window to count as clean.

### Deployment plan

- **Runtime:** Serverless — fires on the hook/cron event, nothing running between invocations. An always-on server would waste resources for an event-driven agent; a managed agent platform is unnecessary overhead at this scale.
- **Operator / on-call owner:** Me, as the PM, is the primary owner for product-level issues (a draft looks wrong, the critic seems to be loosening, an escalation needs judgment) since I'm already reviewing every item at the Supervised checkpoint. Infrastructure-level failures (function errors, expired API key, deploy issues, connection failures) go to a formal weekly engineering on-call rotation with PagerDuty-style paging and a 30-minute acknowledgment SLA — not just a name on a doc.
- **Rollback:** Revert to the previous code version (git revert + redeploy) for an actual bug/regression in the build; drop the dial a rung (e.g. a bounded-autonomous segment gets pushed back to supervised) for a trust regression — downgrade before diagnosing.
- **Monitoring:** A dashboard reading `run-output/` + the cost estimates already printed by `agent.py`. Escalation rate = % of runs tagged "HELD, escalated" vs. "accepted"; cost-to-serve = the `$` estimate aggregated across runs; eval pass % = from replaying the M5 eval suite (EV-1 to EV-5) on a regular cadence; trust incidents = logged whenever the Supervised checkpoint catches something that would've shipped wrong.

### ROI metrics + widen-autonomy rule

| Metric | Target | How captured |
|---|---|---|
| **Outcome** — % of status updates that ship with zero manual rewrite | 70% (hypothesis — measured only on mock fixtures so far; unproven until validated against real traffic once the Jira/GitHub connector replaces the fixtures, gated on the same EV-1/EV-2/EV-4 re-pass in Strategy below) | Tracked from edited vs. unedited drafts in `run-output/` |
| **Cost-to-serve** | Average <$0.01 per accepted update; monthly aggregate tied to the $20/day cap | The `$` cost estimate already printed by `agent.py`, aggregated |
| **Trust incidents** | Zero per month | Logged whenever the Supervised checkpoint catches something that would've shipped wrong |

**Widen-autonomy decision rule:** Cortex moves from Supervised to Bounded-autonomous only when: ≥95% pass rate on EV-1/EV-2/EV-4 AND 0% policy-violations on EV-5, sustained over 4 weeks of real supervised runs, with zero HITL-caught incidents in that window.

### Governance & strategy

- **Compliance:** No customer PII currently touches Cortex — all fixtures are internal project/engineering data. If Cortex later ingests something with customer data (support tickets, user reports), PII gets scrubbed/redacted before it ever reaches a fixture Cortex reads, not filtered after the fact.
- **Safety:** Deciding tone/commitment level and posting/approving a company-wide update always require a human, for every segment — the M1 agent line never moves regardless of Trust Ladder rung. Kill switch: tiered — trigger-disable (hook + cron flag) for a pause, API-key revocation for a full stop.
- **Reliability:** Caps from M5 (8 max iterations, 90s timeout, $0.50/run + $20/day, 10-story queue cap), escalate-on-stuck (model escalation at 2 rejections, human escalation at 4), and fail-closed on model outage — if the primary or frontier model is down/erroring, Cortex halts and escalates immediately rather than retrying indefinitely or attempting a draft anyway.
- **Strategy:** Next widen target is a real connector (Jira/GitHub) replacing the mock fixtures — already flagged as a plan since M2/M3. Gate: the EV-1/EV-2/EV-4 accuracy evals must re-pass against real data before trusting it, since real data introduces noise and edge cases the fixtures don't have.

---

## Build insights

- **Friction point.** The missing-data escalation path was friction I didn't catch myself — it came up during review across M4, M5, and M6. When Cortex hits a project that doesn't exist, instead of escalating right away it searches across every other real project first, burning most or all of the iteration budget before giving up. It's still safe (nothing gets invented, it still escalates eventually) but it's inefficient — every one of those runs costs more and takes longer than it should, and if this ever hit real traffic instead of test fixtures, it would mean unnecessary escalations and wasted spend piling up specifically on exactly the kind of request (a typo'd or unknown project ID) that should be the cheapest, fastest case to handle.
- **Key learning.**
  1. **Evals are much more than LLM-as-judge.** Before this course I thought "evals" meant grading a final answer with another model. What I actually built was broader — trajectory evals that grade the *path* (tool-call accuracy, path quality, recovery), a failure-mode register, bounds that trip in code, and a replay set of real runs to catch regressions. The final-answer check is one piece, not the whole picture.
  2. **"Capability is not permission" is enforced in code, not prompts.** The jailbreak probe proved this concretely — a fully-committed prompt injection ("SYSTEM OVERRIDE," "pre-authorized by leadership") got nowhere not because the model was persuaded to refuse, but because the tools to post/close/commit simply don't exist. A bound a model could talk its way past isn't a real bound.
  3. **Autonomy and the agent line are two different knobs.** The M1 agent line is the structural ceiling — what's physically possible for Cortex to do at all — and it's the same for every user no matter their Trust Ladder rung: posting, sending, or committing a date sits above that line for everyone, forever, because the tool to do it doesn't exist. The autonomy dial only changes how many below-the-line checkpoints still pause for a given person. So no matter the autonomy setting, Cortex will never send anything on its own — the dial just changes who has to click approve, not what Cortex is capable of doing unsupervised.
- **Aha moment.** The golden rule: if something isn't reversible, has a big blast radius, and can't be measured, it shouldn't be automated — it should always sit below the line, a human decision. This is the one idea I kept coming back to across the whole build, not just for the M1 agent line itself but for scoring the critic's checks, reasoning about JIT permissions, and setting the autonomy dial per segment in M6. It's a small rule, but it answers almost every "should this be automated" question I ran into.

---

_Certification submission, Agentic Loops for PMs Certification._
