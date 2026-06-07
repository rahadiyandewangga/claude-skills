---
name: us-stock-analysis
description: Performs structured US stock picking analysis using a 5-layer signal framework (macro, fundamental, technical, smart money, quantitative overlay) plus supplementary factors. Use when the user asks to analyze stocks, suggest stock picks, check a ticker's potential, evaluate a specific stock, compare multiple stocks, perform stock screening, check institutional activity, review a portfolio, check if they should hold or sell, provide portfolio hold/sell/add decisions, assess average cost positions, or apply a stock picking framework for US equities.
---

# US Stock Analysis Framework

## Ticker Analysis Workflow
Use this when the user provides specific ticker(s) to evaluate.

### Pre-Step: Stock Type Classification
Before fetching data, identify the Lynch type — it determines which criteria matter most:

| Type | Signal | Key metric (replaces generic) |
|------|--------|-------------------------------|
| Fast Grower | EPS growth > 20% YoY, small/mid cap | PEG < 1.5 is primary |
| Stalwart | Large cap, EPS 10–15% YoY, stable business | EV/EBITDA vs. peers, FCF yield |
| Cyclical | Earnings tied to cycle (autos, steel, airlines) | Buy on early-cycle upturn; skip the "EPS > 15%" criterion |
| Turnaround | Recovering from distress, ROIC improving | ROIC trajectory + debt paydown; skip PEG |
| Asset Play | Undervalued hard assets vs. market cap | P/B, NAV discount |

State the type in one word before scoring. For Cyclicals and Turnarounds, skip or de-weight EPS growth rate as the primary criterion.

### Step 1: Fetch Current Data
**Run all Core searches simultaneously (parallel, not sequential). For multiple tickers, run Core searches for ALL tickers before scoring any.**

**Core — always run (2 searches cover ~80% of what you need):**
- `"[TICKER] finviz"` → price, MA position, Forward P/E, PEG, short float %, 52W range, analyst target, RSI, EPS next Y, sector
- `"[TICKER] EPS revisions analyst upgrade estimate 2026 zacks OR tipranks"` → revision trend, beat history, consensus PT, recent upgrades/downgrades

**Smart money — run only if stock clears the Fast-Fail Gate below:**
- `"[TICKER] insider buying 13F institutional unusual options dark pool 2026"` → covers 13F, options flow, and dark pool in one search

**Earnings — run only if next earnings is within 3 weeks:**
- `"[TICKER] earnings preview implied move whisper estimate 2026"` → beat probability, options-implied move vs. historical average move

**Fast-Fail Gate:** After Core searches, if ALL three of the following are true → score as Watch/Pass immediately, skip the Smart Money search:
- Price is below the 200-day MA
- Short interest is rising (not falling)
- EPS estimates are being cut (not raised)

### Step 2: Score Each Ticker
Apply the Decision Checklist (see below). Count how many boxes are checked:

- **12-14 boxes checked** → Strong Buy candidate, high conviction
- **9-11 boxes checked** → Buy candidate, medium conviction
- **6-8 boxes checked** → Watch list only, wait for more signals
- **< 6 boxes checked** → Pass, do not enter

### Step 3: Compare & Rank (Multiple Tickers)
When given 2+ tickers, after scoring each individually, **always rank by Rank Score descending** using this formula:

```
Conviction Score  = (Checklist / 14) × 10          → range 0–10
Rank Score        = Conviction Score × R/R ratio
Risk Modifier     = if High Risk → multiply Rank Score × 0.80
                    if Med Risk  → multiply Rank Score × 0.90
                    if Low Risk  → no change
Final Rank Score  = Rank Score × Risk Modifier      → used for ordering
```

**Risk level definitions:**
- **Low** — FCF positive, Net Debt/EBITDA < 1x, stop loss ≤ 8%, sector non-cyclical or macro tailwind confirmed
- **Medium** — FCF positive but debt elevated, or stop loss 8–12%, or sector has mixed macro signals
- **High** — FCF negative, high debt, stop loss > 12%, or pre-earnings position, or speculative catalyst

Always produce this ranking table (sorted by Final Rank Score, highest first):

| Rank | Ticker | Sector | Conviction | Risk | Upside | R/R | Rank Score | Verdict |
|------|--------|--------|------------|------|--------|-----|------------|---------|
| 1 | TICK1 | X | 11/14 (7.9) | Low | +28% | 3.2x | **25.2** | Strong Buy |
| 2 | TICK2 | X | 9/14 (6.4) | Med | +22% | 2.5x | **14.4** | Buy |
| 3 | TICK3 | X | 7/14 (5.0) | High | +35% | 2.1x | **8.4** | Watch |

