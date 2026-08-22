# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs — by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.
>
> 🗂️ **How the lab maps to this file:** In **Part A** (before the lecture) you don't edit this file — you rough-draft on scratch, focused on the per-source calls in **section 2** plus a quick remember/forget + "how it rots" sketch. In **Part B** (after the lecture) you complete **all five sections**; the Lab Guide's guided builder writes this file for you to copy in and commit.

## 1. Context budget

Priority order, if something has to be trimmed: (1) the task brief — defines the job itself; (2) team norms — governs how Cortex must behave; (3) project + activity — the evidence the update is grounded in, pulled just-in-time; (4) roadmap — included when relevant, mainly to check confidentiality flags; (5) past updates — precedent for tone/format, lowest priority since it's not required to produce a correct update.

## 2. Retrieve vs. long-context: per source

For each data source, decide: **retrieve** (narrow a large/changing corpus to the relevant slice) or **long-context** (just include a bounded set you can reason over).

| Source | Size / volatility | Decision | Why |
|---|---|---|---|
| `get_activity` | Large, grows every week | Retrieve | Too big/changing to include whole every run; narrowed to the relevant project's activity |
| `search_past_updates` | Unbounded, grows without limit | Retrieve | Same reason as activity — the corpus only grows, can't include it all |
| `get_roadmap` | Medium, bounded | Long-context | Small enough to include whole (the code returns the full file every time); confidentiality is a separate governance rule, not a reason to narrow |
| `get_norms` | Medium, bounded | Long-context | Small enough to include whole so Cortex can cite the exact rule it relied on |
| `get_task` | One static doc per run | Long-context | Single bounded document, reason over the whole thing |

## 3. Retrieval quality plan

Only the two retrieved sources need agentic moves (the long-context sources are included whole, nothing to grade or verify against retrieval).

| Retrieved source | Move | Why (the actual failure mode it catches) |
|---|---|---|
| `get_activity` | **Self-verification** | The retrieval itself is correct (single exact-ID lookup, no ambiguity) — the failure is the *draft* not reconciling its claims (e.g. status color) against the activity it already pulled. Self-verification (the critic re-checking draft vs. source log) is what catches a Green status sitting next to an open issue. |
| `search_past_updates` | **Document grading** | The naive keyword-overlap search falls back to returning the first 2 corpus items when nothing matches — regardless of relevance. Document grading is needed to catch an irrelevant past update being treated as precedent. |

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (in-loop) | Pulled activity, draft-in-progress, critic's verdict | This run only, purged at exit |
| **Episodic** (past runs) | Last week's status update | Per-project, rolling 1-week TTL — this week's replaces last week's after the run completes |
| **Semantic** (durable facts/prefs) | Team norms, roadmap facts | Per-project, indefinite — updated manually when the source changes (e.g. a data-pack ingest) |
| **Shared** (across agents) | Source data + the draft, shared between Cortex and the critic | Same as working memory's TTL; the critic's internal reasoning stays isolated, only its plain verdict/reasons feed back (per M3 orchestration map) |

## 5. Memory risks & mitigations

| Risk | Where it bites Cortex | Mitigation |
|---|---|---|
| **Drift** | During critic evaluation — a critic that remembered its own past verdicts could gradually loosen its standard over many runs | The critic runs with a completely fresh context every time, no memory of prior verdicts to drift from |
| **Poisoning** | If a fixture entry (past update, decision log) were corrupted with false-but-relevant-looking content | Write-access control — Cortex has no write tool to those files, only a human via git commit can change them — plus self-verification: claims must trace to current pulled activity, not borrowed precedent |
| **Staleness** | Cortex pulling activity/roadmap/norms against fixture files nobody's refreshed, confidently reporting old numbers as current (lived through this before today's ingest: 41% vs. the real 43%) | The manual ingest workflow (download → add → run → push) as the stopgap; forward-note that a live connector (already flagged as a plan in the M3 roster) would remove the manual-refresh dependency |
| **Confidential / retention** | Leaking an embargoed roadmap item (Orbit, Pulsar) into a company-wide update | Layered: the critic explicitly checks for this (no CONFIDENTIAL item in an external/company-wide update); there is no publish tool at all in `tools.py`, so there's no channel for a leak to reach anyone even if it slipped through; human review is the final backstop, not the only one |
