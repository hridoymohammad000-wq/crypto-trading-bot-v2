# Business Rules

## Status

Core V1 business rules approved and locked by Product Owner on 2026-07-11 under `ARCH-001`.

Implementation-specific values listed as deferred in `DECISIONS.md` may not be invented.

## Product and operating mode

- Private, single-owner application
- Binance USD-M Futures market data
- USDT perpetual contracts only
- V1 paper/demo execution only
- Bot may operate 24/7
- One-click Start Bot runs scan, signal, risk approval, paper execution, and background trade management
- Manual Run Scan is diagnostic only and may not create an order
- Live-money trading requires a separate future approval

## Market universe and data

- Use only active, tradable Binance USD-M USDT perpetual symbols
- Exclude delivery contracts, inactive or settling symbols, and leveraged-token products
- Rank by 24-hour quote volume and scan the dynamic Top 20
- Automatic scanning runs every five minutes
- Emit at most the best 10 qualified signals per scan
- Use REST bootstrap plus WebSocket closed-candle updates
- Open candles may not be used for signal generation
- Missing, stale, duplicate, malformed, or unverifiable data must be rejected explicitly
- Signal generation is blocked until required history is ready

## Strategy and signal rules

- Use 5-minute setup plus 1-minute entry confirmation
- Initial strategy families are Breakout plus volume confirmation, Pure SMC, and Hybrid liquidity sweep plus displacement plus FVG retest
- At least one approved strategy must fully match
- Only A+ and A candidates may continue
- Strategy output is a proposal and never authorizes execution
- Required signal fields: symbol, direction, strategy, entry, stop-loss, take-profit, timeframe, confidence, evidence, and rejection reason
- Confidence is capped at 0.95
- Conflicting directions, incomplete signals, calculation failure, and silent fallback are rejected

## Risk rules

- Risk Engine is the final execution-approval authority
- Scalping risk per trade: 0.5% of account equity
- Intraday risk per trade: 1.0% of account equity
- Minimum scalping risk-to-reward: 1:1.5
- Minimum intraday risk-to-reward: 1:2
- Scalping daily loss allocation: 2%
- Intraday daily loss allocation: 3%
- Combined hard daily loss limit: 5%
- Daily loss includes realized plus unrealized P&L
- Combined hard limit stops the bot completely
- Maximum open trades: five
- Position size is based on allowed risk divided by entry-to-stop distance
- Quantity must satisfy exchange step size, minimum quantity, and minimum notional constraints
- A second pending or open trade on the same symbol is prohibited
- Opposite same-symbol exposure is prohibited
- Valid stop-loss protection is mandatory
- Insufficient balance or margin rejects the trade without silent resizing or fake success
- Risk rejections must include code, reason, calculated risk, required margin, and evidence
- Daily counters reset at 12:00 AM Asia/Dhaka and must survive restart

## Paper execution rules

- Only the Execution Engine may create orders
- Breakout uses paper market entry after closed-candle confirmation
- SMC and Hybrid use paper limit entry at approved zones
- Orders cancel when setup invalidates or approved expiry is reached
- Duplicate signal or order IDs must be idempotently rejected
- Order states: pending, submitted, filled, cancelled, rejected, failed
- Position states: opening, open, reducing, closed, emergency-closed
- Paper fills must use real market data and approved fill, slippage, and fee rules
- Unconditional fills and fake success are prohibited
- SL and TP must be attached after fill
- Protection attachment receives at most two attempts
- Protection failure after two attempts triggers emergency close and critical-error state
- Entry fill without valid protection is not success
- Existing pending or open same-symbol exposure blocks a new entry
- Every lifecycle transition, rejection, retry, and close reason must be persisted

## Trade-management rules

- Only Trade Management Engine manages filled positions
- Stop-loss widening is prohibited
- Trailing activates at +1R
- Scalping trailing distance: 0.5%
- Intraday trailing distance: 1.0%
- Long stops may only rise; short stops may only fall
- V1 uses one full-position take-profit
- Averaging down is prohibited
- Pyramiding is prohibited
- Owner manual close is allowed and audited
- Exit reasons: stop_loss, take_profit, trailing_stop, manual_close, risk_shutdown, emergency_close, setup_invalidated
- Close failure receives at most two attempts before critical-error state and new-trade blocking

## Persistence and reconciliation rules

- PostgreSQL is the persistent-state authority
- Persist signals, risk decisions, orders, fills, positions, P&L, fees, bot state, daily counters, rejections, incidents, and lifecycle events
- Database timestamps use UTC
- UI and daily reset use Asia/Dhaka
- Signal, order, fill, and lifecycle events require unique idempotency keys
- Fill, position, and P&L updates must be atomic
- Startup reconciliation must pass before new entries
- Periodic reconciliation is mandatory
- Mismatches create explicit incidents and block new trades
- Silent mismatch repair is prohibited
- Lifecycle history is append-only

## Security, monitoring, and notification rules

- Frontend contains no business, risk, or execution authority
- Python strategy service holds no exchange credentials, executes no orders, and writes no database state
- Secrets remain outside the repository and logs
- Start, stop, reset, and manual-close actions are audited
- Critical incidents block new trades
- Telegram notifications are limited to trade entry, trade close, daily report at 12:00 AM Asia/Dhaka, and critical incident
- Bot remains NOT_READY after restart until health and reconciliation pass
- Production-paper deployment requires unit, integration, end-to-end, and soak-test PASS
- Live-money deployment is prohibited in V1

## Mandatory quality rules

- No business rule may be inferred from previous projects without explicit Product Owner approval
- No mock market data, fake order result, or silent fallback is permitted in the production path
- Invalid, incomplete, stale, or unverifiable data must be rejected with an explicit reason
- Every approved rule requires acceptance criteria and mandatory tests before implementation is considered complete