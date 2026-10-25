# Phase 3.8 — AI CEO pipeline approval-boundary review

Date: 2026-10-26
Scope: architecture review based on the previously recorded read-only scenario inventory. No Make scenario was executed or edited.

## Current understanding
The AI CEO + specialist pipeline (TEST) was previously inventoried as creating or updating project/task records, writing approval requests, and logging execution. The approval recorder and approved-only internal handoff gate are separate scenarios. The inventory explicitly did not establish that the main pipeline cannot proceed to every downstream operation unless the separate gate confirms approval.

A fresh scenario/module retrieval was attempted for scenario 8119348 but the connector returned an internal error. This document therefore distinguishes prior inventory from unverified live configuration.

## Recommended execution boundary
Treat AI planning and draft creation as non-external, reversible work. Any action that publishes content, contacts a lead, spends ad budget, changes a live CRM/Sheet record beyond an explicitly approved scope, or activates a scenario must stop at a fail-closed approval boundary.

Required sequence:
1. Validate request schema and trusted client identity.
2. Check tenant-scoped idempotency reservation.
3. Generate plan and draft tasks without external side effects.
4. Write a pending approval request with client_id, request_id, action scope, payload hash, created_at, expires_at, and status.
5. Wait for an authenticated human decision.
6. Independently verify tenant, request, action scope, expiry, payload hash, and one-time decision in the handoff gate.
7. Perform only the specifically approved internal handoff.
8. Record outcome and release/complete the idempotency reservation deterministically.

## Fail-closed rules
- Missing client identity, missing approval, ambiguous identity, expired approval, changed payload, duplicate decision, or gate error means stop.
- Never infer approval from a user-supplied decisionBy string alone.
- Never let a broad APPROVED status authorize a different action or changed payload.
- Keep publishing, outreach, ad spend, deletion, and scenario activation outside this pipeline unless separately approved and explicitly scoped.
- Never use a client_id supplied by an untrusted caller as the sole tenant authorization mechanism.

## Acceptance tests before activation
- A client A request cannot read, approve, or hand off client B's request.
- A forged decisionBy value is rejected.
- A pending/rejected/expired approval cannot pass the gate.
- Replayed decisions and duplicate request IDs do not duplicate downstream work.
- Changing the action payload after approval invalidates the approval.
- Gate timeout/error stops execution.
- An approved request permits only the approved action and tenant.
- Execution log captures request_id, client_id, decision reference, action scope, and final outcome without exposing secrets.

## Status
Specification only. No configuration changes, test executions, activation, production data changes, PR, merge, or deployment were performed. Production Alpha Estate publishing scenario remains out of scope and untouched.
