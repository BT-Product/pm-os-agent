# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop — by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** Hook (primary) + Cron (backup)

**Why this type?** A hook on inbound messages means Cortex only runs when there's actual work — a heartbeat that polls every 15 minutes burns tokens for no reason when nothing's changed. A Monday 9am cron sweep is the backstop: if a week goes by with no inbound message, leadership shouldn't be looking at stale info just because nobody happened to ask.

**Ruled out:** Heartbeat alone — unnecessary cost, runs whether or not there's anything new. Goal loop — no clear self-validated "done" for a status update; it's a production task, not a goal to chase. Hook-only — risks silently going stale if a week passes with no inbound message. Cron-only — misses same-week requests that come in off-schedule.

**Idempotency / dedupe:** Dedupe by message ID — if the same inbound message fires the hook twice (retry, duplicate delivery), Cortex checks whether that message ID has already produced a queued draft and skips redrafting if so.

## 2. Goal / definition of done

A weekly leadership status update grounded in real pulled activity, plus a capped batch of next-sprint story proposals, both queued for human approval. Cortex never posts the update or adds stories to the real backlog.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | Critic approves the draft, and both the draft and story batch are queued with no unresolved data gaps | Save draft to `run-output/`, hold for human review |
| **Stuck / give up** | 2 critic rejections on `gpt-4o-mini` → escalate to a frontier model; 2 more rejections there (4 total) → give up. Or: a required tool call (e.g. `get_activity`) fails/returns nothing after repeated attempts | Log the failure reason, hold the last draft, escalate to human |
| **Escalate to human** | Project flagged embargoed · draft implies a public GA-date commitment · proposed story batch exceeds `CORTEX_MAX_QUEUE_ITEMS` · (also fires on stuck conditions above) | Immediate stop, flag for human, nothing posted |

## 4. State

Last week's status update, the roadmap, and team norms persist per-project across runs (so Cortex can say things like "up from 39%"). No project's state is visible to another project's run — Northstar's context never bleeds into Vega's or vice versa. Everything else (this run's activity pull, this run's draft) is purged after the run ends.

## 5. The five things a loop can lean on

_`state` is always-on. `connectors` only if you already have one wired (e.g. a Jira key or Google MCP) — otherwise just note it as a plan. `skills`, `subagents`, `work tree` scale with autonomy; "not needed yet, because…" is a valid answer._

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Not needed yet, because Cortex only produces a queued text draft — it never modifies the repo or runs code, so there's no shared state for concurrent runs to collide over. |
| **Skills** (reusable capabilities) | Not needed yet, because the base tools (`get_project`, `get_activity`, `search_past_updates`, `propose_stories`) already cover the workflow. |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | Not wired yet — still running on the mock fixture tools. Plan: swap in a real Jira key / Google MCP once available, same tool interface. |
| **Subagents** (independent check when the loop can't grade itself) | Needed, and already live: the critic is an independent subagent that validates the draft rather than Cortex grading its own work. See M3 orchestration-map.md for more detail. |
| **State tracking** | See §4 above — per-project persistence of last week's update, roadmap, and norms; no cross-project leakage. |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules — you'll add them to their own deliverables then, not here.

## Link to live loop

_[path to your agent in `00-build/`]_
