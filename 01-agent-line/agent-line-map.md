# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner — by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | L | H | Below | none |
| Decide relevant context | H | L | H | Below | none |
| Draft the update | H | L | M | Below | spot-check |
| Decide tone/commitment level | L | H | L | Above | required |
| Flag at-risk/escalation | H | M | H | Below | spot-check |
| Choose what to escalate | H | M | M | Below | spot-check |
| Propose a story batch (capped) | H | L | H | Below | none |
| Post an update / approve a company-wide one | L | H | L | Above | required |

## Agent anatomy (sketch)

- **Model:** gpt-4o-mini by default for drafting and read work; escalate to a frontier model after 2 critic rejections, rather than repeating the same weak reasoning a 3rd time.
- **Tools:** the fixture's set as-is — `get_project`, `get_activity`, `search_past_updates`, `propose_stories` (capped).
- **Memory:** roadmap and team norms persist across runs; everything else (drafts, activity pulls, this run's context) is purged.
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

1. **Pull project state + activity** sits below the line because it's easy to reverse, has a low blast radius, and is easy to verify — deciding factor: all three axes align cleanly.
2. **Decide relevant context** sits below the line for the same reason — easy to reverse, low blast radius, easy to verify — deciding factor: all three axes align cleanly.
3. **Draft the update** sits below the line because it's easy to reverse and has a low blast radius, but is only moderately easy to verify — deciding factor: measurability, which is why it gets a spot-check.
4. **Decide tone/commitment level** sits above the line because it's hard to reverse, has a high blast radius, and is hard to verify — deciding factor: blast radius, since a wrong commitment can't be walked back once leadership has read it.
5. **Flag at-risk/escalation** sits below the line because it's easy to reverse and easy to verify, but carries a moderate blast radius if a real risk goes unflagged — deciding factor: blast radius, which is why it gets a spot-check.
6. **Choose what to escalate** sits below the line because it's easy to reverse, but both blast radius and measurability are moderate — a missed escalation is invisible until it's already a problem — deciding factor: measurability, which is why it gets a spot-check.
7. **Propose a story batch (capped)** sits below the line because it's easy to reverse, has a low blast radius (nothing enters the real backlog until approved), and is easy to verify — deciding factor: all three axes align cleanly, reinforced by the hard cap.
8. **Post an update / approve a company-wide one** sits above the line because it's hard to reverse once it's visible to leadership, has a high blast radius, and is hard to verify in the moment — deciding factor: reversibility, since once it's out, it's out.

## Hardest call

Choosing what to escalate. Reversibility and blast radius both looked fine on their own, but measurability was the axis that settled it — a missed escalation is invisible until it's already a problem, since by definition nobody looked. That asymmetry (false positives are cheap to catch, false negatives aren't) is what earned it a spot-check instead of a clean below-the-line pass.
