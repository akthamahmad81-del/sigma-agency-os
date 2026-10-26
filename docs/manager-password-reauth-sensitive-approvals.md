# Sigma Agency OS — Manager Password Re-Authentication for Sensitive Approvals

Status: approved design; implementation specification for the isolated test build only.
Date: 2026-10-26

## Scope and non-negotiable boundaries
- One manager account is permitted to approve sensitive actions.
- Require fresh password re-authentication for every approval of publishing or another sensitive action.
- Apply only to the Sigma Agency OS test build / branch `feat/multi-client-phase-3`.
- Do not edit, activate, run, or deploy Make scenario `8046075`.
- Do not change live Google Sheets records, publish, send outreach, spend money, or call production APIs.
- Keep all test scenarios inactive; use synthetic test data only.
- This document is a design artifact, not evidence that a running login service has already been deployed.

## Trust boundary
A password field inside a Make scenario, webhook payload, Google Sheet, URL, or client-side-only UI is NOT authentication. The system must verify credentials in a trusted server-side component before issuing an approval authorization. Make receives only a short-lived, signed/opaque verification result or server-side approval ID; never receive or persist the raw password.

## Manager credential lifecycle
1. Configure the single manager account through a secure deployment secret / identity-provider admin flow, not a source file or spreadsheet.
2. Store only a password hash using a modern password-hashing algorithm (Argon2id preferred; scrypt or appropriately configured bcrypt as a fallback), with unique salt and supported work factor. Do not store plaintext or reversible encryption.
3. Enforce HTTPS, generic login errors, rate limiting, escalating cooldown/lockout, and audit failed attempts. Never log the password, request body containing it, session token, or credential hash.
4. Use secure, HttpOnly, SameSite cookies for browser sessions, with CSRF protection where cookie authentication is used. Do not put session or approval tokens in URLs or browser local storage.
5. Require re-authentication for every sensitive approval. Re-authentication must be recent and tied to the exact approval intent; a prior login alone is not enough.
6. Provide a documented password-reset / credential-rotation procedure. Never send a password in email or chat.

## Approval intent and authorization
Before showing the password confirmation prompt, create a pending approval intent containing:
- `approval_id` (unique, unpredictable)
- `client_id` from trusted server-side tenant context
- `request_id`
- `action_scope` (for example `PUBLISH_CAMPAIGN`, `SEND_OUTREACH`, `SPEND_AD_BUDGET`, `DELETE_RECORDS`)
- `payload_hash` of a canonical representation of the exact content/action being approved
- `requested_by`, `created_at`, `expires_at`, and `correlation_id`
- status `PENDING`

After password re-authentication succeeds, the trusted server must atomically mark this exact intent approved and append a decision record with `verified_actor_id=manager_account_id`, `decided_at`, `decision_id`, and the same client/request/action/payload identifiers. Approval should expire quickly (recommended 5 minutes for the re-authentication challenge) and be single-use. A changed payload, action, client, request, expiry, or replay invalidates the approval.

## Make gate requirements
- Remove any security reliance on caller-provided `decisionBy`; treat it as display text only, if retained at all.
- Do not let the public webhook or scenario inputs self-assert `APPROVED`, manager identity, or tenant identity.
- The gate must validate a trusted server-generated authorization for the exact `client_id + request_id + action_scope + payload_hash + approval_id`, verify expiry and one-time use, and fail closed on missing identity, timeout, datastore error, mismatch, rejection, or replay.
- Every side-effecting branch must call the gate immediately before the side effect. A separate “approval recorder” scenario is not sufficient if the main pipeline can bypass the gate.
- Idempotency keys must be tenant-scoped: `client_id + request_id`; approval consumption must be atomic to prevent parallel replay.
- Google Sheets may be used for a non-authoritative audit view, but must not be the authentication provider or sole authorization boundary.

## Synthetic acceptance tests (do not run until separately authorized)
1. Correct manager password + matching intent -> one approval can be consumed once.
2. Wrong password -> no approval; rate limit applies.
3. Missing identity/session -> denied.
4. Employee or arbitrary `decisionBy` text -> denied.
5. Client A approval presented for Client B -> denied.
6. Request ID or action scope mismatch -> denied.
7. Payload changes after approval -> denied.
8. Expired or replayed approval -> denied.
9. Concurrent duplicate consume attempts -> at most one succeeds.
10. Auth service timeout / datastore error -> denied, no side effect.
11. Rejected or pending approval -> denied.
12. Password, hash, session token, or secret never appears in logs, sheets, or Make execution payloads.

## Implementation sequence
1. Build/identify the trusted server-side auth endpoint and manager account configuration in the test app.
2. Add the re-authentication dialog to the test dashboard; display the exact action and client being approved.
3. Add a server-side approval-intent/decision store with single-use semantics.
4. Update test-only Make approval recorder and gate to verify the server-issued authorization and exact intent fields; remove `decisionBy` as a trust input.
5. Update tenant-scoped idempotency and ensure the main test pipeline cannot reach any side-effecting path without the gate.
6. Review blueprints and mappings read-only, then run only synthetic negative/positive tests after separate explicit authorization.
7. Keep all test scenarios inactive until the user separately approves activation.

## Current implementation status
This specification has been committed to the development branch only. No Make scenario was edited, run, or activated as part of this change; no live data or production publishing flow was touched. The actual server-side authentication component and Make gate wiring still need implementation and verification.
