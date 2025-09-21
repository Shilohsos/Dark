# Indicator: OB + BOS + CHOCH + Reversal/Continuation (Pine v5)

## Non-repainting rules
- Use `ta.pivot*` for swings: `ph=ta.pivothigh(high, L, L)` and `pl=ta.pivotlow(low, L, L)`.
- A signal may only be confirmed on `barstate.isconfirmed`.
- No `request.security` repaint: use `barmerge.gaps_on`, and confirm HTF values at close.

## Inputs
- General: `tfHTF` (string, default ""), `swingLen` (int, default 3..10), `lookback` (bars), `maxBoxes` (int).
- OB: `obType` ("both|bullish|bearish"), `obLookback` (bars), `obBodyPct` (0–100) = min body %.
- BOS/CHOCH: `bosWickOK` (bool), `closeOnly` (bool), `bosMinDistATR` (float ATR multiples).
- Reversal/Continuation: `confirmWith` ("none|rsi|atrVol"), `rsiLen` (14), `atrLen` (14), `atrMin` (eg 0.5×).
- Visuals: toggles for boxes/lines/labels, colors, `showDashboard`.

## Definitions (working)
- Swing high/low: confirmed `ph`/`pl`.
- Structure high/low: latest confirmed swing HH/LL queues.
- BOS (break of structure): close (or wick if `bosWickOK`) beyond last opposite swing (HH for bear BOS, LL for bull BOS). If the new break flips trend vs prior structure → **CHOCH**.
- Order Block (OB): last opposing candle **before** an impulsive move that makes BOS.  
  - Bull OB: last bearish candle body before a bull BOS; body % ≥ `obBodyPct`.  
  - Bear OB: last bullish candle body before a bear BOS.  
  - OB box extends until invalidation (box tapped and closed through or BOS the other way).
- Reversal vs Continuation:  
  - **Reversal**: CHOCH just printed + price returns to test the fresh OB and rejects (close back inside + rejection wick).  
  - **Continuation**: BOS in trend direction + price returns to the origin OB and rejects.

## Signals
- Long: (Reversal OR Continuation) in bull context, non-repainting confirmation at bar close.  
- Short: symmetric.

## Outputs
- Draw OB boxes (max `maxBoxes`) with labels.  
- Draw BOS and CHOCH lines/labels at the break bar.  
- Plot last swing HH/LL levels.  
- Alerts: `LongEntry`, `ShortEntry`, `OBMitigated`, `BOS`, `CHOCH`.

## Pseudocode
1. Compute swings (ph/pl) with `swingLen`. Maintain arrays for last HH/LL.
2. Detect BOS/CHOCH on confirmed bars vs stored HH/LL. Update trend state.
3. When BOS confirmed, register the qualifying OB candle (per rules) and start a box (`var box[]`).
4. OB invalidation: close fully beyond opposite edge; mitigation: touch + rejection close back.
5. Reversal/Continuation conditions per definition (+ optional RSI/ATR filters).
6. On signal: place label/arrow; fire `alertcondition`.
