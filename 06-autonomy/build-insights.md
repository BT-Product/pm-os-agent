# Build Insights: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 4, what you learned building it
>
> ✅ **What this validates:** you can reflect on what building it taught you — by the end you'll have proven the friction, the learning, and the aha that changes how you'd design your next agent.

## Friction

The missing-data escalation path was friction I didn't catch myself — it came up during review across M4, M5, and M6. When Cortex hits a project that doesn't exist, instead of escalating right away it searches across every other real project first, burning most or all of the iteration budget before giving up. It's still safe (nothing gets invented, it still escalates eventually) but it's inefficient — every one of those runs costs more and takes longer than it should, and if this ever hit real traffic instead of test fixtures, it would mean unnecessary escalations and wasted spend piling up specifically on exactly the kind of request (a typo'd or unknown project ID) that should be the cheapest, fastest case to handle.

## Learning

1. **Evals are much more than LLM-as-judge.** Before this course I thought "evals" meant grading a final answer with another model. What I actually built was broader — trajectory evals that grade the *path* (tool-call accuracy, path quality, recovery), a failure-mode register, bounds that trip in code, and a replay set of real runs to catch regressions. The final-answer check is one piece, not the whole picture.

2. **"Capability is not permission" is enforced in code, not prompts.** The jailbreak probe proved this concretely — a fully-committed prompt injection ("SYSTEM OVERRIDE," "pre-authorized by leadership") got nowhere not because the model was persuaded to refuse, but because the tools to post/close/commit simply don't exist. A bound a model could talk its way past isn't a real bound.

3. **Autonomy and the agent line are two different knobs.** The M1 agent line is the structural ceiling — what's physically possible for Cortex to do at all — and it's the same for every user no matter their Trust Ladder rung: posting, sending, or committing a date sits above that line for everyone, forever, because the tool to do it doesn't exist. The autonomy dial only changes how many below-the-line checkpoints still pause for a given person. So no matter the autonomy setting, Cortex will never send anything on its own — the dial just changes who has to click approve, not what Cortex is capable of doing unsupervised.

## Aha moment

The golden rule: if something isn't reversible, has a big blast radius, and can't be measured, it shouldn't be automated — it should always sit below the line, a human decision. This is the one idea I kept coming back to across the whole build, not just for the M1 agent line itself but for scoring the critic's checks, reasoning about JIT permissions, and setting the autonomy dial per segment in M6. It's a small rule, but it answers almost every "should this be automated" question I ran into.

## What you'd do differently

I'd build the missing-data fast-path from the start instead of noticing it three separate times (M4, M5, M6) and never actually fixing it. Right now, "the requested thing doesn't exist" falls under the general iteration cap — Cortex searches every other real project before giving up, which is safe but wasteful. If I designed the loop spec again, I'd make "data not found" its own explicit stop condition from day one: the first `project_not_found` on the primary lookup should escalate immediately, not after burning most of the iteration budget searching elsewhere. It's a small fix, but it's the difference between a bound that happens to catch a problem and a bound that was actually designed for it.
