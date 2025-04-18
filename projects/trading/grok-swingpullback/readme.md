Below is a concise README for the `PriceActionPullbackStrategy`, designed to be short, clear, and comprehensive for manual trading. It covers all key elements, including the 0.1% pullback criterion, entry/exit rules, and daily close at 4 PM, tailored for your instruments (SIMAY25, CLMAY25, NQJUN25, GCJUN25, YMJUN25).


# PriceActionPullbackStrategy README

## Overview
A 15-minute chart strategy for long trades in uptrends, targeting pullbacks to swing lows with bullish candlestick patterns. Exits at take-profit, stop-loss, bearish signals, or daily close (4 PM).

## Instruments
- SIMAY25 (Silver): $5000/unit, tick = 0.01
- CLMAY25 (Crude Oil): $1000/unit, tick = 0.01
- NQJUN25 (Nasdaq): $20/point, tick = 0.25
- GCJUN25 (Gold): $100/unit, tick = 0.1
- YMJUN25 (Dow): $5/point, tick = 0.25

## Entry Rules
1. **Uptrend**: Confirm higher highs/lows over 5 bars (75 mins).
2. **Pullback**: Price reaches within 0.1% of swing low (lowest low, 5-10 bars).
   - Calculate: Swing Low × 1.001 = Upper Bound.
   - Example: SI at 29.400 → 29.4294.
3. **Candlestick**: Bullish engulfing (green engulfs red) or doji (open ≈ close).
   - Range ≥ 20 ticks: SI/CL (0.20), NQ/YM (5.0), GC (2.0).
4. **Breakout**: Next bar closes above pattern bar’s high.
5. **Enter**: Long at breakout bar’s close.

## Exit Rules
- **Stop-Loss**: 30 ticks below swing low (e.g., SI: 0.30, NQ: 7.5).
- **Take-Profit**: 2:1 reward-to-risk (e.g., risk 0.31 → TP = Entry + 0.62).
- **Other Exits**: Exit at earliest of:
  - Bearish engulfing (red engulfs green).
  - Price breaks below swing low.
  - Daily close (4 PM, 15:45 bar close).

## Manual Execution
1. **Chart**: 15-min, candlestick view, mark 4 PM.
2. **Identify Uptrend**: Check 5-bar higher highs/lows.
3. **Spot Pullback**: Low of candle ≤ Swing Low × 1.001.
4. **Confirm Pattern**: Bullish engulfing/doji, range ≥ 20 ticks.
5. **Enter on Breakout**: Long when next bar closes above pattern high.
6. **Set Orders**: Stop-loss (30 ticks below), take-profit (2:1).
7. **Monitor**: Exit at take-profit, stop-loss, bearish signal, or 4 PM.
8. **Log**: Record entry/exit prices, reason, profit/loss.

## Notes
- Calculate 0.1% for pullback: e.g., NQ 17191.50 → ≤17208.6915.
- Risk varies: SI ($1550), CL ($310), NQ (~$350), GC (~$340), YM (~$87.50).
- Backtest to assess losses in choppy/downtrend markets.
- Use micro contracts (e.g., 0.2 lots SI) to cap risk at ~$300.

 

This README is compact yet covers all strategy essentials for manual trading. You can note it down or save it as a reference. If you need tweaks (e.g., adding short trades, simplifying terms, or a printable version), let me know!