# Current Task

## Task ID

MDE-001-SPEC

## Title

Market Data Engine Detailed Specification and Test Contract.

## Objective

Define the complete, deterministic Market Data Engine contract for Binance USD-M Futures before implementation begins.

## In scope

- Binance exchange-information and 24-hour ticker sources
- Active USDT perpetual-symbol filtering
- Dynamic Top 20 quote-volume ranking
- Required 1-minute and 5-minute candle history
- REST bootstrap behavior
- WebSocket closed-candle stream behavior
- Candle normalization, ordering, deduplication, and rolling buffers
- Open-candle exclusion
- Startup readiness and downstream processing gate
- Missing, malformed, duplicate, stale, and unverifiable-data rejection
- Reconnect, exponential backoff, and REST gap recovery
- Per-stream freshness and health reporting
- Explicit error and rejection contracts
- Configuration boundaries and validation
- Unit, integration, and recovery test matrix
- Codex-ready implementation scope after Product Owner approval

## Out of scope

- Strategy formulas or implementation
- Risk calculation or implementation
- Paper order execution
- Trade management
- Database schema implementation
- Frontend implementation
- Deployment configuration
- Live-money trading
- Application code changes during this specification task

## Dependencies

- `FOUNDATION-001 — PASS`
- `ARCH-001 — PASS`
- Locked architecture, business rules, and approved decisions on `main`

## Required outputs

- Market-data component boundary
- Exact input and output data contracts
- Required configuration and approved default values
- Readiness, staleness, reconnect, and recovery rules
- Acceptance criteria
- Mandatory test matrix
- Explicit unresolved-decision list, if any
- One bounded Codex implementation task after specification approval

## Completion gate

Implementation may begin only when every behavior required by the Market Data Engine is deterministic, testable, consistent with the locked architecture, and approved without guessed values.

## Execution owner

This specification is prepared in the CEO control chat. Codex is not required until the approved implementation task is ready.