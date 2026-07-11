# Changelog

All notable verified project changes are recorded here.

## 2026-07-11

### Added

- Repository README and governance baseline
- Master delivery plan
- Architecture governance register
- Business-rules register
- Approved decision register
- Progress register
- Current bounded task
- Initial test-report register
- Deployment governance register
- Change log
- Approved product, market-data, strategy, risk, execution, persistence, security, monitoring, and deployment-boundary decisions

### Closed

- `FOUNDATION-001 — Governance and repository documentation baseline`
- `ARCH-001 — Product Owner Decision Discovery and Architecture Lock Preparation`

### Activated

- `MDE-001-SPEC — Market Data Engine Detailed Specification and Test Contract`

### Architecture lock

- Core V1 architecture is locked for a private, single-owner, paper/demo Binance USD-M Futures application.
- React + TypeScript + Vite frontend, Node.js + TypeScript authoritative controller, Python FastAPI stateless strategy service, and PostgreSQL persistence are approved.
- Risk, execution, trade management, persistence, reconciliation, security, monitoring, and notification authority boundaries are recorded.
- Engine-specific values explicitly listed in `DECISIONS.md` remain deferred to their owning bounded specifications and may not be guessed.

### Verification

- Foundation closure was recorded from canonical remote `main` evidence using base commit `2df3cf14d112ba5bd25ef94384267db7971fc29d`.
- ARCH-001 decisions were approved by the Product Owner across seven decision batches.
- No application code or deployment configuration was changed during architecture locking.

### Status

Foundation and core architecture are complete. Market Data Engine specification is the only active task.