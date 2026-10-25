# Phase 3.7 — Approval and idempotency review status

Date: 2026-10-26
Scope: read-only inspection; no scenario executions or changes.

## Confirmed from the approval recorder mapping
- The recorder appends a row to the Approvals sheet using requestId, decision, decisionBy, current time, and comments.
- The subsequent lookup filters by requestId and returns the latest matching row.
- The recorder calls a separate internal handoff gate.
- The visible mappings do not show client_id in the approval append, lookup filter, or gate call payload.
- Authentication of decisionBy was not demonstrated by the inspected mapping.

## Unverified components
Read-only configuration retrieval for the approved-only gate and idempotency guard returned connector errors. Their exact field mappings therefore remain unverified; no conclusions about their internal filters should be treated as confirmed.

## Required before activation
1. Resolve tenant scope from a trusted identity or trusted request record.
2. Verify requestId belongs to that tenant and is eligible for a decision.
3. Authenticate and authorize the decision maker server-side.
4. Filter approval lookups by tenant and request ID.
5. Validate allowed decision values and reject stale or replayed decisions.
6. Ensure the internal handoff gate independently validates the approval.
7. Scope idempotency keys by tenant and define reservation release/retry behavior.
8. Test negative cases with synthetic data in a non-production environment.

## Safety status
All reviewed scenarios remain inactive. No run, edit, or activation was performed. The production Alpha Estate publishing workflow remains untouched. No live Google Sheets data was modified.