Column definitions:
- **Conviction** — checklist score (X/14) and normalized score in parentheses (X/10)
- **Risk** — Low / Medium / High using definitions above
- **Upside** — % from entry zone midpoint to 6M price target
- **R/R** — (Target − Entry) / (Entry − Stop)
- **Rank Score** — Conviction Score × R/R × Risk Modifier (the ordering number)
- **Verdict** — Strong Buy (12–14) / Buy (9–11) / Watch (6–8) / Pass (<6)

Then explain the top pick vs. the runner-up in 2–3 sentences: what separates them and why the rank is what it is.

### Step 4: Output Full Detail for Each Ticker
Use the Per-Stock Detail Block format (see Output Format section). Always open with the observation date and current price at the time of analysis.

---

## Portfolio Review Workflow
Use this when the user provides a list of positions with average costs and wants to know whether to hold, add, trim, or sell each one.

### Input Format
Accept positions in any of these formats:
- `TICKER @ $avg_cost`
- `TICKER: avg_cost`
- `TICKER - avg_cost`

Example: `ASTS @ $72.50, MU @ $724.66, CRM @ $271.75`

If the user provides position sizes or share counts, factor them into the portfolio-level risk summary.

### Step 1: Fetch Current Data for Each Position
For each ticker, use web search to retrieve:
- Current price → calculate unrealized P&L % and direction
- 50-day and 200-day MA (is price above or below?)
- Key recent news or earnings since the position was likely opened
- Thesis status signals: any deteriorating fundamentals, guidance cuts, or competitive threats

### Step 2: Assess Thesis Status
For each position assign one of three states:

| Status | Meaning |
|--------|---------|
| Intact | Core investment thesis unchanged. Revenue/earnings on track, no new structural threats. |
| Weakening | 1-2 thesis pillars are showing cracks (e.g., guidance cut, margin compression, key competitor gaining). Proceed with caution. |
| Broken | The original reason to own the stock no longer holds. Exit is the correct response regardless of P&L. |

### Step 3: Apply the Decision Matrix
Determine the verdict for each position:

| Condition | Verdict |
|-----------|---------|
| Thesis Intact + current price ≤ entry zone + R/R ≥ 2:1 | Add — good opportunity to build position |
| Thesis Intact + price between avg cost and PT1 | Hold — let the thesis play out |
| Thesis Intact + price at or above PT1 | Trim — sell 25-33%, hold rest |
| Thesis Intact + price at or above PT2 | Trim More — sell another 25-33%, trail remainder |
| Thesis Weakening | Reduce — cut to half size, reassess next earnings |
| Thesis Broken OR price below stop loss level | Sell — full exit, do not wait |

### Step 4: Iron Rules (Never Override These)
**Stop loss rule:** If the current price is already below the framework stop level, the sell signal is triggered. Never move the stop lower just to avoid locking in a loss. The stop exists to protect capital.

**No averaging down rule:** Only consider adding to a losing position if ALL of the following are true:
- Thesis is fully Intact (not just "probably fine")
- Current price is at a legitimate technical entry zone (50-day MA, base support)
- R/R is ≥ 2:1 from current price
- The loss is due to market noise, not fundamental deterioration

If any condition fails → Hold or Sell, never add.

**Profit-taking rule:** Use staged exits, not all-or-nothing. Selling in tranches locks in gains while keeping exposure to further upside.

---

## Framework Overview
Analyze stocks through 5 layers in sequence — each layer narrows the universe and adds rigor:

1. **Macro/Sector** → narrows from full market to 2-4 favored sectors
2. **Fundamental** → filters for quality, earnings momentum, and catalysts
3. **Technical** → optimizes entry timing and defines risk/reward
4. **Smart Money** → confirms large capital is aligned with your thesis
5. **Quantitative Overlay** → validates thesis with factor scoring, risk metrics, and position sizing (optional but recommended for portfolio construction)

Layers 1-4 are table-stakes (non-negotiable). Layer 5 is optional for quick analysis but mandatory for portfolio construction (>5 positions), high-conviction sizing (>8%), and pre-earnings positions. Supplementary factors add conviction on top.

---

## Layer 1: Macro & Sector Filter
Check these before looking at individual stocks:

