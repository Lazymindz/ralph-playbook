# X Article Draft - A Ralph Playbook

*This picks up after your intro (the "nah" setup, diagram, and link to full details). These are the key callouts to weave in.*

---

## Three Phases, Two Prompts, One Loop

The diagram clarified something for me: Ralph isn't just "a loop that codes." It's a funnel.

**Requirements → Planning → Building**

Each phase has a different job:

**Requirements** (human + LLM conversation)
- Discuss the idea, identify Jobs to Be Done
- Break JTBDs into topics of concern
- Write specs for each topic (`specs/*.md`)

This happens *before* the loop runs. You're shaping what Ralph will build.

**Planning** (loop with PROMPT_plan.md)
- Ralph studies specs and existing code
- Does gap analysis: what's missing?
- Outputs a prioritized task list (`IMPLEMENTATION_PLAN.md`)
- No implementation, no commits

**Building** (loop with PROMPT_build.md)
- Ralph picks the most important task from the plan
- Implements with subagents, runs tests
- Updates plan, commits, exits
- Loop restarts with fresh context

Same loop mechanism. Different prompts. Swap files to change modes.

The insight: *planning is cheap*. If Ralph goes in circles during building, throw out the plan and regenerate. One planning loop costs nothing compared to wasted building iterations.

---

## Context Is Everything

This might be the most underappreciated part.

- 200K context advertised → ~176K actually usable
- **40-60% utilization is the "smart zone"** — beyond this, quality degrades
- One task per loop = staying in the smart zone

The architecture follows from this constraint:

**Use the main agent as a scheduler, not a worker.** Don't allocate expensive work to the primary context. Spawn subagents instead — each one gets ~156K of context that gets garbage collected when done.

Fan out to avoid polluting the main thread.

Model selection matters too:
- **Opus** for the primary agent (task selection, prioritization need strong reasoning)
- **Sonnet** for most subagents (searches, summaries, file operations)
- **Opus subagents** for complex reasoning (debugging, architectural decisions)

Efficacy first, cost second. Opus on primary prevents wasted loops.

---

## Steering Ralph: Patterns + Backpressure

Ralph's output is steered from two directions.

**Upstream: Your stdlib and existing patterns.** Geoff: "The code that Ralph generates is within your complete control through your technical standard library and your specifications." If Ralph is generating the wrong patterns, update your stdlib to steer it toward correct ones. The existing code shapes what gets generated.

**Downstream: Backpressure.** This is what rejects invalid work after it's generated. Wire in whatever validates your code:
- Tests
- Type checks
- Lints
- Build verification

The prompt says "run tests" generically. Your `AGENTS.md` file specifies the actual commands. This is how backpressure gets project-specific.

Without backpressure, Ralph will "complete" tasks that don't actually work.

Both matter. Patterns guide generation upstream. Backpressure catches failures downstream.

**And backpressure can extend beyond code validation.** Some acceptance criteria resist programmatic checks — creative quality, aesthetics, UX feel. LLM-as-judge reviews can provide backpressure for subjective criteria too. Binary pass/fail that runs until it passes.

More on that in the [full playbook](https://github.com/claytonfarr/ralph-playbook#non-deterministic-backpressure).

---

## Let Ralph Ralph

This is where Ralph asks something of you: **trust**.

Geoff: "You need to trust Ralph to decide what's the most important thing to implement. This is full hands-off vibe coding that will test the bounds of what you consider 'responsible engineering'."

The prompt doesn't prescribe *which* task to do — it says "choose the most important thing." Ralph decides. And it turns out LLMs are surprisingly good at reasoning about priority and next steps when given specs, a plan, and the codebase to study.

Ralph can also:
- Update its own implementation plan as it learns
- Discover bugs and document them for future loops
- Learn operational patterns and update AGENTS.md
- Even create new specs if functionality is missing

The circularity is intentional. **Eventual consistency through iteration.**

Geoff: "Building software with Ralph requires a great deal of faith and a belief in eventual consistency. Ralph will test you." Any problem created by AI can be resolved through a different series of prompts — more loops, different angles.

**The plan is disposable.** Geoff: "I have deleted the TODO list multiple times." If Ralph is going off track, throw it out and regenerate. That's the escape hatch. Regeneration is cheap.

**Tune like a guitar.** Instead of prescribing everything upfront, you observe and adjust reactively. When Ralph fails a specific way, add a sign.

But signs aren't just prompt text. They're anything Ralph can discover:
- **Prompt guardrails** — explicit instructions like "don't assume not implemented"
- **AGENTS.md** — operational learnings about how to build/test
- **Utilities in your codebase** — when you add a pattern to stdlib, Ralph discovers it and follows it

The code itself is a sign. This connects back to upstream steering — your stdlib shapes generation because Ralph studies it and mimics those patterns.

Geoff's critical prompt guardrail: *"Before making changes search codebase (don't assume an item is not implemented) using parallel subagents."*

