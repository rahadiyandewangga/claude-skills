# Claude Skills

A collection of custom skills for [Claude Code](https://claude.ai/code) — structured frameworks that turn Claude into a specialist for a specific domain, invoked with a single slash command.

---

## What is a Claude Skill?

A skill is a markdown file that gives Claude a detailed, opinionated framework for a specific task. Instead of writing a long prompt every time, you type `/skill-name` and Claude follows the framework automatically — consistent output, every time.

Skills live in `~/.claude/skills/` and are picked up automatically by Claude Code.

---

## Available Skills

### `/us-stock-analysis` — US Stock Picking Framework

A 5-layer institutional-grade framework for analyzing US equities. Built for investors who want structured, repeatable analysis rather than gut-feel opinions.

**What it does:**

| Mode | How to trigger | What you get |
|------|---------------|--------------|
| Stock pick recommendation | `give me top 5 stocks to buy right now` | Screened picks with full conviction ranking |
| Single ticker | `analyze NVDA` | Full 5-layer deep dive with entry zone, stop, targets, R/R |
| Multi-ticker ranking | `compare NVDA, AVGO, AMD` | Ranked table by conviction × risk/reward, then full detail per stock |
| Portfolio review | `review my portfolio: NVDA @ $207, ORCL @ $192` | Hold/Trim/Add/Sell verdict per position + portfolio risk summary |

**The 5 layers:**

1. **Macro & Sector** — Fed posture, business cycle phase, VIX regime, sector rotation
2. **Fundamental** — EPS revision momentum, forward growth, valuation vs. peers, balance sheet (FCF, ROIC vs WACC, interest coverage), catalyst quality
3. **Technical** — MA structure, entry setups, risk/reward ≥ 2:1
4. **Smart Money** — 13F institutional flows, dark pool prints, options sweep activity, short interest trend, analyst consensus momentum
5. **Quantitative Overlay** — Fama-French 5-factor scoring to distinguish quality from value traps

**Example prompts:**

```
/us-stock-analysis give me top 5 stocks to buy right now
```
```
/us-stock-analysis give me 3 high-conviction stock picks for this market
```
```
/us-stock-analysis what are the best AI stocks to buy right now?
```
```
/us-stock-analysis what semiconductor stocks should I look at given the current macro?
```
```
/us-stock-analysis I have $5000 to deploy — what would you buy today and why?
```
```
/us-stock-analysis analyze NVDA
```
```
/us-stock-analysis compare ORCL, AVGO, ANET and rank them
```
```
/us-stock-analysis review my portfolio: NVDA @ $207, ORCL @ $192, KO @ $71, LLY @ $1120
```
```
/us-stock-analysis should I hold or sell CRM? I'm down 35% from $287
```

**What the output looks like:**

- Summary ranking table (sorted by Final Rank Score = conviction × R/R × risk modifier)
- Per-stock detail: thesis, catalyst, smart money signals, entry safe zone, stop loss, 6M price target, key risks
- Portfolio review: thesis status (Intact / Weakening / Broken), staged exit plan (PT1/PT2/Stop), portfolio-level risk summary with sector concentration and correlation flag

---

## Installation

**Clone the whole repo:**
```bash
git clone https://github.com/rahadiyandewangga/claude-skills.git ~/.claude/skills
```

**Or copy a single skill:**
```bash
mkdir -p ~/.claude/skills/us-stock-analysis
curl -o ~/.claude/skills/us-stock-analysis/SKILL.md \
  https://raw.githubusercontent.com/rahadiyandewangga/claude-skills/main/us-stock-analysis/SKILL.md
```

Claude Code picks up skills automatically — no restart needed.

---

## Requirements

- [Claude Code](https://claude.ai/code) (CLI, desktop, or VS Code extension)
- No API keys or extra dependencies — skills are pure markdown frameworks

---

## Adding your own skill

1. Create a folder: `~/.claude/skills/your-skill-name/`
2. Add a `SKILL.md` with a frontmatter name and description:

```markdown
---
name: your-skill-name
description: One line explaining when Claude should use this skill.
---

# Your framework here
```

3. Invoke it in Claude Code with `/your-skill-name`