- **Rate cycle:** Fed posture → cutting favors tech/growth/REITs; hiking favors financials/energy/value
- **Business cycle phase:** Use ISM Manufacturing PMI, ISM Services PMI, LEI, jobless claims to identify recovery/expansion/peak/contraction
- **Sector rotation:** Map current cycle phase to favored sectors (recovery → financials/tech; expansion → industrials/energy; peak → staples/healthcare; contraction → utilities/staples)
- **Risk regime:** VIX < 20 = risk-on; VIX 20-30 = quality bias; VIX > 30 = defensive or cash; watch credit spread direction (HY vs IG)
- **EPS revision trend:** If broad analyst upgrades are happening across S&P 500, be more aggressive; if broad cuts, be selective

**Output of Layer 1:** 2-4 favored sectors, ~100-200 stock universe.

---

## Layer 2: Fundamental Filter
Score each stock 1-5 on:

| Criterion | Weight | Check |
|-----------|--------|-------|
| EPS revision momentum | 30% | Estimates raised in last 60 days? |
| EPS growth (forward) | 25% | Projected > 15% YoY? |
| Valuation vs. peers | 20% | PEG < 1.5 or EV/EBITDA below sector median? |
| Balance sheet | 15% | FCF positive, Net Debt/EBITDA < 2x, EBIT/Interest > 3x, ROIC > WACC? |
| Catalyst quality | 10% | Specific, near-term, high-conviction event? |

Catalyst types to look for: earnings beat setup, product launch, margin expansion, M&A, index inclusion, analyst upgrade cycle, regulatory approval.

**Earnings quality check:**
- Revenue-driven beats > cost-cut-driven beats
- FCF ≥ Net Income preferred (divergence = accounting risk)
- CFO/Net Income > 0.8 — if Net Income grows but operating cash flow doesn't, flag as accounting risk
- For SaaS: deferred revenue growing = future revenue locked in

**Output of Layer 2:** 10-20 stock short list with clear catalysts.

---

## Layer 3: Technical Filter
Only enter if ALL of the following are true:
- Price > 50-day and 200-day MA
- 50-day MA > 200-day MA (golden cross structure)
- A clean entry setup exists (see below)
- Risk/reward ratio ≥ 2:1

**Entry setups (pick one):**
- Breakout above key resistance on volume > 150% of 20-day average
- Pullback to 21-day or 50-day EMA holding with a bullish candle
- RS line making new highs before price (leading indicator)
- Cup & handle base completion

**Define before entering:**
```
Entry price: [technical trigger level]
Stop loss:   [below key support, typically -7% to -10% from entry]
Target:      [prior resistance or fundamental-based]
R/R ratio:   (Target - Entry) / (Entry - Stop) ≥ 2.0
```

---

## Layer 4: Smart Money / Market Mover Filter
Large institutional players — hedge funds, pension funds, sovereign wealth funds — move hundreds of millions at once. Their positioning leaves footprints in public data. This layer checks whether the big money is flowing with your thesis or against it.

**Rule:** At least 2 of the 5 signals below must be positive to pass this layer.

### Signal 1: Institutional Accumulation (13F Filings)
Hedge funds and institutions with >$100M AUM file 13F reports quarterly (45-day lag). Look for:
- **New positions:** A high-conviction fund initiated a new stake → bullish
- **Increased stake:** Known fund meaningfully increased an existing position (>20% increase) → bullish
- **Broad accumulation:** Multiple unrelated institutions buying simultaneously → very bullish
- **Concentrated selling:** Multiple funds exiting the same quarter → warning sign

Tools: WhaleWisdom, Dataroma, SEC EDGAR 13F search

Search: `"[TICKER] 13F institutional ownership Q1 2026"`
Search: `"[TICKER] hedge fund holdings increase 2026"`

### Signal 2: Dark Pool & Block Trade Activity
Dark pools are private exchanges where large institutions execute orders anonymously to avoid moving the public market. High dark pool volume signals quiet accumulation or distribution before a price move.

- **Dark pool buy prints > 40% of total volume:** Institutions accumulating off-exchange → bullish
- **Large block trades at ask price:** Buyers are willing to pay the ask = urgency = bullish
- **Block trades at bid price:** Sellers are offloading urgently = bearish

Tools: Unusual Whales (dark pool feed), Finviz dark pool, Quant Data

Signal to look for: "Dark pool volume spike on [TICKER]"
Bullish flag: Repeated dark pool prints at or above ask, clustered over 2-5 days

### Signal 3: Smart Options Flow (Informed Capital)
Not all options activity is equal. Smart money leaves specific fingerprints:

| Options Signal | Description | Interpretation |
|---------------|-------------|----------------|
| Sweep orders | Single large order filled aggressively across multiple exchanges | Urgency = informed buyer |
| OTM calls, short expiry, large premium | Out-of-money calls expiring soon, bought in size | Bet on near-term catalyst |
| Call/Put ratio skew | Calls bought vs. puts ratio well above stock's norm | Bullish sentiment shift |
| IV divergence | IV rising while stock is flat or down | Someone buying protection or positioning for a move |
| Rolling positions | Large calls being rolled to higher strikes | Original bull thesis extended |

