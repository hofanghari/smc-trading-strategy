# SMC Agent Knowledge Pack — H4/H1 Bias → M15 Execution
**Version:** 1.0  
**Purpose:** Compact, retrieval-friendly operating knowledge for Antigravity agents and trading bots.  
**Source:** `smc_master_sop_h4_h1_m15.md`  
**Use:** Load as strategy reference; do not treat this document as proof of profitability.

## 0. Non-negotiable operating contract

1. **H4 = macro structure. H1 = directional bias, POIs and liquidity draw. M15 = execution trigger.**
2. **Do not execute from a setup name alone.** Validate every required condition and the risk gate.
3. **No complete confirmation = no trade.** Return `NO_TRADE` and list failed/missing conditions.
4. Separate observed facts from inferred structure. If swing points, POI boundaries, or candle confirmation cannot be determined reliably, return `INSUFFICIENT_DATA`.
5. Treat all risk limits as hard constraints. Never raise risk to recover losses.
6. This pack describes a strategy framework, not a guarantee of win rate or returns.

## 1. Timeframe responsibilities

| Timeframe | Responsibilities | Must produce |
|---|---|---|
| H4 | Macro BOS/CHoCH, major swing, protected high/low, extreme POIs, EQH/EQL and major liquidity | `h4_bias`, key structure levels, major POIs/liquidity |
| H1 | Intermediate bias, H1 OB/FVG, premium/discount, liquidity draw and target | `h1_bias`, active POI, swing range, target liquidity |
| M15 | POI reaction, inducement/sweep, CHoCH or CISD, displacement/FVG/iFVG, entry/SL | `trigger_status`, entry plan, invalidation, R:R |

**Synchronization:** H4/H1 provide context and permission; M15 alone triggers execution. The source checklist requires H1 and H4 structure to agree. If they conflict, do not force a trade; return `NO_TRADE` unless a separately approved rule explicitly handles conflict.

## 2. Canonical definitions

- **BOS:** Candle body closes beyond the nearest relevant swing high in an uptrend or swing low in a downtrend; continuation signal.
- **CHoCH:** Candle body closes through the swing level that supported the prior BOS impulse; possible directional change.
- **Protected High/Low:** Swing origin of a BOS impulse. A break invalidates the associated structure thesis.
- **Order Block (OB):** Last opposite-colour candle before a strong displacement that both creates an FVG and breaks structure (BOS/CHoCH).
- **Extreme POI:** OB/FVG at the extreme/origin of the H1/H4 impulse.
- **Decision POI:** OB/FVG mid-impulse associated with the next structure break; require M15 CHoCH/CISD confirmation.
- **FVG:** Three-candle imbalance where candle 1 and candle 3 wicks do not overlap.
- **iFVG:** Failed/breached FVG that may reverse role on a retest (support ↔ resistance).
- **BSL:** Liquidity above prior highs, EQH, PDH/PWH, or session highs.
- **SSL:** Liquidity below prior lows, EQL, PDL/PWL, or session lows.
- **Liquidity sweep:** Price trades through a known liquidity level and rejects back; a wick alone is not sufficient evidence of reversal without confirmation.
- **IDM / inducement:** Internal swing likely to attract early entries before price reaches the intended POI.
- **Premium / Discount:** Relative to the selected H1/H4 swing range midpoint (Fib 0.5). Premium → seek SHORT; Discount → seek LONG.
- **OTE:** Fibonacci 0.618–0.786 zone.
- **CISD:** M15 candle closes through the open of the originating candle for the move that led to the sweep, indicating delivery/momentum shift.
- **Double sweep:** First sweep of external H1/H4 liquidity, then a second sweep of internal M15 liquidity before the confirmed move.

## 3. Standard decision pipeline

### Step A — Build higher-timeframe context
1. Identify relevant confirmed H4 and H1 swing highs/lows and BOS/CHoCH.
2. Determine directional bias and whether H4/H1 agree.
3. Select the active H1/H4 swing range and its 0.5 midpoint.
4. Mark unmitigated OB/FVG POIs; label each `EXTREME` or `DECISION`.
5. Mark external liquidity and the likely target (`BSL` for long targets, `SSL` for short targets, subject to structure).

### Step B — Wait for location
6. Wait until price reaches a valid H1/H4 POI. Do not enter simply because price is in premium/discount.
7. Record nearby internal M15 liquidity/IDM and whether the POI has already been mitigated.

### Step C — Require M15 confirmation
8. Confirm a sweep of relevant M15 SSL for a long or BSL for a short (or the specific external sweep required by a reversal setup).
9. Require a **closed M15 candle** to confirm CHoCH or CISD in the intended direction.
10. Confirm a resulting displacement and usable M15 FVG/iFVG/OB for entry refinement.
11. If confirmation is absent, contradictory, or arrives after the POI thesis is invalidated: `NO_TRADE`.

