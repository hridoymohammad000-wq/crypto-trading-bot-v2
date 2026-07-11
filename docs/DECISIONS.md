# Decisions

This file records only approved project decisions.

## Decision format

- ID:
- Date:
- Title:
- Status: Proposed / Approved / Rejected / Superseded
- Context:
- Decision:
- Evidence:
- Consequences:
- Product Owner approval:

## Approved decisions

### DEC-001 — Governance baseline

- Date: 2026-07-11
- Status: Approved
- Decision: Crypto Trading Bot Development Bible v2.0 — LOCKED governs this project. GitHub is the single source of truth, and one bounded task may be active at a time.
- Consequences: No worker may silently change scope, architecture, or business rules. Test evidence is mandatory before completion claims.
- Product Owner approval: Approved during project foundation setup.

### DEC-002 — Product scope and operating mode

- Date: 2026-07-11
- Status: Approved
- Decision: Build a private, single-owner crypto trading web application for Binance USD-M Futures. V1 is paper/demo only, uses USDT perpetual contracts, operates 24/7, and exposes a one-click Start Bot flow. Manual Run Scan is diagnostic only and may not create orders. Live-money trading is outside V1 and requires separate Product Owner approval after paper validation, testing, and soak testing.
- Consequences: No SaaS or multi-user scope, no live execution path, no mock market data, and no silent fallback.
- Product Owner approval: Approved all in ARCH-001 decision batch 1.

### DEC-003 — Market universe and scanning

- Date: 2026-07-11
- Status: Approved
- Decision: Use active, tradable Binance USD-M USDT perpetual symbols only. Exclude delivery contracts, inactive or settling symbols, and leveraged-token products. Rank dynamically by 24-hour quote volume, scan the Top 20 every five minutes, and emit at most the best 10 qualified signals. Market data uses REST bootstrap plus WebSocket closed-candle updates. Open, missing, stale, duplicate, or malformed candle data may not reach downstream processing. Startup signal generation remains blocked until required history is ready. Existing open or pending exposure blocks a second trade on the same symbol.
- Consequences: The market-data service must provide deterministic readiness, rejection, deduplication, ordering, staleness, and recovery evidence.
- Product Owner approval: Approved all in ARCH-001 decision batch 2.

### DEC-004 — Strategy and signal boundaries

- Date: 2026-07-11
- Status: Approved
- Decision: Support 1-minute scalping execution and 5-minute intraday setup with 5-minute setup plus 1-minute entry confirmation. Initial strategy families are Breakout plus volume confirmation, Pure SMC using MSS/CHoCH with OB/FVG, and Hybrid liquidity sweep plus displacement plus FVG retest. Only closed candles may generate signals. At least one approved strategy must fully match. Only A+ and A signals may become trade candidates. Strategy output is a proposal only and must include symbol, direction, strategy, entry, stop-loss, take-profit, timeframe, confidence, evidence, and rejection reason. Confidence is capped at 0.95. Conflicting directions, incomplete signals, unavailable calculations, and fallback behavior are rejected explicitly.
- Consequences: Exact strategy formulas and thresholds must be defined in later bounded strategy specifications before implementation.
- Product Owner approval: Approved all in ARCH-001 decision batch 3.

### DEC-005 — Risk authority and capital protection

- Date: 2026-07-11
- Status: Approved
- Decision: The Risk Engine is the final execution-approval authority. Risk per trade is 0.5% of account equity for scalping and 1.0% for intraday. Minimum risk-to-reward is 1:1.5 for scalping and 1:2 for intraday. Daily net-loss allocations are 2% for scalping and 3% for intraday, with a combined hard stop at 5%. Daily loss includes realized P&L plus unrealized P&L. Maximum open trades is five. Position size is derived from allowed risk divided by entry-to-stop distance and must respect Binance quantity, step-size, minimum-quantity, and minimum-notional constraints. Duplicate or opposite exposure on the same symbol is blocked. Valid stop-loss protection is mandatory. Insufficient balance or margin rejects the trade without fake success or silent resizing. Daily counters reset at 12:00 AM Asia/Dhaka.
- Consequences: Every risk rejection must expose a deterministic code, reason, calculated risk, required margin, and evidence.
- Product Owner approval: Approved all in ARCH-001 decision batch 4.

### DEC-006 — Paper execution and order lifecycle

