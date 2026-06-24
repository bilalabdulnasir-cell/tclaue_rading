# Institutional Swing Suite

A confluence-based swing-trading toolkit for TradingView (Pine Script v5) that combines the techniques institutional desks actually use into a single, weighted **buy/sell decision engine** — instead of firing off one indicator at a time.

It fuses:

- **Smart-Money Concepts** — market structure (BOS/CHoCH), Order Blocks, liquidity sweeps
- **Imbalance** — Fair Value Gaps with a displacement filter (multi-zone, auto-mitigation)
- **Premium/Discount** — dealing range, equilibrium, and the OTE entry pocket
- **Volume Profile** — POC, Value Area High/Low (volume-at-price)
- **Anchored VWAP** — session/week/month VWAP with standard-deviation bands
- **Volume confirmation** — relative-volume (RVOL) gating

A signal only triggers when **enough of these agree** (a weighted score) **and** the mandatory gates pass (bias + location + a real institutional zone).

> ⚠️ **Risk disclaimer.** This is an educational tool, not financial advice and not a guarantee of profit. No indicator is "perfect." Trading involves substantial risk of loss. Backtest thoroughly, paper-trade first, and never risk money you can't afford to lose.

---

## What's in here

| File | Purpose |
|---|---|
| [`indicators/daily_liquidity_orderflow_vp.pine`](indicators/daily_liquidity_orderflow_vp.pine) | **Simple 3-pillar model (start here)** — a focused Pine v6 indicator for the **Daily** timeframe: **Liquidity Sweep + Order Flow + Volume Profile**. Marks where institutions enter and exit with clear LONG/SHORT/EXIT labels and alerts. |
| [`indicators/swing_institutional_suite.pine`](indicators/swing_institutional_suite.pine) | **Full visual indicator** — all 8 modules, on-chart zones, Volume Profile, a live dashboard, signal labels, and alerts. Use this to *read* the chart and get alerts. |
| [`strategies/swing_institutional_strategy.pine`](strategies/swing_institutional_strategy.pine) | **Backtestable strategy** — the same confluence engine wired to entries/exits, risk-based sizing, a fixed stop and a 2-stage scale-out. Use this to *validate* on historical data. |
| [`docs/STRATEGY.md`](docs/STRATEGY.md) | **The playbook** — the deep "why" behind every module, the scoring model, and the exact rules for entries, stops, targets, and risk. **Read this.** |

The strategy omits Volume Profile (too heavy to compute reliably inside a backtest engine), so its max score is 12 vs. the indicator's 13. Everything else matches.

---

## The simple Daily model — Liquidity Sweep + Order Flow + Volume Profile

If the 8-module suite is more than you want, [`indicators/daily_liquidity_orderflow_vp.pine`](indicators/daily_liquidity_orderflow_vp.pine) is a **clean, modern (Pine v6) indicator built for the Daily timeframe** that does one job well: find where institutions accumulate/distribute, then mark the entry and the exit.

It requires **all three pillars to agree** before a signal:

1. **Liquidity Sweep** — price runs a prior swing high/low to grab resting stops, then *reclaims* the level (a stop-hunt/trap). This is the moment desks fill against trapped traders.
2. **Order Flow** — true buy-vs-sell **volume delta**, estimated from *intrabar* data via `request.security_lower_tf` (1H candles inside each daily bar by default). It confirms whether buyers or sellers are actually in control on the sweep — the modern way to read order flow on a standard chart.
3. **Volume Profile** — POC / Value-Area High-Low (volume-at-price). Longs are only taken at a **discount** (at/below the POC), shorts only at a **premium**, and the opposite value edge (VAH/VAL) is the logical target.

**Entry → Exit logic**
- **LONG** when a bullish sweep is fresh **and** order-flow delta is positive **and** price is in discount. Mirror for **SHORT**.
- **Stop** goes just beyond the sweep extreme (ATR-padded). **Target** is the opposite value-area edge, falling back to a fixed R-multiple.
- **Exit** fires on the stop, the target, or an opposite sweep (flip). One position at a time, so the chart reads as a clean sequence of institutional **entries and exits** with `LONG` / `SHORT` / `EXIT` labels, a 3-pillar dashboard, and alerts.

> Tuned for the **Daily** timeframe (1H order-flow intrabars). On a Daily chart the defaults work as-is; if you change chart timeframe, set the *Order-Flow Lower Timeframe* below the chart's.

---

## Quick start (TradingView)

1. Open [TradingView](https://www.tradingview.com) → any chart → **Pine Editor** (bottom panel).
2. Copy the contents of a `.pine` file from this repo, paste it into the editor.
3. Click **Add to chart**.
   - For the **indicator**: you'll see zones, VWAP, the Volume Profile levels, and a dashboard top-right.
   - For the **strategy**: open the **Strategy Tester** tab to see backtest stats.
4. Open the indicator's settings (gear icon) to tune inputs by group.

### Setting up alerts (indicator)
- Add the indicator → click the **alarm clock** → Condition: *Institutional Swing Suite* → choose **ISS · BUY** or **ISS · SELL** → set to **Once Per Bar Close** (important — avoids repaint).

---

## How to read the dashboard

The top-right panel shows every factor as **YES/no** for both BUY and SELL, plus:
- **SCORE / 13** — the weighted confluence total (green when it clears your threshold).
- **GRADE** — A+ (≥11), A (≥9), B (≥7). Trade A/A+; treat B as "watch."
- **SIGNAL** — BUY / SELL / WAIT.
- **PLAN** — suggested stop and TP2 for the active signal.

A label and chart background highlight print on the bar a signal triggers, and the entry/SL/TP levels are drawn for you.

---

## Default tuning

Defaults are set for **stocks / indices, swing timeframes (1H–4H)**: session-anchored VWAP, daily HTF bias filter, score threshold 7. See the [playbook](docs/STRATEGY.md#5-recommended-settings) for per-timeframe and per-asset presets (crypto, forex, day-trading, position-trading).

---

## Core trade rules (TL;DR)

1. **Only** take A/A+ signals that agree with higher-timeframe trend.
2. Enter on signal-bar close (or a limit into the FVG/OB zone).
3. Stop goes beyond the invalidating structure (auto-calculated, ATR-padded).
4. Scale out 50% at **1R**, move to breakeven, let the rest run to **2R** / next liquidity.
5. Risk a fixed **1%** (or less) of equity per trade. **Sizing is the edge.**

Full reasoning and edge cases are in [`docs/STRATEGY.md`](docs/STRATEGY.md).
