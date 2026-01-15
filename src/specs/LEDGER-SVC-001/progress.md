# Spec Progress

Active Spec ID: LEDGER-SVC-001

Spec ID: LEDGER-SVC-001
Title: Ledger Service (Python/Postgres/AWS)
Owner: Platform Engineering

## Resume Snapshot
- Current Stage: Draft
- Current Version: v0.1
- Last Updated: 2026-01-15
- Next Action: Request persona reviews and validate open questions with stakeholders.
- Pending Personas: Project Manager, Product Experimentation Architect, Staff UI/UX Designer, Software Engineer, Staff Engineer in Test, Tech Spec Director.

## Current Focus
- Validate throughput, retention, and migration assumptions.
- Confirm invariants and API contract details with upstream teams.

## Persona Review Ledger
| Persona | Status | Version | Link | Notes |
| --- | --- | --- | --- | --- |
| Project Manager | Not Requested | - | - | - |
| Product Experimentation Architect | Not Requested | - | - | - |
| Tech Spec Director | Not Requested | - | - | - |
| Software Engineer | Not Requested | - | - | - |
| Staff UI/UX Designer | Not Requested | - | - | - |
| Staff Engineer in Test | Not Requested | - | - | - |

## Decision Log
| Date | Decision | Rationale | Owner |
| --- | --- | --- | --- |
| 2026-01-15 | Append-only ledger with enforced double-entry invariants | Auditability and deterministic balance reconstruction | Platform Engineering |
| 2026-01-15 | Outbox pattern for downstream events | Prevents event loss and keeps writes atomic | Platform Engineering |

## Assumption Register
| Assumption | Status (Pending/Verified) | Owner | Notes |
| --- | --- | --- | --- |
| Single-currency per transaction in v1 | Pending | Platform Engineering | Multi-currency deferred; confirm with finance |
| No PAN/PCI data stored in ledger | Pending | Platform Engineering | Keep PCI scope out of service |
| Upstream services can provide idempotency keys | Pending | Platform Engineering | Needed for safe retries |

## Risk Register
| Severity | Risk | Mitigation | Owner |
| --- | --- | --- | --- |
| High | Invariant violation under concurrency | DB constraints + transactional writes + test harness | Platform Engineering |
| Medium | Data migration errors from legacy systems | Dual-write + reconciliation checks before cutover | Platform Engineering |
| Medium | High write volume causing latency | Indexing strategy + partitioning + snapshots | Platform Engineering |

## Open Questions
- [ ] Expected peak TPS and data volume for sizing and partitioning.
- [ ] Required retention period beyond 7 years for compliance.
- [ ] Single-currency per transaction is assumed; is multi-currency needed in v1?
- [ ] Which upstream systems will dual-write during migration?

## Change Timeline
| Date | Version | Summary | Author | Journal Entry |
| --- | --- | --- | --- | --- |
| 2026-01-15 | v0.1 | Initial draft created | Avery | journal/20260115-1025-initial-draft.md |