Tools: Unusual Whales, Barchart unusual activity, Market Chameleon

Search: `"[TICKER] unusual options activity sweep 2026"`
Bullish flag: Sweep orders on calls, OTM, 30-60 day expiry, premium > $500K

### Signal 4: Short Interest Trend (Direction Matters More Than Level)
The level of short interest matters, but the direction is the key signal:
- Short interest declining over the last 2-3 reporting periods → Bears are giving up, covering → bullish
- Short interest rising while stock is also rising → shorts are fighting the trend = squeeze fuel
- Short interest declining while stock rises = most bullish combination (shorts being forced out)
- Short interest rising while stock falls = bears are pressing and winning → bearish confirmation

Search: `"[TICKER] short interest trend May 2026"`
Bullish flag: Short interest as % of float declining 2+ consecutive reporting periods

### Signal 5: Analyst Consensus Momentum
Analyst upgrades and price target increases cause institutional clients to reposition:
- Multiple upgrades in 30 days from different major banks → institutional clients are being moved long
- Consensus price target raised above current price by >15% → broad analyst community sees upside
- Initiation of coverage with Buy → new institutional audience exposed to the stock (demand catalyst)
- Single downgrade amid multiple upgrades → lone bear, usually not enough to stop a move

Search: `"[TICKER] analyst upgrade price target 2026"`
Bullish flag: 2+ upgrades in last 30 days; consensus target ≥ 15% above current price

### Smart Money Scoring

| Score | Interpretation |
|-------|---------------|
| 4-5 signals positive | Strong institutional tailwind — high conviction, consider full size |
| 2-3 signals positive | Passes the layer — institutions broadly aligned |
| 1 signal positive | Marginal pass — proceed with half-size until more confirmation |
| 0 signals positive | Fails Layer 4 — smart money is not aligned, wait or pass |

**Output of Layer 4:** Confirmation that institutional capital is flowing with your thesis, or a warning to wait.

---

## Layer 5: Quantitative Overlay
**Purpose:** Layer 5 adds quantitative rigor to validate your thesis, optimize position sizing, and prevent common mistakes (buying at volatility extremes, hidden correlation risk, value traps). This layer uses academic factor models and risk-adjusted metrics that elite hedge funds rely on.

**When to use:**
- **Mandatory:** Portfolio construction (>5 positions), high-conviction sizing (>8%), pre-earnings positions
- **Optional:** Quick single-stock analysis (but recommended for learning)
- **Progressive enhancement:** Start with Section 5.1 (Factor Scoring), add others as you get comfortable

Time investment: 10-15 minutes per stock with free tools.

---

### 5.1 Factor Exposure Scoring (Fama-French 5-Factor Model)
The Fama-French model identifies factors that explain 90%+ of long-term stock returns. By scoring a stock's exposure to these factors, you can distinguish quality cheap stocks from value traps, and momentum with quality from momentum bubbles.

**The 5 Factors:**

| Factor | What It Measures | Positive Signal | Negative Signal | Academic Evidence |
|--------|-----------------|-----------------|-----------------|-------------------|
| Value (HML) | High book/market ratio | P/B < sector median, P/E < sector median | P/B or P/E > 2x sector median | Value outperforms growth over 30+ years |
| Size (SMB) | Small cap vs. large cap | Market cap < $10B | Market cap > $500B | Small caps outperform large caps historically |
| Profitability (RMW) | Operating profit margin | Gross Profit / Assets > 0.35 | Gross Profit / Assets < 0.20 | Profitable firms outperform unprofitable |
| Investment (CMA) | Asset growth discipline | Asset growth < 10% YoY | Asset growth > 25% YoY (overinvesting) | Conservative firms outperform aggressive |
| Momentum (UMD) | 12-month price momentum | 12-month return > +15% | 12-month return < 0% | Winners continue winning for 6-12 months |

**Scoring System:**

For each factor, assign:
- **+2:** Strong positive exposure (top quartile)
- **+1:** Moderate positive exposure (above median)
- **0:** Neutral exposure (at median)
- **-1:** Moderate negative exposure (below median)
- **-2:** Strong negative exposure (bottom quartile)

**Total Factor Score:** Sum of all 5 factors = -10 to +10

**Interpretation:**

