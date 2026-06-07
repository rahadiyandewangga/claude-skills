# Claude Skills

Personal Claude Code skills — reusable prompting frameworks invoked via `/skill-name` in Claude Code.

## Skills

| Skill | Description |
|-------|-------------|
| [us-stock-analysis](./us-stock-analysis/SKILL.md) | 5-layer US stock picking framework: macro, fundamental, technical, smart money, quantitative overlay. Supports single ticker analysis, multi-ticker ranking, and portfolio review. |

## How to install

Copy any skill folder into your `~/.claude/skills/` directory. Claude Code will pick it up automatically.

```bash
cp -r us-stock-analysis ~/.claude/skills/
```
