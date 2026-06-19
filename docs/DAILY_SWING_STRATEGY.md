# Daily Swing Confluence — Playbook

> A **new, standalone** swing system built specifically for the **1-Day (1D) timeframe**.
> It is *not* derived from the other strategy in this repo — it uses a leaner,
> 6-factor confluence model and produces clean **BUY / SELL** signals.
>
> **Reality check.** No signal is a guarantee. Markets are adversarial and partly
> random. Edge comes from process: trade only high-confluence setups, size risk so
> no single loss hurts, and let positive expectancy play out over many trades. Every
> signal is a *hypothesis*, not a certainty. Backtest and paper-trade first.

---

## 1. Why the 1-Day timeframe

Swing trades are held for **days to weeks**. The daily chart is the natural home for that:
each candle is one full session, so daily structure, gaps and volume reflect *decisions by
real participants* rather than intraday noise. Everything below is tuned for 1D candles. The
scripts will print a warning if you load them on another timeframe (you can still use them,
but the defaults assume daily).

> **Note on VWAP.** A *session* VWAP is meaningless on a 1D chart (one bar = one whole
> session). So this system uses a **Rolling** volume-weighted average (default 30 days) — or,
> if you prefer, a Week / Month / Quarter / Year **anchored** VWAP. Either way it answers one
> question: *are buyers or sellers in control of the recent value?*

---

## 2. The six factors (max score = 9)

| # | Factor | Weight | What it answers |
|---|---|:---:|---|
| 2 | **FVG Retest** | 2 | Is price rebalancing an imbalance? |
| 3 | **Order Block** | 2 | Is price back at an institutional origin? |
| 4 | **Volume Profile (VAH/VAL/POC)** | 1–2 | Is price at fair-value support/resistance? |
| 5 | **VWAP Side** | 1 | Which side is in control? |
| 6 | **Volume Imbalance** | 1 | Where is order-flow pressure? |
| 8 | **Liquidity Sweep** | 1 | Did a stop hunt just reverse? |

### 2 · FVG Retest (weight 2)
A **Fair Value Gap** is a 3-candle imbalance — price moved so fast it left an untraded gap
(`low > high[2]` for a bullish gap). Markets tend to revisit and "rebalance" these zones. We
only count gaps larger than a fraction of ATR (a **displacement filter**, so tiny gaps are
ignored), track several live gaps at once, and score the factor when price **retests** a
gap — i.e. comes *back into* a zone that formed earlier. A retest of a **bullish** FVG is a
buy clue; a **bearish** FVG retest is a sell clue. Zones expire by age or when price closes
through them (mitigation).

### 3 · Order Block (weight 2)
An **Order Block** is the **last opposing candle before an explosive move** — the last
down-candle before a rally (bullish OB) or the last up-candle before a sell-off (bearish OB).
That candle is where a large player absorbed the other side before driving price. When price
returns to it, those resting orders often defend the level. We locate the genuine
last-opposite candle before the displacement that created the FVG, then score the factor on a
**retest** of that zone.

### 4 · Volume Profile — VAH / VAL / POC (weight 1–2)
Instead of volume-over-time, a **Volume Profile** shows volume-**at-price** over a lookback
window (default 120 daily bars ≈ 6 months):
- **POC (Point of Control)** — the most-traded price = strongest magnet / fair value.
- **Value Area (VAH/VAL)** — the band holding ~70% of volume = where the "real" market lives.

Price gravitates to the POC and reacts at the value-area edges. Scoring:
- **+1** when price sits in the *correct half* of value (between VAL and POC for buys; between
  POC and VAH for sells).
- **+2** (the strong case) when price is *at the value edge* — tagging/reclaiming **VAL** for
  buys, or tagging/rejecting **VAH** for sells. That's the highest-quality fair-value entry.

*(Computed every bar via a fast typical-price histogram, so it works across history and inside
the backtest — not just on the last bar.)*

### 5 · VWAP Side (weight 1)
The volume-weighted average price is the benchmark institutions are measured against. **Above
VWAP = buyers in control; below = sellers.** +1 to buys when `close > VWAP`, +1 to sells when
`close < VWAP`. Default is a **Rolling 30-day** VWAP (always meaningful on a daily chart);
anchored Week/Month/Quarter/Year options are available.

### 6 · Volume Imbalance — order-flow pressure (weight 1)
A proxy for buy-vs-sell **delta** when a real order-flow feed isn't available. For each bar we
split volume by where price closed inside its range:
- buy volume ≈ `volume × (close − low) / (high − low)`
- sell volume ≈ `volume × (high − close) / (high − low)`