- Date: 2026-07-11
- Status: Approved
- Decision: V1 uses paper/demo execution only. Only the Execution Engine may create orders. Breakout signals use paper market entry after closed-candle confirmation; SMC and Hybrid use paper limit entry at approved zones. Orders expire or cancel when setup validity fails. Duplicate signal or order IDs are idempotently rejected. Order states are pending, submitted, filled, cancelled, rejected, or failed. Position states are opening, open, reducing, closed, or emergency-closed. Paper fills use real market data and a configured fill, slippage, and fee model; unconditional fills are prohibited. SL and TP must be attached after fill, with at most two attempts. Protection failure after two attempts triggers emergency close and critical-error state. Entry fill without protection is not success. Every lifecycle transition, rejection, retry, and close reason is persisted. Restart requires restoration and reconciliation before new entries.
- Consequences: Exact slippage values, fee sources, and pending-order expiry windows remain implementation-spec parameters and may not be guessed.
- Product Owner approval: Approved all in ARCH-001 decision batch 5.

### DEC-007 — Trade management, persistence, and reconciliation

- Date: 2026-07-11
- Status: Approved
- Decision: After fill, only the Trade Management Engine manages the position. Stop-loss widening, averaging down, and pyramiding are prohibited. Trailing activates at +1R, with 0.5% distance for scalping and 1.0% for intraday. Long stops may only rise and short stops may only fall. V1 uses one full-position take-profit. Manual close is owner-authorized and audited. Exit reasons are stop_loss, take_profit, trailing_stop, manual_close, risk_shutdown, emergency_close, and setup_invalidated. Close failure receives at most two attempts before critical-error state and new-trade blocking. PostgreSQL is the persistent-state authority. Signals, risk decisions, orders, fills, positions, P&L, fees, bot state, daily counters, rejections, and lifecycle events are persisted. Database timestamps are UTC; UI and daily reset use Asia/Dhaka. Idempotency keys are mandatory. Fill, position, and P&L updates are atomic. Startup and periodic reconciliation are mandatory. Mismatches create explicit incidents and block new trades; silent repair is prohibited. Lifecycle history is append-only.
- Consequences: State must survive restart, and no process-local state may become authoritative.
- Product Owner approval: Approved all in ARCH-001 decision batch 6.

### DEC-008 — System architecture, deployment, security, and monitoring

- Date: 2026-07-11
- Status: Approved
- Decision: Use React, TypeScript, and Vite for the frontend; Node.js and TypeScript for the authoritative controller backend; Python FastAPI for stateless strategy analytics; and PostgreSQL for persistence. The Node backend owns database, market-data coordination, risk, execution, and bot state. Python may not hold exchange credentials, execute orders, or write the database. Frontend, controller, strategy service, and database are separately deployable. Backend services are Docker-compatible. Environments are development, paper-staging, and production-paper. Secrets remain in hosting secret management or environment variables only. The application uses a single owner account, secure password hashing, token or session expiry, CORS allowlisting, rate limiting, input validation, and protected control endpoints. Start, stop, reset, and manual-close actions are audited. Health and monitoring cover controller, strategy service, database, market data, structured logs, latency, stale data, reconciliation, and bot state. Critical incidents block new trades. Telegram notifications are limited to trade entry, trade close, daily report at 12:00 AM Asia/Dhaka, and critical incidents. Production-paper deployment requires unit, integration, end-to-end, and soak tests. Live-money deployment is outside V1. Automated PostgreSQL backup and restore testing are mandatory. Restart remains NOT_READY until health and reconciliation pass. Business and risk logic may not live in the frontend.
- Consequences: Redis is not mandatory in V1 and requires later approval if evidence shows it is needed.
- Product Owner approval: Approved all in ARCH-001 decision batch 7.

## Deferred implementation parameters

The following are not architecture conflicts and must be resolved in later bounded specifications before their owning module is implemented:

- Exact strategy formulas, lookback windows, thresholds, scoring, and invalidation details
- Market-data history depth, stale thresholds, reconnect timing, and reconciliation cadence
- Paper slippage model values, fee source, and pending-order expiry windows
- Margin model, leverage ceiling, and other exchange-specific paper-account parameters
- Deployment provider, backup retention, data retention, and operational alert destinations

No worker may invent these values. Each must be explicitly approved and recorded before implementation.