If you wake up to find Ralph duplicating work, tune this instruction. This nondeterminism is Ralph's Achilles' heel — and the primary thing you'll iterate on.

The mindset shift: You're not controlling Ralph. You're setting constraints, observing behavior, and tuning.

---

## Outside the Loop

You've moved out of the loop. Ralph is doing the work. But you're not gone — you're *on* the loop.

**Your job now is observation and course correction.**

Especially early on, sit and watch. What patterns emerge? Where does Ralph go wrong? What signs does he need? The prompts you start with won't be the prompts you end with — they evolve through observed failure patterns.

This requires visibility into what Ralph is doing:
- Stream output so you can watch in real-time
- Review commits and plan updates between loops
- Check the implementation plan for drift or circular work

When something goes wrong, you have escape hatches:
- `Ctrl+C` stops the loop
- `git reset --hard` reverts uncommitted changes
- Regenerate the plan if trajectory goes off track

And remember: **use protection.**

Ralph requires `--dangerously-skip-permissions` to operate autonomously. This bypasses all permission prompts — so a sandbox becomes your only security boundary.

Philosophy: **"It's not if it gets popped, it's when. What's the blast radius?"**

Running without isolation exposes credentials, browser cookies, SSH keys, and access tokens on your machine. Run in isolated environments with minimum viable access:
- Only the API keys and deploy keys needed for the task
- No access to private data beyond requirements
- Restrict network connectivity where possible

Options: Docker sandboxes locally, E2B/Modal for remote/production.

You're outside the loop, but you're still the engineer. Ralph does the work. You observe, protect, and tune.

---

## Enhancements Worth Exploring

I've been noodling on several extensions to the core approach. Still determining viability, but the opportunities seem promising:

**[AskUserQuestion for Requirements](https://github.com/claytonfarr/ralph-playbook#use-claudes-askuserquestiontool-for-planning)** — Use Claude's built-in interview tool to systematically clarify JTBD, edge cases, and acceptance criteria before writing specs.

**[Acceptance-Driven Backpressure](https://github.com/claytonfarr/ralph-playbook#acceptance-driven-backpressure)** — Derive test requirements during planning from acceptance criteria. Prevents "cheating" — can't claim done without required tests passing.

**[Non-Deterministic Backpressure](https://github.com/claytonfarr/ralph-playbook#non-deterministic-backpressure)** — LLM-as-judge for subjective criteria (tone, aesthetics, UX). Binary pass/fail reviews that iterate until pass.

**[Work Branches](https://github.com/claytonfarr/ralph-playbook#work-branches)** — If you want parallel workstreams, scope at plan creation (deterministic), not task selection (non-deterministic). Asking Ralph to "filter to feature X" at runtime is unreliable. Instead: create a scoped plan per branch upfront, then Ralph picks "most important" from an already-focused plan.

**[JTBD → Story Map → SLC](https://github.com/claytonfarr/ralph-playbook#jtbd--story-map--slc-release)** — Reframe topics as user activities, map into a journey, slice horizontally for Simple/Lovable/Complete releases.

Each builds on the core philosophy — not replacing it.

---

## Full Details

The complete playbook with code samples, prompt templates, and deeper dives on each enhancement:

**[github.com/claytonfarr/ralph-playbook](https://github.com/claytonfarr/ralph-playbook)**

---

*Credit to [@GeoffreyHuntley](https://x.com/GeoffreyHuntley) for creating Ralph and sharing his process openly. This is my attempt to distill and extend his work — any errors are mine.*