Summed over a short window (default 3 bars), if one side beats the other by the **dominance
threshold** (default 10%) it counts as genuine pressure: +1 to buys on net buying, +1 to sells
on net selling. A pretty setup with *opposing* order flow is a warning.

### 8 · Liquidity Sweep (weight 1)
Stops cluster just beyond recent swing highs/lows. A **sweep** spikes through a prior extreme
to trigger those stops, then **immediately rejects back inside** — classic stop-hunt reversal.
A sweep of sell-side liquidity (a new low that closes back up) is a **buy** clue; a sweep of
buy-side liquidity (a new high that closes back down) is a **sell** clue. A short recency
window lets a sweep from the last few bars still count.

---

## 3. How a signal is built

```
buy_score  = FVG(2) + OB(2) + VolumeProfile(0..2) + VWAP(1) + VolImbalance(1) + Sweep(1)
sell_score = symmetric
```

A **BUY** prints only when **all** of these hold:
1. `buy_score ≥ threshold` (default **5 of 9**), **and**
2. **gate:** price is inside a real **FVG or Order Block** zone (toggle: *Require FVG or OB*),
   **and**
3. the optional EMA trend gate agrees (OFF by default).

**SELL** is the mirror image. Buy and sell are **mutually exclusive** — the higher score wins,
and an exact tie stands down. The signal **triggers** on the bar it first becomes true (on bar
close), so it does not repaint after the candle closes.

**Grades:** `A+` ≥ 8 · `A` ≥ 6 · `B` ≥ 5. Trade A/A+; treat B as "watch."

This is deliberately strict — you'll get **fewer** signals than a single-pattern indicator.
That's the point: a handful of high-quality swings beats constant action.

---

## 4. How to trade it

**Entry**
1. Wait for a **BUY / SELL** label or alert (prints on the daily close).
2. Confirm the grade (read the dashboard). Prefer **A / A+**.
3. Enter on the signal-bar close, or set a limit into the FVG / OB zone for a better fill.

**Stop loss** — placed just beyond the structure that invalidates the idea: below the
OB/FVG low (buys) or above the high (sells), padded by `0.5 × ATR`. If price closes beyond your
stop, the thesis is wrong — be out.

**Targets & management**
- **TP1 = 1R** → scale out ~50%, move stop to breakeven (trade is now risk-free).
- **TP2 = 2R** runner — or trail to the **POC**, the opposite value-area edge, or the next
  liquidity pool.

**Risk** — a fixed small % of equity per trade (**1%** default in the strategy). With ~2R
winners you only need to be right ~40% of the time to be net profitable. **Position size is the
real edge, not the entry.**

**Stand down** when score is below threshold, when buy and sell scores are close (chop), or
into major scheduled news.

---

## 5. Recommended settings (1D defaults)

| Input | Default | Note |
|---|---:|---|
| Min FVG size (× ATR) | 0.25 | Raise for fewer, cleaner gaps |
| OB lookback | 7 | Candles before the impulse |
| VP lookback / rows | 120 / 40 | ~6 months of value context |
| VWAP anchor | Rolling 30 | Or Quarter for an anchored benchmark |
| Volume Imbalance window / threshold | 3 / 10% | Order-flow pressure sensitivity |
| Liquidity lookback / recency | 20 / 3 | Swing pool size / sweep freshness |
| Min score | 5 of 9 | Lower = more signals; 6–7 = stricter |
| Require FVG or OB | on | The core gate |
| Optional EMA trend gate | off | Turn on to trade with a 50-EMA trend |

- **More signals?** Lower the score to 4, or turn off the zone gate.
- **Fewer / cleaner?** Raise to 6–7, turn on the EMA trend gate, and only trade A+.
- **Crypto:** sweeps and volume profile shine; a Weekly-anchored VWAP suits 24/7 markets.
- **Forex:** lean on FVG/OB/sweep/VWAP — tick volume makes the volume factors less reliable.

---

## 6. Honest limitations

- **Volume Profile** uses a typical-price histogram for speed; it's a close approximation of a
  true tick profile, not an exact one.
- **Volume Imbalance** is an *estimate* from candle structure, not real exchange delta — it's a
  pressure proxy, strongest as confirmation rather than a standalone trigger.
- **Backtest ≠ live.** Always include realistic commission + slippage (the strategy defaults
  do). Validate across several symbols and periods — curve-fitting one symbol is self-deception.
- **No edge is permanent.** Regimes change; re-validate periodically.

The goal isn't to predict the future. It's to act only when the odds are stacked, lose small
when wrong, and win bigger when right — and let the math do the rest.