| Total Score | Meaning | Implication |
|-------------|---------|-------------|
| +7 to +10 | Multi-factor tailwind | Highest conviction — quality stock at reasonable price with momentum |
| +4 to +6 | Balanced positive | Solid setup — thesis supported by multiple factors |
| +1 to +3 | Slight positive | Marginal edge — requires strong Layer 2-4 signals to proceed |
| -3 to 0 | Neutral to slight negative | Caution — likely a single-factor bet (e.g., only momentum, no quality) |
| -10 to -4 | Multi-factor headwind | High risk — value trap, overvalued, or negative momentum |

**How to Calculate (Free Data Sources):**

Use Finviz.com (free screener) and Yahoo Finance to get these inputs:

| Factor | Data Points Needed | How to Calculate | Interpretation |
|--------|--------------------|-----------------|----------------|
| Value | P/B, P/E, Sector P/B, Sector P/E | Each ratio scored separately: ≥20% below sector median → +1; ≥20% above sector median → -1; within ±20% of sector median → 0. Total Value = P/E score + P/B score (range: -2 to +2) | +2 = both metrics cheap vs. peers / -2 = both expensive vs. peers |
| Size | Market Cap | < $2B → +2 / $2B-$10B → +1 / $10B-$200B → 0 / $200B-$500B → -1 / > $500B → -2 | Smaller caps have higher expected returns but more volatility |
| Profitability | Gross Profit, Total Assets | Gross Profit / Assets ratio: > 0.40 → +2 / 0.30-0.40 → +1 / 0.20-0.30 → 0 / < 0.20 → -1 to -2 | Higher ratio = more efficient profit generation |
| Investment | Total Assets YoY growth | Asset growth (YoY): < 5% → +2 / 5-10% → +1 / 10-20% → 0 / 20-30% → -1 / > 30% → -2 | Disciplined asset growth > aggressive empire-building |
| Momentum | 12-month stock return | 12-month return: > +30% → +2 / +15% to +30% → +1 / 0% to +15% → 0 / -15% to 0% → -1 / < -15% → -2 | Positive momentum tends to persist 6-12 months |

**Step-by-Step Calculation Example:**

**Stock: NVDA** (as of a sample observation date)

- *Value Factor:*
  - NVDA P/E: 45x | Sector (Semiconductors) P/E: 25x → -1 (expensive vs. peers)
  - NVDA P/B: 50x | Sector P/B: 8x → -1 (expensive)
  - Value score: **-2** (overvalued on traditional metrics)
- *Size Factor:* Market cap: $2.8T → **-2** (mega-cap)
- *Profitability Factor:* Gross Profit: $90B, Total Assets: $150B / Ratio: 90/150 = 0.60 → **+2** (highly profitable)
- *Investment Factor:* Asset growth: 12% YoY → **0** (moderate, disciplined)
- *Momentum Factor:* 12-month return: +180% → **+2** (strong momentum)

**Total Factor Score:** -2 (Value) -2 (Size) +2 (Profitability) +0 (Investment) +2 (Momentum) = **0**

*Interpretation:* NVDA has a neutral factor score. It's expensive and mega-cap (negative), but highly profitable with strong momentum (positive). This is a quality growth at a premium setup — only buy if Layers 2-4 are exceptionally strong (e.g., major earnings beat, new product cycle). Not a screaming bargain, but not a red flag either.

---

**Stock: JPM** (as of a sample observation date)

- Value: P/E 12x vs. sector 13x → 0 (only 7.7% below sector, threshold requires ≥20% discount); P/B 1.8x vs. sector 1.5x → 0 (exactly at 1.2x threshold, not above) → Value: **0**
- Size: Market cap $600B → **-2** (mega-cap)
- Profitability: Gross Profit / Assets = 0.25 → **0** (banks have lower ratios structurally)
- Investment: Asset growth 5% YoY → **+1** (disciplined)
- Momentum: 12-month return +25% → **+1**

**Total:** 0 -2 +0 +1 +1 = **0** (neutral, mega-cap with momentum but not cheap enough to score value premium)

---

**Stock: AAPL** (as of a sample observation date)

- Value: P/E 30x vs. sector 25x → 0 (exactly at 1.2x threshold, not above); P/B 50x vs. sector 8x → -1 (6.25x sector, far above threshold) → Value: **-1**
- Size: Market cap $3.5T → **-2**
- Profitability: Gross Profit / Assets = 0.45 → **+2**
- Investment: Asset growth 8% YoY → **+1**
- Momentum: 12-month return +35% → **+2**

**Total:** -1 -2 +2 +1 +2 = **+2** (balanced positive, quality growth with momentum, P/B drag partially offset by other factors)

---

