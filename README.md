# clawesome

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace.

## Plugins

- **[cut-the-complexity](plugins/cut-the-complexity/skills/cut-the-complexity/SKILL.md)** — Catch over-engineering, speculative generality, and catch-all framework designs in plans (from `/plan`, design docs, or another agent's proposal) before approving them. Distilled from Eugene Yan's *[Simplicity is An Advantage but Sadly Complexity Sells Better](https://eugeneyan.com/writing/simplicity/)*.
- **[agentic-ai-evaluator](plugins/agentic-ai-evaluator/skills/agentic-ai-evaluator/SKILL.md)** — Evaluate an AI, LLM, RAG, or agent codebase against a production system-design rubric (8 dimensions: business framing, orchestration, context, memory, training/alignment, infra, evaluation, robustness) and write a structured `EVALUATION.md` report at the repo root.

## Installation

Add the marketplace, then install a plugin from inside Claude Code:

```
/plugin marketplace add trakru/clawesome
/plugin install cut-the-complexity@clawesome
/plugin install agentic-ai-evaluator@clawesome
```

Once installed, invoke a skill with its slash command (e.g. `/cut-the-complexity`) or let it trigger when its description matches the situation.

### Manual install (without the marketplace)

```bash
git clone https://github.com/trakru/clawesome.git
mkdir -p ~/.claude/skills
cp -r clawesome/plugins/cut-the-complexity/skills/cut-the-complexity ~/.claude/skills/
cp -r clawesome/plugins/agentic-ai-evaluator/skills/agentic-ai-evaluator ~/.claude/skills/
```

## License

MIT — see [LICENSE](LICENSE).
