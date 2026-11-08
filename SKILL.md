# SMC Trading Strategy & Data Integrity

## Summary

A quantitative Smart Money Concepts (SMC) and Price Action decision engine for multi-timeframe market analysis, deterministic setup validation, risk management, strict candlestick data integrity, and Binance Futures bot architecture.

## When to Use

- Analyzing trade setups across H4, H1, M15, M5, or M1 timeframes using SMC, liquidity sweeps, CISD, Order Blocks, FVG/iFVG, or Volume Profile.
- Auditing or debugging SMC scanner pipelines, OHLCV data feeds, or TradingView price alignment.
- Evaluating trade entries, Stop Loss (SL), Take Profit (TP), Risk-to-Reward (R:R), and position sizing before issuing a signal.
- Generating structured trade audit records (`APPROVED`, `NO_TRADE`, `INSUFFICIENT_DATA`, `REJECT`).
- Architecting or coding modular Binance Futures trading bots, Paper Trading engines, or quantitative strategies (Hedge Mode, Sentiment, ML/LSTM, Market Making, Arbitrage).

## Operating Contract & Invariants

1. **Market Data Feed Decoupling:** Always source OHLCV market data from production live feeds (e.g., `fapi.binance.com`), even when executing in paper or testnet mode. Never use sandbox/testnet market data for technical analysis.
2. **Confirmed Closed Candle Rule:** Calculate all macro/intermediate structures (BOS, CHoCH, Swing Points, OB, FVG, Liquidity Sweeps) exclusively on closed candles (`df.iloc[:-1]`). Use the forming candle's current price only for distance checks or live execution triggers.
3. **Data Freshness & OHLCV Math Gate:** Before evaluating setups, verify that `High >= max(Open, Close)`, `Low <= min(Open, Close)`, `Volume >= 0`, timestamps are strictly ascending without duplicates, and the latest candle is fresh (not stale/expired).
4. **No Complete Confirmation = No Trade:** Never infer missing levels, fabricate volume/Order Flow metrics, or assume profitability without empirical backtest validation. Return `NO_TRADE` or `INSUFFICIENT_DATA` if any mandatory condition fails.
5. **Default Signal-Only Mode:** Output trade evaluations as structured signals unless external execution permissions are explicitly granted.

## Decision Pipeline

### Step 1: Data Integrity Pre-Flight
- Verify data source is live production and confirm OHLCV values match reference market data.
- Drop unclosed higher-timeframe candles before structure detection.
- If data is stale, missing, or mathematically invalid, halt and return `INSUFFICIENT_DATA`.

### Step 2: Higher-Timeframe Context (H4 / H1)
- Identify confirmed 5-candle Swing Highs/Lows, BOS, CHoCH, and Protected Highs/Lows.
- Establish `h4_bias` and `h1_bias`. Require H4 and H1 structural alignment unless a specific override rule is defined.
- Define the active H1/H4 dealing range and its 0.5 Fibonacci midpoint (`Discount` < 0.5 for LONG; `Premium` > 0.5 for SHORT).
- Map unmitigated `EXTREME` and `DECISION` Points of Interest (OB/FVG) and external liquidity targets (`BSL` / `SSL`).

### Step 3: Intermediate & Micro Confirmation (M15 / M5 / M1)
- Wait for price to reach an unmitigated H1/H4 POI. Do not enter solely based on Premium/Discount zones.
- Confirm liquidity clearance (Single Sweep, Double Sweep, or Inducement clearance).
- Require a **closed execution candle** confirming CHoCH, CISD (body closing beyond the originating expansion candle's open), or strong rejection (Pinbar/Engulfing) with displacement and FVG/iFVG formation.

### Step 4: Setup Matching & Validation
Match the observed price action against deterministic setups in `references/setups_and_risk.md`.
- Do not automatically mirror LONG-only setups into SHORT setups without explicit rules.
- Validate all mandatory setup conditions separately from supporting confluence factors.

### Step 5: Risk Engine & Structured Audit Output
- Calculate `Risk Amount = Risk Capital * Risk Percentage` (using actual drawable/risk capital, capped at 1.0%-1.5% per trade).
- Compute `Position Size = Risk Amount / Estimated Loss Per Unit at SL` using instrument-specific tick/contract multipliers and estimated fees/slippage.
- Enforce hard risk gates: minimum R:R >= `1:2.0` after costs, daily drawdown circuit breaker (`4.0%`), max 2 correlated open positions (`> 0.8` correlation), and 6-hour pause after 2 consecutive session losses.
- Output the full YAML audit record defined in `references/setups_and_risk.md`.

## Bot Architecture & Paper Trading Implementation

When designing or implementing a Binance Futures trading bot:
- **Modular Structure:** Separate configuration (`config.py`), exchange connection (`exchange_client.py` with `ccxt`/`python-binance` and Hedge Mode support), signal generation (`strategy.py`), simulated execution (`paper_trader.py` with 0.04% Taker fee accounting and `trade_history.json` logging), risk controls (`risk_manager.py`), and alerting (`logger_notifier.py`).
- **Live Data + Paper Execution:** Always pair live production OHLCV feeds (`fapi.binance.com`) with the local `paper_trader.py` simulator rather than relying on Binance Testnet candles.
- Consult `references/binance-crypto-trading-bot-guide.md` for quantitative strategy archetypes (Hedge Mode Crossover, Top Trader Long/Short Ratio, LSTM/Random Forest, Passivbot Market Making, Market-Neutral Arbitrage), recommended Python libraries, and open-source GitHub references.

## Reference Files

- `references/setups_and_risk.md`: Deterministic setup rules (SMC A+, Double Sweep + CISD, Level Reclaim + IFVG, Extreme OB + IDM, Volume Profile POC, EMA/RSI/MACD/Vix Fix filters), postmortem candlestick checks, and final YAML trade record schema.
- `references/binance-crypto-trading-bot-guide.md`: Binance Futures bot module architecture, paper trading workflow, quantitative strategy archetypes, Python stack, and GitHub reference directory.

## Gotchas

- **Forex Pips vs. Crypto Ticks:** Never apply fixed Forex pip buffers (e.g., "1-2 pips") directly to crypto futures, gold, or indices. Always convert stop buffers to symbol-appropriate tick sizes or ATR/volatility units.
- **Unclosed Forming Bars:** A temporary wick on an unclosed H1/M15 candle frequently causes false BOS/Sweep signals (repainting). Always wait for candle close.
- **Arbitrary TP Stretching:** If a setup's natural structural target yields an R:R below `1:2.0`, reject the trade rather than artificially extending the Take Profit target.