**Integration with Layer 2 (Fundamental Filter):**

Add this optional checkpoint to Layer 2:
- Bonus **+1 point** in Layer 2 scoring if Factor Exposure Score ≥ +5 (multi-factor tailwind)
- Penalty **-1 point** in Layer 2 scoring if Factor Exposure Score ≤ -3 (multi-factor headwind)

This prevents chasing stocks that look good on surface metrics (EPS growth, momentum) but are structurally poor on factor exposure (e.g., unprofitable momentum stock with massive asset growth = future value trap).

---

**When Factor Scoring Saves You:**

- **Value Trap:** Stock looks cheap (P/E 8x), but Factor Score = -6 (unprofitable, negative momentum, overinvesting in assets) → Pass. Cheap + bad = value trap.
- **Quality Growth:** Stock looks expensive (P/E 40x), but Factor Score = +6 (highly profitable, disciplined investment, positive momentum) → Proceed if Layers 2-4 confirm. Expensive + excellent = potential winner.
- **Momentum Bubble:** Stock has +80% 12-month return, but Factor Score = -4 (overvalued, unprofitable, overinvesting) → Caution. Pure momentum with no quality = likely crash after catalyst fails.

---

## Supplementary Factors
Use these to differentiate between candidates that all pass the 3 layers, or to size up higher-conviction positions:

| Factor | Signal | Tool |
|--------|--------|------|
| Insider buying | Cluster buying (multiple insiders) is strongly bullish | OpenInsider.com, SEC Form 4 |
| Institutional flows | New positions by high-conviction funds | WhaleWisdom, dataroma.com |
| Short interest | > 15% of float = squeeze amplifier if catalyst hits | Finviz, Yahoo Finance |
| Unusual options activity | Large OTM call buying above normal volume | Unusual Whales, Barchart |
| Secular theme alignment | AI, defense, nuclear/power, GLP-1, reshoring | Qualitative |
| Management quality | ROIC > 15%, active buyback program, guidance accuracy | Company filings |
| Earnings quality | Revenue-driven beats, FCF ≥ Net Income | Earnings transcripts |
| Seasonality | Cross-check seasonal patterns as tailwind/headwind | StockCharts seasonality |

---

## Decision Checklist
Run through every candidate before adding a position:

**Macro**
- [ ] Stock is in a sector with current macro tailwinds
- [ ] Risk regime (VIX, credit spreads) supports being long

**Fundamental**
- [ ] EPS estimates raised in the last 60 days
- [ ] Projected EPS growth > 15% YoY
- [ ] Valuation reasonable vs. peers (PEG < 1.5 or EV/EBITDA below sector median)
- [ ] FCF positive, Net Debt/EBITDA < 2x, EBIT/Interest > 3x, ROIC > WACC
- [ ] Clear near-term catalyst identified

**Technical**
- [ ] Price above 50-day and 200-day MA
- [ ] Clean entry setup present
- [ ] Risk/reward ≥ 2:1
- [ ] RS line trending up vs. S&P 500

**Smart Money**
- [ ] Institutional accumulation: high-conviction fund added/increased position (13F)
- [ ] Short interest trending down (bears giving up)
- [ ] Smart options flow: sweep orders or large OTM call buying present
- [ ] Analyst consensus momentum: 1+ upgrade or PT raise in last 30 days

**Quantitative Overlay** *(Optional — for portfolio construction, high-conviction sizing, or pre-earnings positions)*
- [ ] Factor exposure score ≥ +4 (balanced positive or multi-factor tailwind)
- [ ] Factor exposure does NOT show value trap pattern (cheap + unprofitable + negative momentum)
- [ ] Not buying at volatility extremes (deferred to Week 2)
- [ ] Risk-adjusted metrics calculated (deferred to Week 3)
- [ ] Portfolio correlation checked if adding to existing positions (deferred to Week 3)

**Checklist Interpretation:**
- 12-14 core boxes checked (Layers 1-4) → Strong Buy, medium conviction
- 14-19 total boxes checked (Layers 1-5) → Strong Buy, high conviction with quantitative validation
- 9-11 core boxes → Buy candidate, medium conviction
- < 9 core boxes → Pass or watch list

---

## Output Format
When presenting stock recommendations, always produce:

### Summary Table
Always sorted by Final Rank Score (highest conviction + best risk/reward first). Never sort alphabetically or by sector.

| Rank | Ticker | Sector | Conviction | Risk | Upside | R/R | Rank Score | Verdict |
|------|--------|--------|------------|------|--------|-----|------------|---------|
| 1 | TICK | Sector | 11/14 (7.9) | Low | +28% | 3.2x | **25.2** | Strong Buy |

