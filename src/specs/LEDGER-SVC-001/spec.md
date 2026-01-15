# Ledger Service (Python/Postgres/AWS)

Active Spec ID: LEDGER-SVC-001

Spec ID: LEDGER-SVC-001
Version: v0.1
Status: Draft
Owner: Platform Engineering
Last Updated: 2026-01-15
Related: progress.md | journal/ | reviews/

## Executive Summary
- Value statement: Provide a canonical, append-only ledger service that guarantees double-entry correctness and auditability for all monetary movements.
- Target outcome: Consolidate financial postings into a single service with deterministic balances, consistent APIs, and verifiable audit trails.
- Success metrics:
  - 0 unbalanced transactions accepted in production.
  - p95 posting latency < 300ms at steady state.
  - 99.99% successful write availability (monthly).
  - 100% idempotent acceptance on duplicate requests with same key.
- Primary risks:
  - Concurrency or validation gaps that allow imbalance or drift.
  - Data migration errors from legacy systems.
  - Performance degradation under high write volume.

## Problem Statement
Because monetary events are currently recorded in multiple services, finance and operations cannot rely on a single source of truth for balances or audits, which risks reconciliation errors, delayed close, and compliance exposure.

## Goals
- Provide a stable API to post ledger transactions and query balances.
- Enforce double-entry invariants at write time.
- Guarantee idempotent writes per client-provided key.
- Support audit-ready history (append-only entries with full provenance).
- Emit reliable downstream events for analytics and reconciliation.

## Non-Goals
- Payment processing or settlement orchestration.
- Billing/invoicing logic or pricing rules.
- End-user UI or reporting dashboard.
- Automated FX conversion (multi-currency support is deferred).

## User Stories / Jobs
- As a billing service, I want to post a charge with an idempotency key so duplicate retries do not create duplicate entries.
- As a finance analyst, I want to query balances and entries as-of a timestamp for audit and close.
- As a reconciliation job, I want to export ledger entries by account and date range.

## Scope Boundaries
- In-scope:
  - Ledger accounts, transactions, entries (postings), and balance queries.
  - Reversals via compensating transactions.
  - Snapshotting for faster balance reads.
- Out-of-scope:
  - UI/portal for finance operations.
  - External settlement or payout logic.
  - Policy-based approvals or workflow management.
- Deferred:
  - Multi-currency transactions and FX rates.
  - Multi-tenant ledger segregation beyond a single org boundary.

## System Context
- Systems touched:
  - Upstream product services (billing, payments, refunds, adjustments).
  - Data warehouse / analytics pipelines.
- Integration points:
  - REST JSON API for posting and querying.
  - Event emission via outbox -> SNS/SQS or EventBridge.
- Constraints:
  - Backend in Python.
  - Postgres as the primary ledger store.
  - AWS hosting with managed services.

## Domain Model
- Entities:
  - LedgerAccount: id, name, type (asset, liability, revenue, expense), status, metadata.
  - LedgerTransaction: id, external_ref, status (posted, reversed), created_at.
  - LedgerEntry: id, transaction_id, account_id, amount, currency, direction (debit/credit), memo.
  - BalanceSnapshot: account_id, as_of, balance, currency.
- States:
  - Account: active, closed.
  - Transaction: posted, reversed.
- Invariants:
  - Sum of entry signed amounts per transaction must equal 0.
  - All entries in a transaction share the same currency.
  - Accounts must be active at posting time.
  - Every write is idempotent for a given (source, idempotency_key).

## API and Contract Surfaces
- Endpoints (v1):
  - POST /v1/accounts
  - GET /v1/accounts/{account_id}
  - POST /v1/transactions
  - GET /v1/transactions/{transaction_id}
  - POST /v1/transactions/{transaction_id}/reverse
  - GET /v1/entries?account_id=&from=&to=
  - GET /v1/accounts/{account_id}/balance?as_of=
- Payloads (POST /v1/transactions):
  - idempotency_key (header), external_ref, currency, entries[]
  - entries[]: account_id, amount, direction (debit|credit), memo
- Versioning:
  - URL versioning (/v1). Additive fields only without breaking changes.
- Error semantics:
  - 400 invalid payload or missing fields.
  - 409 idempotency conflict or duplicate external_ref.
  - 422 unbalanced transaction or invariant violation.
  - 404 account/transaction not found.
  - 503 transient storage or dependency failure.

## Data Flow and Storage
- Sources:
  - Synchronous API calls from upstream services.
- Transformations:
  - Validate request -> construct transaction + entries -> write in a single DB transaction -> outbox insert -> async event publish.
- Storage:
  - Postgres tables: accounts, transactions, entries, idempotency_keys, balance_snapshots, outbox.
  - Constraints and indexes enforce invariants (unique idempotency key per source, FK integrity, transaction-level check constraints).
- Retention:
  - Ledger entries retained indefinitely (minimum 7 years, configurable by policy).
- Privacy notes:
  - No card or bank details stored. Store only external references and hashed tokens when needed.

## UX Flows
- Primary journeys:
  - Post transaction -> receive transaction id and entries.
  - Query balance as-of timestamp.
  - Export entries by account/date for reconciliation.
- Edge cases:
  - Duplicate idempotency key returns original transaction.
  - Reversal posts a compensating transaction with linkage to original.
- Empty states:
  - Account with zero balance returns 0 and empty entry list.
- Accessibility notes:
  - N/A for API, but error responses must be structured and documented.

## Security and Compliance
- Threat model:
  - Unauthorized postings, replay attacks, tampering with ledger history.
- Sensitive data handling:
  - Encrypt at rest (RDS + KMS). TLS in transit. IAM least privilege.
- Policy alignment:
  - SOC2 controls; avoid PCI scope by not storing PAN data.

## Observability
- Logs:
  - Structured logs with request_id, idempotency_key, transaction_id.
- Metrics:
  - posting_success_rate, posting_latency_ms, unbalanced_rejects, idempotency_hits, outbox_lag_seconds.
- Traces:
  - OpenTelemetry spans across API -> DB -> outbox publisher.
- Alert thresholds:
  - Error rate > 1% over 5 minutes.
  - Outbox lag > 5 minutes.
  - Any unbalanced acceptance (should be 0).

## Testing Strategy
- Unit tests:
  - Ledger validation (balance invariants, currency constraints, account status).
  - Idempotency key handling logic.
- Integration tests:
  - Postgres transaction atomicity, constraint enforcement, and outbox correctness.
- End-to-end tests:
  - Post transaction -> query balance -> verify event emission.

## Rollout Plan
- Feature flags:
  - Dual-write flag to mirror to ledger without impacting source of truth.
- Migration steps:
  - Backfill historical entries -> validate balances -> enable write path.
- Rollback criteria:
  - Disable ledger write flag and revert to legacy system if invariants fail.

## Dependencies
- AWS RDS Postgres, AWS ECS/Fargate (or EKS), AWS KMS, CloudWatch, SNS/SQS or EventBridge.
- Python stack: FastAPI, SQLAlchemy, Alembic, Pydantic.

## Open Questions
- [ ] Expected peak TPS and data volume for sizing and partitioning.
- [ ] Required retention period beyond 7 years for compliance.
- [ ] Single-currency per transaction is assumed; is multi-currency needed in v1?
- [ ] Which upstream systems will dual-write during migration?

## Appendix
- Links:
- Related specs:
