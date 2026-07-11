# Architecture

## Status

Architecture is not yet locked. This document records approved system boundaries only after Product Owner approval.

## Initial boundaries

- Market data must come from verified exchange APIs.
- Strategy analysis, risk approval, execution, persistence, frontend, and monitoring must remain clearly separated.
- No module may silently bypass another module's authority.
- Persistent state must survive process restarts.
- Paper/demo mode is mandatory before any live-money consideration.

## Change control

Any core architecture change requires:

1. Architect review
2. Conflict check against Bible v2.0 — LOCKED
3. Product Owner approval
4. Decision record in `DECISIONS.md`
5. Updated tests and documentation