### Per-Stock Detail Block (for each stock)
- **Investment thesis:** 1-2 sentence bull case
- **Key catalyst:** Specific near-term event driving re-rating
- **Quantitative scorecard** (if Layer 5 used):
  - Factor exposure: [+X/10] (Value: +X, Size: +X, Profitability: +X, Investment: +X, Momentum: +X)
  - Interpretation: [e.g., "Multi-factor tailwind — quality + momentum" or "Expensive growth, requires strong catalyst"]
  - *[Note: If Layer 5 not calculated, show: "Quantitative scorecard: [Not calculated — quick analysis mode]"]*
- **Smart money signals:** List which of the 5 smart money signals are positive (e.g., "13F: Druckenmiller increased stake; Options: sweep calls detected; Analysts: 2 upgrades last 30 days")
- **Entry safe zone:** Specific $ price range that offers favorable risk/reward (e.g., "$142 – $148"). Derived from: pullback to 50-day MA, prior support level, or base of recent consolidation. Never recommend entry at an extended/near-52-week-high price.
- **Stop loss:** Specific $ level and % from the midpoint of the entry zone
- **6M price target:** Specific $ target and % upside from entry zone midpoint
- **Risk/reward ratio:** (Target − Entry midpoint) / (Entry midpoint − Stop)
- **Key risks:** 2-3 specific downside scenarios

### Entry Safe Zone Calculation Rules
When deriving the entry safe zone, use this priority order:

1. Pullback to rising 50-day MA — ideal for trending stocks (safest entry)
2. Prior consolidation base or breakout retest — for stocks that just broke out
3. 21-day EMA bounce zone — for momentum stocks with tight bases
4. Discount to current price of 5-15% — only if the stock is in a tight range near support

Always express as a $ range (low – high), not a single price. The range should be ≤ 5% wide.

Example:
```
Current price:    $162
50-day MA:        $148
Entry safe zone:  $145 – $150  (pullback to 50-day MA area)
Stop loss:        $138  (-8% from $150)
6M target:        $185  (+25% from $147 midpoint)
R/R ratio:        (185-147) / (147-138) = 4.2x  ✓
```

### Observation Date
Always state the observation date at the top of every analysis in this format:
```
Observation date: [Day, Month DD YYYY]
Current price at observation: $XX.XX
```

This makes it clear when the analysis was done and allows the user to recalibrate entry zones if they return to this analysis days or weeks later.

**Always include this caveat:**
> Analysis based on data available on the observation date above. Prices move daily — always re-verify current price vs. entry zone before acting. This is a research framework, not financial advice.

---

## Portfolio Review Output Format
Use this output format when running a Portfolio Review (user provides average costs).

### Portfolio Summary Table
Always start with this table for a quick at-a-glance view:

```
Observation date: [Day, Month DD YYYY]

| # | Ticker | Avg Cost | Current | P&L % | Thesis | Verdict | PT1 | PT2 | Stop |
|---|--------|----------|---------|-------|--------|---------|-----|-----|------|
| 1 | TICK   | $XX.XX   | $XX.XX  | +X%   | Intact | Hold    | $XX | $XX | $XX  |
```

Column definitions:
- **Avg Cost:** The user's average cost basis per share
- **Current:** Price at time of analysis (state observation date)
- **P&L %:** Unrealized gain/loss from avg cost to current price
- **Thesis:** Intact / Weakening / Broken
- **Verdict:** Add / Hold / Trim / Trim More / Reduce / Sell
- **PT1:** First profit-take level — sell 25-33% of position here
- **PT2:** Second profit-take level — sell another 25-33% here
- **Stop:** Hard stop level — exit remainder if breached

### Per-Position Detail Block
After the summary table, provide a detail block for each position:

```
─────────────────────────────────────────
[TICKER] — [Verdict in CAPS]
─────────────────────────────────────────
Avg cost:       $XX.XX
Current price:  $XX.XX  (P&L: +/-X% / +/-$X.XX per share)

Thesis status: [Intact / Weakening / Broken]
→ [One sentence explaining why — specific, not generic]

Staged exit plan:
  PT1 → $XX.XX  (+X% from avg cost) — sell 25-33% of position
  PT2 → $XX.XX  (+X% from avg cost) — sell another 25-33%
  Trail remainder with stop at $XX.XX

Hard stop: $XX.XX — exit full remaining position if breached
Re-entry level: $XX.XX – $XX.XX (if sold, this is where the framework re-triggers a buy)

What would change this view:
  → Bull flip: [Specific signal that would make this more bullish]
  → Bear flip: [Specific signal that would trigger a downgrade]

Action: [Precise, actionable instruction]
─────────────────────────────────────────
```

