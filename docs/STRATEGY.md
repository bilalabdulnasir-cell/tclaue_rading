# The Institutional Swing Playbook

> A deep dive into *why* this system is built the way it is, and *how* to actually trade it.
>
> **Reality check first.** No indicator is "perfect," and anyone who tells you they have a strategy that always wins is selling something. Markets are adversarial and partly random. What separates consistently profitable traders from everyone else is **process**: trading only high-confluence setups, sizing risk so no single loss hurts, and letting a positive expectancy play out over hundreds of trades. This system is a *decision framework* for finding those setups — not a money printer. Treat every signal as a hypothesis, not a certainty.

---

## 1. The core idea: confluence, not patterns

Most retail indicators fire on a **single** event — an RSI cross, one Fair Value Gap, one moving-average touch. The problem: every one of those events happens constantly, most of them fail, and you can't tell the good ones from the noise.

Institutions don't think in single signals. A desk asks a *stack* of questions before committing size:

1. **Which way is the market actually trending?** (bias)
2. **Is price expensive or cheap relative to the recent range?** (location)
3. **Is there an inefficiency price wants to rebalance?** (imbalance / FVG)
4. **Is there evidence a large player already transacted here?** (order block)
5. **Did price just run someone's stops to grab liquidity?** (sweep)
6. **Where is the most volume actually trading — where is fair value?** (volume profile / VWAP)
7. **Is real participation showing up, or is this a thin drift?** (volume)

A trade is only worth taking when **most of these agree**. That's exactly what this system encodes: each question becomes a weighted factor, and a signal only triggers when the weighted **score** clears a threshold **and** the non-negotiable gates pass. One factor lighting up means nothing; seven of them aligning is a setup.

---

## 2. The eight building blocks

### 1) Market Structure → Bias (weight 2 — the heaviest)
Price makes higher highs/higher lows in an uptrend and the reverse in a downtrend. We detect confirmed swing pivots, then watch for:
- **BOS (Break of Structure):** price closes beyond the last swing *in the trend direction* → trend continues.
- **CHoCH (Change of Character):** the *first* break *against* the prevailing trend → trend may be flipping.

Bias is the heaviest weight because **trading against structure is the single fastest way to lose**. A higher-timeframe (HTF) EMA filter sits on top so your 1H entries respect the daily trend.

### 2) Premium / Discount (weight 1 — a mandatory gate)
Take the most recent confirmed swing high and low — that's the **dealing range**. The 50% level is **equilibrium**.
- Below equilibrium = **discount** → only place to look for *buys*.
- Above equilibrium = **premium** → only place to look for *sells*.

This is the institutional version of "buy low, sell high." The 0.62–0.79 retracement band is the **OTE (Optimal Trade Entry)** — the deep-discount/premium pocket where risk:reward is best. *Buying in premium is how retail gets trapped at the top.*

### 3) Fair Value Gaps — imbalance (weight 2)
A FVG is a 3-candle pattern where price moved so fast it left a gap (bar 1's high < bar 3's low for a bullish gap). That gap is an **imbalance** — a price zone that traded one-directionally. Markets tend to revisit and "rebalance" these zones.

We only count gaps larger than a fraction of ATR (the **displacement filter**) so tiny, meaningless gaps are ignored — a real institutional footprint is a *violent* move, not a one-tick gap. We track **multiple** live FVGs at once and remove them once price fills (mitigates) them.

### 4) Order Blocks (weight 2)
An order block is the **last opposing candle before an explosive move** — e.g., the last down-close candle right before a sharp rally. The logic: that candle is where a large player absorbed the opposite side before driving price. When price returns to it, those resting orders often defend the level. We find the genuine last-opposite candle before the displacement, not just "the candle two bars ago."

### 5) Liquidity (weight 2)
Stops cluster in obvious places — just beyond equal highs/lows and prior swings. That clustered stop volume **is** liquidity, and price is drawn to it. A **sweep** (a.k.a. stop hunt) is when price spikes through a prior extreme to trigger those stops, then *immediately rejects back*. A sweep of sell-side liquidity (below) that reverses up is a high-quality **buy** trigger — you're entering right after the "smart money" filled their position on trapped sellers. We also mark **equal highs/lows** as resting liquidity pools (magnets/targets).

### 6) Volume Profile (weight 1 — indicator only)
Instead of volume-over-time, Volume Profile shows volume-**at-price** over a lookback window:
- **POC (Point of Control):** the single most-traded price = strongest magnet / fair value.
- **Value Area (VAH/VAL):** the price band containing ~70% of volume = where the "real" market lives.

