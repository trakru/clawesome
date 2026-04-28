# clawesome

A small collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

## Skills

- **[cut-the-complexity](skills/cut-the-complexity/SKILL.md)** — Catch over-engineering, speculative generality, and catch-all framework designs in plans (from `/plan`, design docs, or another agent's proposal) before approving them. Distilled from Eugene Yan's *[Simplicity is An Advantage but Sadly Complexity Sells Better](https://eugeneyan.com/writing/simplicity/)*.

## Installation

Drop a skill into your personal Claude Code skills directory:

```bash
git clone https://github.com/trakru/clawesome.git
mkdir -p ~/.claude/skills
cp -r clawesome/skills/cut-the-complexity ~/.claude/skills/
```

Claude Code will pick up the skill automatically. Invoke it explicitly with `/cut-the-complexity` or let it trigger when its description matches the situation (e.g., when reviewing a plan).

## License

MIT — see [LICENSE](LICENSE).
