---
name: agentic-ai-evaluator
description: Evaluate an AI/ML/agent codebase against a production system-design rubric and write an EVALUATION.md report at the repo root. Use when the user asks to review, audit, critique, assess, or "see what's wrong with" an AI/LLM/agent/RAG repo, or asks how production-ready their AI solution is. Trigger on phrases like "review my agent", "audit this RAG pipeline", "is this production ready", "what would a staff engineer say about this", or whenever the user opens a clearly AI/agent project and asks for a code review.
---

# Agentic AI Evaluator

Read a repo, understand what it's trying to do, evaluate it against the rubric below, and write a single `EVALUATION.md` at the repo root.

## When to use

Trigger for substantive review of an AI codebase. Do **not** trigger for general (non-AI) software reviews, narrow code-quality questions ("is this Python idiomatic"), or one-shot doc help.

## Workflow

1. **Frame the problem.** Read `README.md` end-to-end (and any `docs/`, `ARCHITECTURE.md`, `DESIGN.md` at the root). State back, in one paragraph, what the solution is trying to do, who it's for, and what success looks like. If unclear, write your best inference and flag the ambiguity — don't block.

2. **Map the solution.** Get a structural read before judging. Enumerate top-level folders, then read load-bearing files: entry points, orchestration logic, prompt templates, tool definitions, retrieval code, eval harnesses, infra configs, tests. Read enough to answer: how does a request flow through this system? What models are called? What tools exist? Where is state kept? What is evaluated? Where are the safety checks? Capture as a short, neutral "Solution summary" — descriptive, no judgment yet.

3. **Evaluate against the rubric below.** For each dimension that applies to this repo, write three short subsections — **Successes**, **Failures and gaps**, **What we'd add or change**. Cite specific files. Distinguish actual failures (code does the wrong thing) from gaps (the dimension isn't addressed at all, which may be appropriate for the maturity stage). Skip dimensions that genuinely don't apply (e.g., training/alignment for a single-prompt summarizer); don't pad with N/A stubs.

