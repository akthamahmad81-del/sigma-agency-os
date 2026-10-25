# Sigma Agency OS — Phase 3.2: Existing Data Inventory & Migration Plan

Status: planning artifact only. No spreadsheet rows, headers, Apps Script, Make scenarios, or production publishing workflows were changed.

## Known current workbook
Spreadsheet: Sigma Agency — Leads & Qualification
Tabs previously observed:
- Leads
- Clients
- Approvals
- Execution_Log
- Audit_Log

A prior test record was reported as SIGMA-END2END-001. Treat this as synthetic/test data unless independently verified. The exact current header names, column order, formulas, and Make mappings have NOT been inspected in this step.

## Read-only inventory required before schema changes
For each tab, record:
1. Exact header names and column order.
2. Whether rows contain formulas, validation rules, filters, protected ranges, or named ranges.
3. Current count of populated rows (without copying personal data into this repository).
4. Which Make scenarios read/write the tab and which fields they map.
5. Whether client_id exists and whether every non-empty row has a valid value.
6. Any Apps Script references to sheet names, column numbers, or header strings.

Do not export contact details or other personal data into this repository. Only retain schema metadata and aggregate counts.

## Proposed minimal schema alignment
These are proposals, not live changes:
- Clients: client_id, display_name, lifecycle_status, service_tier, timezone, locale, created_at, updated_at.
- Leads: add/confirm client_id; dedupe key should be client-scoped.
- Approvals: add/confirm client_id; require action, status, requester, approver, timestamps, expiry.
- Execution_Log: add/confirm client_id, correlation_id, actor, action, result, timestamp.
- Audit_Log: add/confirm client_id where applicable, event metadata, timestamp, redacted details.
- ClientSettings (new tab only after inventory): client_id, setting_key/category, setting_value, version, updated_at.
- AgentProfiles (new tab only after inventory): agent_profile_id, client_id, role, allowed scopes, approval requirements, enabled flag.

Do not rename existing tabs or columns before checking mappings. Prefer additive changes and header-based lookups over hard-coded column positions where possible.

## Migration sequence
1. Capture a schema-only snapshot (headers, validations/formulas metadata, mappings; no lead PII).
2. Confirm client IDs and lifecycle values.
3. Create a backup copy of the workbook before any live schema migration.
4. Add missing client_id fields only after mapping review; do not mass-fill ambiguous records automatically.
5. Verify all existing Alpha Estate rows map to CL_ALPHA_ESTATE only where confirmed.
6. Test with synthetic Client A and Client B records.
7. Confirm each test principal can access only its own records through server-enforced authorization.
8. Verify deduplication is scoped by client_id.
9. Review logs and rollback plan.
10. Request explicit approval before any live write, Make activation, or production deployment.

## Acceptance criteria
- [ ] Current headers and mappings captured without modifying data
- [ ] Existing production publishing scenario remains untouched
- [ ] No Make scenario activated
- [ ] No credentials or personal lead data committed to GitHub
- [ ] Client ID coverage is known and exceptions are listed
- [ ] Cross-client isolation tests pass before launch
- [ ] Rollback plan reviewed and approved

## Blocker / limitation
This document is not evidence that the workbook has been inspected live. The connected tools available in this step did not provide direct Google Sheets read access, so the current header inventory remains pending. Do not guess column names or claim a live audit is complete.
