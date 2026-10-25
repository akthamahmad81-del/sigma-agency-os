# Sigma Agency OS — Phase 3.3: Multi-Client Isolation Test Plan

Status: test design only. No tests were run against live services and no live records were created.

## Goal
Demonstrate that each client can access only its own records before any multi-client launch. A spreadsheet column alone is not a security boundary: authorization must be enforced in the backend or trusted execution layer.

## Synthetic test fixtures
Use invented data only:
- Client A: `CL_TEST_A`
- Client B: `CL_TEST_B`
- Lead A: `LEAD_TEST_A_001`, client_id `CL_TEST_A`
- Lead B: `LEAD_TEST_B_001`, client_id `CL_TEST_B`
- Approval A: `APR_TEST_A_001`, client_id `CL_TEST_A`
- Approval B: `APR_TEST_B_001`, client_id `CL_TEST_B`

Do not place these fixtures into the production workbook until the owner approves a dedicated test environment or a backed-up, explicitly approved test procedure.

## Test matrix
| ID | Attempt | Expected result |
|---|---|---|
| ISO-01 | Client A requests its own lead | Allow |
| ISO-02 | Client A requests Client B lead by ID | Deny; no record details returned |
| ISO-03 | Client A attempts list-all leads | Return only Client A records |
| ISO-04 | Client A submits a new lead while spoofing client_id = Client B | Reject or overwrite with authenticated tenant ID |
| ISO-05 | Client A tries to approve Client B approval | Deny |
| ISO-06 | Client A attempts to update Client B settings | Deny |
| ISO-07 | Client A reuses a dedupe key used by Client B | Do not cross-block or expose Client B's record |
| ISO-08 | A request has no authenticated tenant context | Deny by default |
| ISO-09 | A user changes role or tenant assignment | Require authorized admin; write audit event |
| ISO-10 | A scenario or agent tries an unapproved external action | Stop and request approval |
| ISO-11 | Error response occurs | Do not expose secrets, tokens, or other tenant data |
| ISO-12 | Client is disabled/offboarded | Deny new access; preserve audit trail per retention policy |

## Required design controls
1. Resolve client_id from authenticated identity/session, not from a user-supplied request field alone.
2. Enforce authorization server-side on every read and write, including list, search, export, and background-agent paths.
3. Scope every lead lookup and dedupe operation to client_id.
4. Use allow-listed actions and default-deny external effects.
5. Keep secrets in a secrets manager or protected connection settings; never in Sheets, HTML, logs, or repository files.
6. Record actor, client_id, action, timestamp, outcome, and correlation ID in audit logs; redact sensitive values.
7. Ensure background agents and Make scenarios have explicit client scope and least privilege.
8. Prevent test data from entering production publishing workflows.
9. Use synthetic data for negative tests; record pass/fail evidence without copying personal data.
10. Define backup, rollback, retention, and client offboarding procedures before launch.

## Release gate
Do not declare multi-client isolation secure until:
- The actual request/authentication path is identified.
- Backend authorization is implemented and reviewed.
- All applicable ISO tests pass in a non-production environment.
- Cross-client list, direct-ID, export, agent, and automation paths are covered.
- Logs show denials without leaking the target record.
- A rollback plan and owner approval exist.

## Current blockers
- No direct Google Sheets read/write tool was available in this session.
- Current Apps Script deployment code and its authentication model have not been audited here.
- The Make scenario-to-sheet mapping has not been inventoried.
- Therefore this plan is not a security certification and should not be treated as evidence that isolation is implemented.
