# clawesome

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace.

## Plugins

- **[cut-the-complexity](plugins/cut-the-complexity/skills/cut-the-complexity/SKILL.md)** — Catch over-engineering, speculative generality, and catch-all framework designs in plans (from `/plan`, design docs, or another agent's proposal) before approving them. Distilled from Eugene Yan's *[Simplicity is An Advantage but Sadly Complexity Sells Better](https://eugeneyan.com/writing/simplicity/)*.

## Installation

Add the marketplace, then install the plugin from inside Claude Code:

```
/plugin marketplace add trakru/clawesome
/plugin install cut-the-complexity@clawesome
```

Once installed, invoke the skill with `/cut-the-complexity` or let it trigger when its description matches the situation (e.g., when reviewing a plan).

### Manual install (without the marketplace)

```bash
git clone https://github.com/trakru/clawesome.git
mkdir -p ~/.claude/skills
cp -r clawesome/plugins/cut-the-complexity/skills/cut-the-complexity ~/.claude/skills/
```

## License

MIT — see [LICENSE](LICENSE).