### Portfolio-Level Risk Summary
Append this block at the end of every portfolio review output:

```
═══════════════════════════════════════════
PORTFOLIO RISK SUMMARY
═══════════════════════════════════════════
Observation date: [Day, Month DD YYYY]
Total positions reviewed: X

Overall P&L: [+/-X% blended if sizes known, or list per position]

Sector concentration:
  [Sector A]: X positions (XX% of portfolio) [FLAG if >40%]
  [Sector B]: X positions (XX% of portfolio)

Position count: X / recommended 8-15  [FLAG if <8 or >15]

Highest-risk positions: [List tickers with Weakening/Broken thesis or positions below stop]

Correlation check (paste tickers into portfoliovisualizer.com → Backtest Portfolio → 6-month window):
  Highly correlated pairs (>0.7): [list pairs, or "None detected"]
  ⚠ Flag if ≥2 pairs exceed 0.7 — diversification is illusory at that level

⚠ Risk flags:
  [ ] Sector over-concentration: [flag if any sector >40%]
  [ ] Thesis broken positions still held: [flag tickers]
  [ ] Positions below stop loss: [flag tickers]
  [ ] Over-concentrated in single stock (>10% portfolio): [flag tickers]
  [ ] Hidden correlation: ≥2 pairs with >0.7 correlation [flag pairs]
  [ ] All clear — no critical flags [use only if no flags triggered]

Key action items (priority order):
  1. [Most urgent action]
  2. [Next action]
  3. [Optional add]
═══════════════════════════════════════════
```

**Rules for the risk summary:**
- Always check sector concentration. If one sector holds >40% of counted positions, flag it.
- Flag any position where the thesis is Broken or price is already below the stop level — these are highest priority.
- Flag if total position count is outside 8-15. Under 8 = under-diversified. Over 15 = difficult to monitor.
- For correlation: ≥2 pairs above 0.7 = hidden concentration risk even if sectors look diverse. The most dangerous form is tech + tech-adjacent (e.g., NVDA + AMD + SMCI all move together).
- The priority action list should be ranked: Sells first, Trims second, Potential Adds last.
- If the user did not provide position sizes, note: "Position sizes not provided — concentration % estimates assume equal weighting."

---

## Portfolio Construction Rules
- Max 10% per position; start at half-size, add on confirmation
- Max 40% in any single sector
- Target 8-15 positions total
- Honor stop losses without exception — never average down into a loser
- Re-evaluate each position after every earnings report
- Re-evaluate macro layer monthly

## Useful Free Tools

### Layers 1-4 Tools
| Purpose | Tool |
|---------|------|
| Stock screener | finviz.com |
| EPS revisions | zacks.com |
| Technical charts | tradingview.com |
| Insider activity | openinsider.com |
| Institutional 13F | whalewisdom.com, dataroma.com |
| Unusual options + dark pool | unusualwhales.com |
| Options flow detail | marketchameleon.com, barchart.com |
| Dark pool prints | quantdata.us, unusualwhales.com |
| Short interest trend | finra.org, finviz.com |
| Analyst ratings history | tipranks.com, marketbeat.com |
| Fed rate expectations | CME FedWatch Tool |
| Macro indicators | FRED (St. Louis Fed) |
| Sector rotation + ETF flows | StockCharts.com (SPDR sector ETFs) |

### Layer 5 Tools (Quantitative Overlay)
| Metric | Free Source | How to Get |
|--------|-------------|------------|
| Factor Exposure (Value, Size) | Finviz, Yahoo Finance | P/B, P/E, Market Cap from Finviz screener |
| Factor Exposure (Profitability) | Yahoo Finance, Company 10-K | Gross Profit / Total Assets from Balance Sheet + Income Statement |
| Factor Exposure (Investment) | Yahoo Finance | Total Assets growth YoY from Balance Sheet |
| Factor Exposure (Momentum) | Yahoo Finance, TradingView | 12-month price return calculation |
| Correlation Matrix | Portfolio Visualizer (free), Python pandas | Upload portfolio tickers, get 6-month correlation matrix |
| Sharpe/Sortino Ratio | Yahoo Finance + Excel/Python | Download daily returns, calculate in spreadsheet (see Week 3) |
| IV Percentile | Barchart (free tier), TradingView | 52-week IV range, current IV percentile |
| Earnings Beat Probability | Earnings Whispers, TipRanks, Optionistics | Historical beat rate + estimate revisions + options-implied move (see Week 4) |
