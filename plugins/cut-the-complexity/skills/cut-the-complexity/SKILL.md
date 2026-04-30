---
name: cut-the-complexity
description: Use when reviewing an implementation plan from /plan, a design doc, or another agent's proposal before approving it - applies complexity-bias awareness to catch over-engineering, speculative generality, invented requirements, and catch-all framework designs dressed as "best practice."
---

# Cut the Complexity

## Overview

Plans look more credible when they have more parts. Multiple files, abstractions, "frameworks", configurable knobs, and future-proofing all signal effort and expertise. This is **complexity bias**, and it is the dominant failure mode of LLM-generated plans — including ones you wrote yourself in an earlier turn.

Your job as reviewer is the counterweight: assume the plan is more complex than it needs to be until proven otherwise.

> "Simplicity is a great virtue but it requires hard work to achieve it and education to appreciate it. And to make matters worse: complexity sells better." — Edsger Dijkstra
> "Everything should be made as simple as possible, but not simpler." — Einstein
> "The more simple any thing is, the less liable it is to be disordered, and the easier repaired when disordered." — Thomas Paine

## When to Use

- The user shows you a plan and asks "what do you think?"
- You just finished a `/plan` and are gating it before execution
- You're reviewing an RFC, design doc, or PR description that proposes an approach
- A subagent reports a planned approach and you need to approve it

Skip for: bug fixes, renames, single-line config changes — anything with no design surface.

## The Review Procedure

### Step 1: Anchor on the simplest plan, before reading theirs

Sketch — out loud — the simplest plan that satisfies the **stated** request. Literal pseudocode is fine.

Without this anchor, you'll judge the proposal against itself ("looks reasonable") instead of against alternatives. The anchor is the most important step; do not skip it.

### Step 2: Match solution surface to problem surface

A 5-line ask deserves a 5-line plan. A new endpoint deserves a handler, not a framework. Scope inflation is the #1 symptom of complexity bias and the easiest to spot once you have the anchor.

### Step 3: For each complexity unit, ask "is the juice worth the squeeze?"

Walk the plan. For every proposed new file, abstraction, dependency, configuration option, table, endpoint, queue, or test file:

1. What does it cost? (build, review, test, maintain, document, onboard others, deprecate later)
2. What concrete in-scope benefit does it buy?
3. **If removed, does the stated request still get satisfied?** If yes, it's a candidate for cutting.

### Step 4: Name the complexity smells

| Smell | What it sounds like in the plan |
|---|---|
| Speculative generality | "future-proof", "extensible", "foundation for", "sets us up for" |
| Catch-all framework | "registry", "strategy pattern", "supports both X and Y", "pluggable" |
| Cargo-culted best practice | Adds a layer because That's How It's Done, not because the problem demands it |
| Invented requirements | Auth, caching, metrics, retries, rate limits, idempotency the user didn't ask for *and that aren't satisfied more simply elsewhere* |
| Unjustified dependencies | Pulls in a library when the stdlib or 20 lines would do |
| Phase-2 smuggling | Implements what *might* be needed later in the same change |
| One-size-fits-all | Single generic system serving multiple use cases when 2–3 focused ones would total less code |

### Step 5: Address concerns at the simplest layer that solves them

Real concerns (idempotency, retries, observability, race conditions) often do apply. But they have a **simplest** form. Before approving a framework that addresses them, ask:

- Idempotency → is there a unique constraint or natural key that handles it?
- Retries → does the upstream system already retry? does the existing HTTP client?
- Observability → does the existing logger/tracer cover this path already?
- Concurrency → can the DB transaction or a single advisory lock handle it?

A plan that introduces a *system* to address a concern that a *line of code* could address is the bias in action.

### Step 6: Counter-bias check — don't reflexively prefer reuse either

The opposite trap: reaching for an existing heavyweight tool because it's familiar. Ask both:

- "Is there an existing simple thing we should use instead of building?"
- "Is the existing thing they reached for actually heavier than 30 lines of focused code?"

If a proposal pulls in a 50k-LOC dependency to use 1% of it, that's also complexity bias.

### Step 7: Prefer multiple focused solutions over one catch-all

If the plan proposes a single generic system handling several use cases, check whether 2–3 single-purpose solutions would total less code. Catch-all systems couple all their users together and become impossible to deprecate; focused systems can be deleted independently.

## Output Format

Structure your review as:

1. **Verdict** — approve / request changes / send back
2. **The simplest plan** — your anchor from Step 1, concretely
3. **Cut list** — each complexity unit to remove, with one-line justification
4. **Keep list** — what's load-bearing, so the author knows what survived
5. **Top concern in one sentence**

## Reviewer Failure Modes

| Failure | Counter |
|---|---|
| "Looks thorough, ship it" | Thorough ≠ correct. Compare to the simplest plan first. |
| "They thought about future case X — that's good" | Future-proofing **is** the bias. Demand evidence the case will actually happen within the relevant horizon. |
| "I don't want to be pedantic about a few extra files" | Each extra file is real ongoing cost. Be pedantic; that's the job. |
| "Maybe they know something I don't" | Ask. Do not approve on assumed expertise. |
| "Removing the abstraction means rewriting later" | Usually false. Adding a simple thing later when the need is real is cheap. Carrying speculative complexity now is not. |
| "It's well-factored, even if large" | Good factoring of unnecessary code is still unnecessary code. |

## Red Flags — Default to "Request Changes"

- Plan is >2× the size of your Step 1 anchor
- Any abstraction whose only justification is "future use"
- Any requirement not in the original ask **and** not load-bearing for the ask
- Any heavy dependency added without considering a small inline alternative
- Effort estimate is >3× what the simplest plan would take
- Phrases like "production-grade", "extensible framework", "foundation for" without a concrete near-term use case attached

When you spot any of these, request changes and **propose the simpler version concretely** — don't just say "simplify."

## Source

Distilled from Eugene Yan, *Simplicity is An Advantage but Sadly Complexity Sells Better* (Aug 2022). https://eugeneyan.com/writing/simplicity/