4. **Synthesize and recommend.** After the per-section pass, write 2–5 cross-cutting observations (patterns spanning sections — usually the most valuable findings, because they describe the *shape* of what's wrong), 5–10 ranked recommendations (ranked by expected blast radius if left unaddressed), and a "What we'd learn from this design" section split into worth-copying patterns and failure modes for other teams to avoid.

5. **Write `EVALUATION.md`** at the repo root. Structure: problem framing → solution summary → per-dimension findings → cross-cutting observations → top recommendations → lessons. If `EVALUATION.md` already exists, ask before overwriting. After writing, give the user a 3–5 sentence chat summary highlighting headline findings and pointing at the file.

Use no numeric scores; qualitative prose only. Numbers invite debate and grade-grubbing; prose forces specificity.

## The rubric

Eight dimensions. For each: what good looks like, common failure modes, and signals to grep for in the code. Cite specific signals in the report — "no offline eval harness in `evals/` and no fixtures for the orchestration loop" beats "evaluation seems weak."

### 1. Business & ML objectives

**Good:** README names the user, the task in operational terms, success criteria, and the cost of being wrong. Acknowledges sequential decision-making under uncertainty (for agents). Risk-tiered framing: actions categorized by reversibility and blast radius; autonomy scales accordingly.

**Failures:** Vision pitch with no operational task definition ("our AI helps customers" — doing what?). Treats a multi-step problem as single-turn. Trust and reversibility unaddressed — irreversible actions taken with no distinction. Or the opposite: every action gated by human approval, system useless.

**Signals:** One-paragraph problem statement an outsider could understand? Named success criteria (completion rate, satisfaction, cost per task)? Stated non-goals? Any mention of action tiers, permissions, reversibility, human-in-the-loop?

### 2. High-level design

**Good:** Coherent orchestration architecture (ReAct loop or explicit state machine). Subsystems clearly separated: context assembly, tool registry/execution, guardrails, memory. Loop is interruptible, observable, bounded by step or token budget.

**Failures:** "Agent" is one prompt with tools bolted on. No clear loop — control flow buried in a megaprompt and emerges from model whim. Subsystems tangled — prompts, tool calls, retrieval, parsing interleaved in one function. No budget; tasks run unboundedly. Streaming and observability are vague aspirations.

**Signals:** Named orchestration component (`agent.py`, `loop.py`, `orchestrator.py`, `graph.py`)? Typed tool interface (function/JSON schemas, Pydantic)? Where do permission checks happen — centralized or scattered? How does the loop terminate? Single context-assembly site, or ad-hoc?

### 3. Data & context engineering

**Good:** Intentional trajectory-collection strategy (expert demonstrations, filtered synthetic, human feedback) rather than treating training data as something that just exists. At inference: deliberate context assembly with priority-based packing — recency vs. relevance vs. instructions. Story for what gets summarized, what gets truncated, what is non-negotiable (system instructions, safety guidelines, currently-relevant tool schemas).

**Failures:** "Stuff everything into the prompt" — context grows unboundedly, model ignores earlier instructions. Top-k retrieval bolted on with no relevance threshold. No separation between system instructions, tool outputs, and user content (prompt injection vector). Conversation history dropped wholesale or kept verbatim forever.

**Signals:** Single owner for prompt construction? Retrieval present — embedding model, fine-tuned or off-the-shelf? Relevance filtering or pure top-k? Tool outputs distinguishable from user input (separate roles, tagged messages)? Summarization/compression code? `trajectories/` or `data/sessions/` with scoring or filtering?

### 4. Memory & representation

**Good:** Explicit story across three horizons — within-session (what just happened), across-session for the same user/task (episodic), persistent factual or procedural knowledge (semantic, procedural). Tier-appropriate storage (vector DB with HNSW for semantic similarity, structured stores for filterable episodic, parameterized templates for procedural). A controller decides what's worth persisting; staleness handled (decay weights, re-validation, expiry).

**Failures:** Implicitly stateless, re-derives everything every turn. Or: everything persisted forever, vector store grows unboundedly, retrieval gets noisier, stale references to deleted resources accumulate. Memory writes unconditional. No distinction between "user prefers X" (worth persisting) and "user got an error 17 minutes ago" (ephemeral).

**Signals:** Persistent store beyond model context (vector DB, KV, SQL)? Memory controller, or unconditional writes? Expiry, decay, or re-validation? Episodic, semantic, procedural distinguished — or mashed together? Reusable workflow templates (procedural memory), or reasoning from scratch every time?

### 5. Pipeline: modeling, training & alignment

**Good:** Model choice justified — size, capability, latency-cost. Tool-use fine-tuning with differentiated loss weighting (tool tokens weighted higher than reasoning tokens), negative examples for schema adherence. Alignment via reward model trained on trajectory-level human preferences and applied with DPO or similar — not just SFT on positive examples. Multi-agent communication via typed schemas with confidence and citations, not free-form.

**Failures:** Frontier model used for every step regardless of difficulty. No fine-tuning despite obvious opportunities; or fine-tuning that's standard next-token prediction with no thought to tool-token weighting. Alignment is "we wrote a longer system prompt." Multi-agent uses free-form chat between agents — guarantees error amplification and hallucination cascades.

**Signals:** Which models are called? Routing layer that picks model size by step type, or always the same model? Fine-tuning artifacts (training scripts, dataset prep, weights, fine-tuned model IDs)? Negative-example sections in prompts ("not like this") or only positive? Multi-agent: typed schemas, confidence scores, citations? Any RLHF/DPO/preference data?

### 6. Infrastructure & serving

**Good:** Acknowledges agent inference is bursty and stateful, not synchronous request-response. Async/event-driven design — state serialized between steps, GPUs not held idle, tool execution decoupled from inference. Cost control first-class: model routing, per-task token budgets, summarization of older context. Tool execution sandboxed (containers with resource limits, no default network, ephemeral filesystems). Idempotency tracked — retries don't double-send. Elastic scaling via serialized state.

**Failures:** Whole agent loop in a single Python process holding a GPU for minutes while waiting on tool calls. Tools run with full system permissions; an LLM that hallucinates `rm -rf /` happily executes it. Retries duplicate side effects. No cost control — a single ambiguous task can burn the daily token budget. Concurrency is "we'll figure it out later." No distinction between hot and cold state.

**Signals:** Tool calls executed in a sandbox (Docker, gVisor, Firecracker, restricted shell) or raw `subprocess`? Token budgeting code, per-task cost tracking? Model router, multiple model tiers? State store between turns — in-process, Redis, DB? Idempotency logic (request IDs, dedupe keys) for side-effecting tools? How would this run 1,000 concurrent users — queues, async workers, capacity planning evidence?

### 7. Evaluation & metrics

**Good:** Hierarchical eval framework — task-level (did it complete?), trajectory-level (was the path efficient and recoverable?), behavior-level (communication, escalation, refusal). Online and offline both present. Non-determinism confronted: tasks run multiple times, seeds logged for replay, consistency tracked as a metric. A/B tests use stratified sampling because cross-task variance is enormous. Maintained benchmark suite catches regressions.

**Failures:** "We tried it and it worked." No held-out benchmark. Pass/fail tests on unit functions but no end-to-end trajectory tests. LLM-as-judge used naively, uncalibrated against human ratings. Completion measured but not trajectory efficiency or escalation rate — an agent that thrashes 50 steps and eventually succeeds looks identical to one that succeeds in 5. No online metrics — no instrumentation for cost, time, satisfaction. Non-determinism dismissed as "the model being weird that day."

**Signals:** `evals/`, `benchmarks/`, or `tests/integration/` with substantive content? Full agent-trajectory fixtures, or only single-turn? LLM-as-judge code calibrated against human ratings? Random seeds logged, sessions replayable? Per-task metric collection (token counts, latency, tool call counts) in production? Consistency measured across multiple runs of the same task? A/B or feature-flag scaffolding?

### 8. Robustness & deep dives

**Good:** Layered defenses against prompt injection — input sanitization, structured message formats with role boundaries, output validation against a deny-by-default action policy. Multi-agent failures contained — sub-agents return confidence and citations, orchestrator verifies high-impact claims, circuit breakers halt runaway loops. Capability boundaries explicit; structured human handoff when confidence drops. New users or task types start at lower autonomy and earn more. Tool parameters constrained (whitelisted file paths, bounded query complexity, restricted endpoints). Actions classified by reversibility; irreversible ones require explicit approval. Full decision trace stored as a structured graph that can be replayed and inspected.

**Failures:** Prompt injection treated as theoretical. Tool outputs concatenated raw into the prompt with no role separation. No permission system — tools either exist or don't, any agent can call any tool with any arguments. Failures cascade silently because sub-agents don't return confidence and the orchestrator trusts everything. No human-in-the-loop path; agent either succeeds autonomously or returns a useless error. Logs are ad-hoc print statements that can't reconstruct what happened. Adversarial inputs unconsidered.

**Signals:** Search for "injection", "sanitiz", "validate", "guardrail", "permission" — anything? What does it actually check? Tool outputs wrapped in a distinct message role (tool/function/observation), or concatenated as raw strings? Per-tool capability or permission declaration? Action classification (read-only/reversible/irreversible)? Multi-agent: structured results with confidence? Circuit breaker (max retries, steps, cost)? Human-handoff path and payload? Full traces queryable, decision DAG reconstructable?

## Tone

Write like a senior engineer reviewing a colleague's design — direct, concrete, charitable about intent, unflinching about gaps. Cite specific files and lines. Avoid hedging that drains the review of signal ("it might be nice to consider possibly adding..."). When something is good, say it's good and why; when something is broken, say it's broken and what breaks. The audience is the team using the report, not management — they want to know what to do next.
