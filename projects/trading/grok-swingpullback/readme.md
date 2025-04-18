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


To visually represent the `PriceActionPullbackStrategy`, I’ll create a concise flowchart using Mermaid syntax. This diagram will illustrate the decision-making process for entering and exiting trades, capturing the key steps: confirming an uptrend, identifying a pullback within 0.1% of a swing low, verifying candlestick patterns, entering on breakout, and exiting based on take-profit, stop-loss, bearish signals, or the 4 PM daily close. Mermaid is ideal for this as it’s text-based, clear, and renders well in supported environments (e.g., GitHub, online Mermaid editors). Below is the diagram code, followed by a brief explanation and instructions for rendering it.

### Mermaid Flowchart for PriceActionPullbackStrategy
```mermaid
graph TD
    A[Start: 15-min Chart] --> B{Uptrend? <br> Higher highs/lows <br> over 5 bars}
    B -->|No| A
    B -->|Yes| C{Pullback to Swing Low? <br> Low ≤ Swing Low × 1.001 <br> (0.1% range)}
    C -->|No| A
    C -->|Yes| D{Candlestick Pattern? <br> Bullish Engulfing or Doji <br> Range ≥ 20 ticks}
    D -->|No| A
    D -->|Yes| E{Breakout? <br> Next bar closes > <br> Pattern bar high}
    E -->|No| A
    E -->|Yes| F[Enter Long: <br> At breakout bar close]
    F --> G[Set Orders: <br> SL = Swing Low - 30 ticks <br> TP = Entry + 2×(Entry - SL)]
    G --> H{Monitor: <br> Check each 15-min bar}
    H -->|Take-Profit Hit| I[Exit: Take-Profit]
    H -->|Stop-Loss Hit| J[Exit: Stop-Loss]
    H -->|Bearish Engulfing| K[Exit: Bearish Signal]
    H -->|Price < Swing Low| L[Exit: Swing Low Break]
    H -->|Time = 4 PM| M[Exit: Daily Close <br> (15:45 bar close)]
    I --> N[Log Trade: <br> Prices, Reason, P/L]
    J --> N
    K --> N
    L --> N
    M --> N
    N --> A
```

### Explanation
- **Flow**: The diagram starts with a 15-minute chart and checks conditions sequentially: uptrend, pullback within 0.1%, candlestick pattern, and breakout. If any condition fails, it loops back to monitoring. After entering a trade, it sets stop-loss (30 ticks below swing low) and take-profit (2:1 reward-to-risk), then monitors for exit triggers (take-profit, stop-loss, bearish engulfing, swing low break, or 4 PM close). All paths end with logging the trade.
- **Key Steps**:
  - **Uptrend**: Ensures higher highs/lows over 5 bars.
  - **Pullback**: Price must be ≤ Swing Low × 1.001 (e.g., for SI at 29.400, ≤29.4294).
  - **Candlestick**: Bullish engulfing or doji, range ≥ 20 ticks (e.g., 0.20 for SI, 5.0 for NQ).
  - **Breakout**: Next bar’s close above pattern high.
  - **Exits**: Multiple conditions checked each bar, including 4 PM daily close.
- **Conciseness**: The diagram is streamlined to fit manual execution needs while covering all rules from the README.

### Rendering Instructions
Since I can’t directly render images, you can visualize the flowchart using these methods:
1. **Online Mermaid Editor**:
   - Visit [mermaid.live](https://mermaid.live/).
   - Copy-paste the Mermaid code above into the editor.
   - The flowchart will render instantly. You can export it as PNG/SVG for your notes.
2. **GitHub or Compatible Platforms**:
   - If you’re documenting in a GitHub README, paste the code within triple backticks (```mermaid ... ```).
   - GitHub will render it in preview or when published.
3. **Local Tools**:
   - Use a Mermaid-compatible tool like VS Code with the Mermaid extension.
   - Paste the code and export the diagram.

### Example Context
For SIMAY25 (8 Apr chart):
- Swing low at 29.400 → 0.1% = 29.4294.
- Bullish engulfing at 29.410 (within range), range 0.30 (>20 ticks).
- Breakout bar closes at 29.41.
- Trade: Enter at 29.41, SL at 29.10 (29.400 - 0.30), TP at 30.03 (29.41 + 0.62).
- Monitor until TP, SL, bearish signal, swing low break, or 4 PM.

### Notes
- **Alternative Formats**: If Mermaid isn’t ideal, I can describe a text-based ASCII diagram or suggest another tool (e.g., PlantUML, Graphviz). However, Mermaid is lightweight and widely supported.
- **Customization**: If you want specific labels (e.g., instrument examples like “SI: 0.20 range”) or a simpler flow (e.g., fewer exit paths), I can adjust the code.
- **Integration**: Save the rendered diagram alongside the README for quick reference during manual trading.

Try rendering the Mermaid code in [mermaid.live](https://mermaid.live/) to see the flowchart. If you need help with rendering, a different diagram style, or adding specific examples (e.g., NQJUN25 trade path), let me know!