### Step D — Construct and validate the trade
12. Entry: M15 FVG/OB retest or an explicitly defined market trigger after candle close.
13. Stop: beyond the relevant sweep extreme with a volatility/market-appropriate buffer.
14. Target: relevant H1 liquidity or opposing POI; compute R:R from actual proposed entry, stop and target.
15. Reject if projected R:R is below the active minimum or if any risk gate fails.
16. Size the position from allowed cash risk and actual loss per unit at the stop, including fees/slippage where supported by the execution system.
17. Before execution, emit the full audit record in §7.

## 4. Setup catalogue (deterministic rules)

### `SMC_A_PLUS_M15_01` — H1 POI + M15 confirmation
**LONG requirements — all must be true**
- H1 bias bullish (HH/HL); H4/H1 structure alignment passes.
- Price is in H1 Discount (< 0.5 of the selected H1 swing).
- Price touches an unmitigated H1 demand OB or bullish FVG.
- M15 sweeps SSL / inducement low.
- M15 body closes above the recent M15 swing high (bullish CHoCH) and leaves bullish FVG.
**Entry:** Buy limit at the top boundary of bullish M15 FVG, or market only after the defined FVG-retest trigger.  
**SL:** Below the lowest sweep wick + buffer.  
**TP:** TP1 at M15 swing high (source says target 1:2); TP2 at H1 BSL/EQH (source says target ≥1:3).

### `SMC_SWEEP_CISD_M15_02` — Major sweep + CISD reversal
**SHORT requirements — all must be true**
- Price sweeps H1 BSL such as EQH, PDH or Asia high.
- Price rejects back into the prior range.
- Bearish M15 CISD: body closes below the open of the bullish expansion candle that led to the sweep.
- Bearish M15 FVG or bearish iFVG forms.
**Entry:** Market short after CISD candle close, or sell limit at the M15 iFVG.  
**SL:** Above sweep high + buffer.  
**TP:** H1 SSL / Asian low; source target range 1:2.5–1:4.

### `SMC_DOUBLE_SWEEP_M15_03` — External then internal sweep
**LONG requirements — all must be true**
- Price enters H1 demand and first sweeps H1 SSL.
- Price rebounds and forms internal M15 lows likely to attract early buyers' stops.
- Price returns and sweeps those internal lows inside the H1 extreme OB.
- Strong bullish M15 displacement creates bullish FVG.
**Entry:** Buy limit at FVG created after the second sweep.  
**SL:** Below the second sweep low + buffer.  
**TP:** TP1 at nearby M15 liquidity; TP2 at H1 major high; source target ≥1:3.

### `SMC_INVERSION_FVG_M15_04` — iFVG continuation retest
**LONG requirements — all must be true**
- H1 is expanding strongly upward.
- A bearish M15 FVG is breached and an M15 bullish displacement candle closes above it.
- Price retests the upper boundary of the inverted FVG.
- M15 rejection is visible (e.g. long lower wick or bullish engulfing).
**Entry:** Buy market only after retest rejection confirmation.  
**SL:** Below the lower boundary of the inversion FVG + buffer.  
**TP:** Next H1 resistance / unmitigated supply; source target 1:2–1:3.

**Direction coverage warning:** The source defines the long side for setups 1, 3 and 4, and the short side for setup 2. Do not automatically mirror these into opposite-direction setups without separate validation and approval.

## 5. Risk engine

### Limits specified by the source
- Maximum account risk per trade: **1.5%**.
- Daily drawdown circuit breaker: **4%**.
- Minimum acceptable R:R: **2.0 (1:2)**.
- Maximum correlated open positions: **2** when correlation exceeds **0.8**.
- Move SL to breakeven + spread buffer at **+1.5R**.
- Close **50% at +2R**; let the remaining 50% target H1 BSL/SSL with trailing stop.
- After TP1, trail behind M15 swing lows/highs or M15 20 EMA.
- After **2 consecutive M15 losses in the same session**, stop trading for **6 hours**.

### Position sizing principle
Use:
`position_size = allowed_cash_risk / estimated_loss_per_unit_at_stop`

Where:
`allowed_cash_risk = current_account_equity × risk_fraction`

The original file also gives `Lot_Size = (Account_Balance × Risk_Percentage) / (Stop_Loss_Pips × Pip_Value)` and an XAUUSD example. Adapt units, contract multiplier, tick/pip value, fees and slippage to the actual instrument/exchange; never apply a Forex pip formula directly to crypto contracts.

### Risk gates
- Reject trades with R:R below the configured minimum.
- Reject new entries when daily drawdown breaker is active.
- Reject entries exceeding correlated-position limit.
- Reject if stop distance, contract multiplier, quantity step, tick size or loss-at-stop cannot be calculated reliably.
- Risk must be computed from current equity/balance using the execution environment's approved policy, not hard-coded lot sizes.

## 6. Pre-trade checklist (all TRUE)

