# Architecture

## Status

Core architecture locked by Product Owner approval on 2026-07-11 under `ARCH-001`.

Implementation-specific parameters remain deferred to bounded module specifications and may not be inferred.

## Product boundary

- Private, single-owner crypto trading web application
- Binance USD-M Futures market data
- USDT perpetual contracts only
- V1 paper/demo execution only
- 24/7 operation
- Live-money trading is outside V1 and requires separate Product Owner approval

## Deployable components

### Frontend

- React + TypeScript + Vite
- Displays authoritative API state
- Sends approved control commands only
- Contains no trading, risk, or execution authority

### Controller backend

- Node.js + TypeScript
- Authoritative owner of market-data coordination, database access, bot state, risk approval, paper execution, trade management, reconciliation, authentication, API control, and notifications
- No downstream module may bypass its authority boundaries

### Strategy service

- Python FastAPI
- Stateless analytics only
- Returns strategy proposals and evidence
- Holds no exchange credentials
- Executes no orders
- Writes no database state

### Database

- PostgreSQL
- Single persistent-state authority
- Stores signals, risk decisions, orders, fills, positions, P&L, fees, daily counters, bot state, rejections, incidents, and append-only lifecycle events
- Database timestamps use UTC

### Optional infrastructure

- Redis is not mandatory in V1
- It may be introduced only after evidence, architecture review, and Product Owner approval

## Authority chain

```text
Verified Binance market data
  -> Market Data Engine
  -> Strategy Service proposal
  -> Risk Engine approval or rejection
  -> Paper Execution Engine
  -> Trade Management Engine
  -> PostgreSQL persistence and reconciliation
  -> API, frontend, monitoring, and Telegram notification
```

No module may silently bypass another module's authority.

## Market-data boundary

- REST bootstrap plus WebSocket closed-candle updates
- Dynamic Top 20 active Binance USD-M USDT perpetual symbols by 24-hour quote volume
- Maximum best 10 qualified signals per scan
- Automatic scan every five minutes
- Manual scan is diagnostic only
- Open, missing, stale, duplicate, malformed, or unverifiable data is rejected explicitly
- Signal generation remains blocked until required history is ready

## Strategy boundary

- 5-minute setup plus 1-minute entry confirmation
- Strategy families: Breakout plus volume, Pure SMC, and Hybrid liquidity sweep plus displacement plus FVG retest
- Strategy output is proposal only
- Only A+ and A candidates may continue
- Confidence is capped at 0.95
- Conflicting, incomplete, or failed calculations are rejected explicitly

## Risk boundary

- Risk Engine is the final execution-approval authority
- Risk per trade: 0.5% scalping, 1.0% intraday
- Minimum risk-to-reward: 1:1.5 scalping, 1:2 intraday
- Daily hard stop: combined 5%, including realized and unrealized P&L
- Maximum five open trades
- Duplicate and opposite same-symbol exposure is blocked
- Valid stop-loss is mandatory

## Execution boundary

- V1 uses internal paper/demo execution only
- Real market data drives fill decisions
- Fake success and unconditional fills are prohibited
- Idempotent signal and order handling is mandatory
- Protection attachment receives at most two attempts
- Unprotected filled positions trigger emergency close and critical-error state

## Trade-management boundary

- Trade Management Engine owns filled positions
- Stop-loss widening, averaging down, and pyramiding are prohibited
- Trailing starts at +1R
- Scalping trailing distance: 0.5%
- Intraday trailing distance: 1.0%
- V1 uses one full-position take-profit
- Close failure receives at most two attempts before critical-error state

## Persistence and reconciliation

- Persistent state survives restart
- Fill, position, and P&L updates are atomic
- Idempotency keys are mandatory
- Startup reconciliation must pass before new entries
- Periodic reconciliation is mandatory
- Mismatch creates an explicit incident and blocks new trades
- Silent repair is prohibited
- Daily counters survive restart and reset at 12:00 AM Asia/Dhaka

## Security

- Single owner account
- Secure password hashing and token or session expiry
- CORS allowlist, rate limiting, input validation, and protected control endpoints
- Secrets remain in environment variables or hosting secret management
- Secrets may not enter source control or logs
- Start, stop, reset, and manual-close actions are audited

## Monitoring and recovery

- Structured logs and explicit health endpoints
- Health coverage for controller, strategy service, PostgreSQL, and market-data connectivity
- Metrics for latency, stale data, reconciliation, bot state, and critical failures
- Bot remains `NOT_READY` after restart until health and reconciliation pass
- Critical incidents block new trades
- Telegram notifications are limited to trade entry, trade close, daily report, and critical incident

## Deployment boundary

- Separately deployable frontend, controller backend, strategy service, and PostgreSQL
- Backend services are Docker-compatible
- Environments: development, paper-staging, and production-paper
- Production-paper requires unit, integration, end-to-end, and soak-test PASS
- Automated PostgreSQL backup and documented restore testing are mandatory

## Deferred module specifications

Before implementation, the owning bounded task must define and obtain approval for:

- Exact strategy formulas and thresholds
- Candle history depth and stale-data timing
- Reconnect and gap-recovery timing
- Paper slippage, fees, and pending-order expiry
- Margin and leverage parameters
- Reconciliation cadence, retention, deployment provider, and alert destinations

## Change control

Any core architecture change requires:

1. Conflict check against Crypto Trading Bot Development Bible v2.0 — LOCKED
2. Product Owner approval
3. Decision record in `DECISIONS.md`
4. Updated architecture, business rules, tests, and changelog
5. Verification before implementation or deployment