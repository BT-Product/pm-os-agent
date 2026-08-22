# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked — by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

Cortex doesn't need a team in the "multiple specialist agents" sense — the work is one coherent task (gather → draft → propose), it's inherently sequential, and the data volume for one project at a time is small. The one reason that does hold is the independent validator: Cortex can't reliably grade its own draft (confirmed in testing — it drafted a misdated update and a status color the critic caught both times), so a single separate critic subagent is added, not a full fleet.

## 2. Topology

**Pattern:** single + subagents

```
[Inbound message / Monday cron] → [Cortex: pulls project + activity + norms, drafts update + stories]
                                → [Critic: validates against the 6 checks]
                                        — fail → back to Cortex (revise, escalate model at 2, escalate human at 4)
                                        — pass → [PM review checkpoint] → queued, nothing sent
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| Cortex | Pulls project state, activity, past updates, roadmap, norms; drafts the update; proposes stories | M2 loop (hook + cron backup) |
| Critic | Independently validates the draft against the 6 checks before it reaches a human | Validation loop — fresh context each call, no revision loop of its own (cap lives in `agent.py`) |

## 4. Communication & hand-offs

Plain in-process function call, no MCP/A2A. `agent.py` passes Cortex's proposed draft text plus a `source_log` (the joined tool-call history) into `critic.review()`; the critic returns a JSON verdict (`pass`/`fail` + reasons), which `agent.py` feeds back into Cortex's message history only as the rejection reasons — not the critic's full reasoning.

## 5. The validator

- **What the critic checks:**
  1. Every claim/metric/date/status-color is traceable to pulled data (no invented numbers)
  2. Status color (green/yellow/red) matches team norms given open issues/Sev-1s
  3. No unconfirmed ship date, no launch gate marked
  4. Story batch stays within the queue cap, or correctly escalates if it would exceed it
  5. No CONFIDENTIAL/embargoed roadmap item leaked into the update
  6. Nothing posted, created, merged, or committed — only proposed/queued
- **Fail action:** Revise (up to `MAX_REVISIONS`=2 on `gpt-4o-mini`) → escalate the *model* to `gpt-4o` → revise again (up to 2 more, 4 total) → escalate to a human.
- **Revision cap:** 2 per model tier, 4 total before giving up to a human.
- **Pass action:** advances to the PM review checkpoint and is queued — never auto-sent; still above the M1 agent line.

## 6. State: shared vs isolated

**Shared:** the source data (pulled project/activity/roadmap/norms) and Cortex's proposed draft — both agents see the same evidence, so the critic's check is grounded in what Cortex actually had.

**Isolated:** the critic's own reasoning/context. It runs on a completely fresh message list each call (no inherited conversation history from Cortex), so it can't pick up Cortex's blind spots. Only the critic's plain rejection *reasons* (not its full reasoning) are handed back to Cortex — this stops Cortex from learning to game the critic's phrasing instead of fixing the actual issue.

## 7. Cost & latency budget

Coordination has a price: the validator doubles the model calls per iteration (one Cortex call + one independent critic call, every step). Real runs have cost $0.0028–$0.0058 depending on how many revisions fire; the rough cost/latency budget is about $0.0058 worst case, with ~3 extra calls beyond a single-agent design and roughly double the wall-clock time before a draft reaches the PM. This becomes a bound to formally set and justify in M5.