1. **Trend alignment:** H1 and H4 structure agree with the proposed direction.
2. **Location:** Long in Discount; Short in Premium, based on a clearly defined swing range.
3. **POI validity:** Price touched a still-valid, unmitigated H1/H4 OB/FVG.
4. **Liquidity event:** Required M15 sweep/IDM condition occurred.
5. **M15 confirmation:** Closed candle confirms CHoCH or CISD in the intended direction.
6. **R:R:** Entry → SL → TP1 meets the configured minimum.
7. **Risk engine:** Position size, correlated exposure, daily DD and session-loss controls all pass.
8. **Order validity:** Entry/SL/TP levels are valid for the symbol's tick/price rules and order type.

If any item fails, do not place an order. Return the failed checks and `NO_TRADE`.

## 7. Required agent output schema

For each scan, produce a compact machine-readable record:

```yaml
timestamp_utc: ""
symbol: ""
execution_mode: "SIGNAL_ONLY | PAPER | LIVE"
h4_bias: "BULLISH | BEARISH | RANGE | UNKNOWN"
h1_bias: "BULLISH | BEARISH | RANGE | UNKNOWN"
structure_alignment: "PASS | FAIL | UNKNOWN"
swing_range:
  low: null
  high: null
  midpoint_0_5: null
price_zone: "DISCOUNT | PREMIUM | MIDPOINT | UNKNOWN"
poi:
  timeframe: "H1 | H4"
  type: "OB | FVG | iFVG | NONE"
  class: "EXTREME | DECISION | UNKNOWN"
  low: null
  high: null
  mitigated: null
liquidity:
  target: "BSL | SSL | UNKNOWN"
  sweep_seen: false
  sweep_level: null
m15_confirmation:
  choch: false
  cisd: false
  displacement: false
  fvg_type: "BULLISH | BEARISH | NONE"
setup_id: ""
decision: "NO_TRADE | CANDIDATE | APPROVED"
entry: null
stop_loss: null
take_profit_1: null
take_profit_2: null
risk_percent: null
position_size: null
projected_rr_tp1: null
projected_rr_tp2: null
failed_gates: []
reasoning_summary: ""
```

Only `APPROVED` means all strategy and risk gates passed. A candidate is not an order. The execution layer must independently enforce its own safety and order-validation rules.

## 8. Source ambiguities that must not be silently guessed

1. **R:R threshold conflict:** General risk engine says minimum 1:2; workflow says target ≥1:2.5; setup targets vary (1:2, ≥1:3, 1:2.5–1:4). Until configured by the system owner, use **1:2 as the hard minimum** and preserve each setup's stated target as its preferred target; do not claim all setups require 1:2.5.
2. **Stop buffer units:** Source says “1–2 pips” for several setups, but the asset universe includes crypto, Forex, gold and indices. Convert buffer to symbol-appropriate ticks/volatility units; do not assume universal pip size.
3. **H4/H1 conflict:** The checklist requires agreement but no conflict-resolution rule is defined. Default to `NO_TRADE`.
4. **CISD reference candle:** Implementation must define exactly how the originating expansion candle is selected; if ambiguous, return `INSUFFICIENT_DATA`.
5. **Swing/Fibonacci anchoring:** Source does not fully define swing-selection algorithms or tie-breaking. Use the project's approved swing detector; do not redraw the range opportunistically to justify a trade.
6. **Order fills and partial exits:** Source gives limit/market concepts but does not specify fill simulation, partial-fill handling, fee/slippage accounting, or exchange-specific order constraints. These belong to the execution engine and must be tested separately.
7. **“Highest win rate” language:** The source describes Extreme POI as having the highest win rate, but provides no supporting performance dataset in this file. Treat this as a hypothesis, not a proven statistic.

## 9. Suggested implementation safeguards (engineering recommendations, not source claims)

- Evaluate signals using completed candles unless a rule explicitly supports intrabar processing.
- Prevent look-ahead bias: do not use future candles to confirm BOS/CHoCH/FVG or to simulate fills.
- Log every gate, price level, candle timestamp and rejection reason for later audit.
- Backtest with fees, slippage, funding where applicable, realistic limit-order fills and out-of-sample periods.
- Keep strategy signal generation separate from order execution and risk enforcement.
- Start in `SIGNAL_ONLY` or `PAPER`; do not enable live orders without explicit owner approval and separate validation.
- Measure expectancy, profit factor, drawdown, trade count, exposure and robustness across symbols/regimes. Do not optimize solely for win rate.
- Store the strategy version/config used for every decision so results are reproducible.

## 10. Minimal agent instruction

> Apply this SOP in order: H4/H1 context → valid POI → M15 sweep → closed-candle CHoCH/CISD → FVG/iFVG entry refinement → R:R and risk gates → structured audit output. Never invent missing levels or confirmations. If any required fact is unavailable or any gate fails, return `NO_TRADE` or `INSUFFICIENT_DATA` with explicit reasons. Treat the strategy as a hypothesis to validate, not a promise of profitability.