Price tends to gravitate to the POC and react at the value-area edges. We use **VAL** as a buy-support reference and **VAH** as a sell-resistance reference. *(This lives in the indicator only; it's too heavy to compute reliably bar-by-bar inside a backtest engine.)*

### 7) Anchored VWAP + bands (weight 1)
VWAP is the **volume-weighted average price** since an anchor (session/week/month) — the benchmark institutions are literally measured against. Above VWAP = buyers in control; below = sellers. The ±standard-deviation **bands** mark statistical stretch: a tag of the lower band in an uptrend is a classic mean-reversion buy zone.

### 8) Relative Volume (weight 1)
A reversal on **high relative volume** (RVOL > ~1.3× average) means real participation is confirming the move. A pretty setup on dead volume is a trap waiting to happen.

---

## 3. The scoring model

Each factor contributes points to a directional score (max **13** in the indicator, **12** in the strategy without Volume Profile):

| Factor | Weight | Why |
|---|---:|---|
| Market structure bias | 2 | Don't fight the trend |
| FVG retest (imbalance) | 2 | The entry trigger |
| Order block retest | 2 | Institutional footprint |
| Liquidity sweep | 2 | Smart-money fill / trap |
| HTF trend agreement | 1 | Multi-timeframe alignment |
| Premium/Discount location | 1 | Buy cheap / sell expensive |
| VWAP side | 1 | Benchmark control |
| Volume Profile (VAL/VAH) | 1 | Fair-value reference |
| Volume confirmation (RVOL) | 1 | Real participation |

**Mandatory gates** (no signal fires unless ALL are true):
- Bias agrees with the trade direction, **and**
- Price is retesting a real FVG **or** Order Block, **and**
- HTF trend agrees (if the filter is on), **and**
- Location is correct (discount for buys, premium for sells).

Then the weighted score must clear your **threshold** (default 7/13). Grades: **A+** ≥ 11, **A** ≥ 9, **B** ≥ 7. Trade A/A+ setups; treat B as "watch."

This is deliberately strict. You will get **fewer** signals than a typical indicator — that's the point. Swing trading is about a handful of high-quality trades, not constant action.

---

## 4. How to actually trade it (the rules)

**Entry**
1. Wait for a **BUY** (or SELL) label / alert — it only prints on bar close.
2. Confirm grade is **A or A+** (read the dashboard). Skip B-grades unless you have an external reason.
3. Enter on the close of the signal bar, or set a limit order into the FVG/OB zone for a better fill.

**Stop loss**
- The system places it just beyond the structure that invalidates the idea: below the order-block/FVG low (buys) or above the high (sells), padded by 0.5× ATR. **If price closes beyond your stop, the thesis is wrong — be out.**

**Targets & management**
- **TP1 = 1R** (scale out ~50%, then move stop to breakeven — now the trade is risk-free).
- **TP2 = 2R** (runner). Or trail to the next liquidity pool / opposite FVG / the POC.
- Logical targets: opposing liquidity (equal highs/lows), VAH/VAL, the POC, or equilibrium of the range.

**Risk**
- Risk a **fixed small %** of equity per trade (1% default in the strategy). With ~2R winners you only need to be right ~40% of the time to be net profitable. **Position size is the real edge — not the entry.**

**Filters / when to stand down**
- Skip when bias is neutral (0) or HTF disagrees.
- Skip into major scheduled news.
- One position per symbol (no pyramiding by default).

---

## 5. Recommended settings

These ship as the defaults (tuned for **stocks / indices**). Adjust to taste.

| Timeframe / use | Pivot Len | FVG Age | VWAP Anchor | HTF | Score Threshold |
|---|---:|---:|---|---|---:|
| Day-trade 5–15m | 5–6 | 60–80 | Session | Daily | 7 |
| **Swing 1H–4H (default)** | **8** | **120** | **Session/Week** | **Daily** | **7** |
| Position 1D | 10–12 | 200 | Month | Weekly | 6–7 |

- **Forex:** lower the volume weighting's importance in your reading (tick volume is unreliable); lean on structure, FVG, OB, liquidity, VWAP.
- **Crypto:** anchor VWAP **Weekly** (24/7 market); sweeps and volume profile are especially powerful.
- Want more signals? Lower the threshold to 6 and/or turn off the HTF filter. Want fewer/cleaner? Raise to 9+ and only trade A+.

---

## 6. Honest limitations

- **Repainting/lag:** Swing pivots confirm `Pivot Length` bars *after* they form — unavoidable, and the reason signals are reliable rather than early. HTF values can update intrabar; judge on closed bars.
- **Volume Profile** is computed on the last bar over a fixed lookback — it's a *current-context* tool, not a historical signal, and is intentionally excluded from the backtest.
- **Backtest ≠ live.** Always include realistic commission + slippage (the strategy defaults do). Curve-fitting to one symbol is self-deception — validate across several symbols and time periods.
- **No edge is permanent.** Regimes change. Re-validate periodically.

The goal isn't to predict the future. It's to only act when the odds are stacked, to lose small when you're wrong, and to win bigger when you're right. Do that with discipline and the math takes care of the rest.
