# Business Rules

## Status

No trading strategy, risk threshold, execution rule, symbol universe, timeframe, or exchange configuration is approved in this repository yet unless explicitly recorded here and in `DECISIONS.md`.

## Locked governance rules

- No business rule may be inferred from previous projects without explicit Product Owner approval.
- No mock market data, fake order result, or silent fallback is permitted in the production path.
- Invalid, incomplete, stale, or unverifiable data must be rejected with an explicit reason.
- Strategy output does not itself authorize execution; the approved risk authority must decide.
- Paper/demo validation is required before any live-money consideration.
- Every approved rule must have acceptance criteria and mandatory tests.
