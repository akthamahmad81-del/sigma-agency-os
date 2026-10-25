# Phase 3.9 — Approval remediation implementation checklist

Date: 2026-10-26
Status: implementation-ready checklist; Make edits are blocked pending successful fresh scenario reads.

## Scope guard
Only test scenarios may be changed after their current state is retrieved and validated. Do not modify, run, or activate the production publishing scenario (8046075). Do not touch live client records or publish/send/spend.

## Remediation order
1. **Request identity:** ensure every pipeline request has a trusted client_id and request_id. Reject missing values.
2. **Tenant-scoped idempotency:** key reservations by client_id + request_id; reject cross-tenant collisions; define stale reservation recovery.
3. **Approval request record:** write client_id, request_id, action_scope, payload_hash, status=PENDING, created_at, expires_at, requested_by, and correlation_id.
4. **Authenticated decision:** derive decision maker from an authenticated integration/user context, not an arbitrary decisionBy text field. Validate decision in an allowlist.
5. **Recorder:** append immutable decision records including client_id, request_id, decision, verified_actor_id, decided_at, and comments; use a unique decision_id.
6. **Latest decision lookup:** filter by both client_id and request_id, then validate status, actor, timestamp, expiry, and decision_id.
7. **Independent gate:** the handoff gate must repeat the tenant/request/action/payload/expiry checks itself; never trust a caller-provided APPROVED flag alone.
8. **Pipeline wiring:** every side-effecting branch must route through the gate. Planning and draft creation can occur before approval; publishing, outreach, spend, deletion, and live record changes cannot.
9. **Audit:** log outcome and correlation identifiers without secrets or unnecessary personal data.
10. **Negative tests:** cross-tenant access, forged actor, missing identity, rejected/pending/expired decision, changed payload, replayed decision, duplicate request, datastore failure, and gate timeout must all stop closed.

## Acceptance criteria
- No path reaches a side-effecting module without a successful gate result tied to the same client, request, action scope, and payload hash.
- One approval cannot authorize a different payload or another client.
- Replays and duplicate requests do not duplicate downstream work.
- Every failure path stops safely and is observable in the execution log.
- Test cases use synthetic data and inactive test scenarios only.

## Current blocker
Fresh Make reads for scenario 8119348 (AI CEO + Specialist Agents Pipeline TEST), scenario 8119840 (Approved-only Internal Handoff Gate TEST), and scenario 8119576 (Idempotency Guard TEST) returned connector internal errors. Without current blueprints and module IDs/configuration, editing would be unsafe and could break a test flow. No scenario edit or run was attempted.

## Safety status
No scenario was activated or executed. No Google Sheets data changed. Production Alpha Estate publishing remains untouched. No PR, merge, or deployment.
