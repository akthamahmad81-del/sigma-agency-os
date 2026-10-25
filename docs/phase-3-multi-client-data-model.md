# Sigma Agency OS — Phase 3.1: Multi-Client Data Model

Status: design proposal; not connected to production systems.

## Safety boundaries
- Do not modify or activate the existing Alpha Estate publishing scenario.
- Do not activate any Make scenario as part of this phase.
- Do not expose Google Sheets, Apps Script endpoints, API keys, tokens, or webhooks in the public dashboard.
- Use synthetic test records until isolation and authorization checks pass.
- This document is a design artifact only; it does not change live Sheets or Apps Script.

## Core entities

### Clients
One record per customer account.
- client_id: immutable unique identifier, e.g. CL_ALPHA_ESTATE
- display_name
- lifecycle_status: prospect | onboarding | active | paused | offboarded
- service_tier
- timezone
- locale
- created_at, updated_at
- data_owner / authorized_admins (references, not passwords or secrets)

### ClientSettings
One record per client and setting category.
- client_id
- brand_voice
- approved_channels
- service_flags
- approval_policy
- monthly_budget_limit (optional; informational until explicitly approved)
- settings_version
- updated_at

### Leads
Every lead must belong to exactly one client.
- lead_id: immutable unique identifier
- client_id: required foreign key
- source
- name and contact fields (minimum necessary)
- status
- consent_or_contact_basis
- created_at, updated_at
- dedupe_key: normalized contact identifier scoped to client_id

### Approvals
Every sensitive action requires an approval record.
- approval_id
- client_id
- requested_action
- payload_summary (avoid unnecessary personal data)
- requested_by
- status: pending | approved | rejected | expired
- approved_by, approved_at, expires_at
- execution_reference (only after an authorized execution)

### ExecutionLog / AuditLog
Append-only event records:
- event_id
- client_id
- actor_type and actor_id
- action
- target_type and target_id
- result
- timestamp
- correlation_id
- redacted_details

### AgentProfiles
AI role configuration scoped to a client:
- agent_profile_id
- client_id
- role
- allowed_read_scopes
- allowed_write_scopes
- requires_approval_for
- model/provider reference (no API secrets)
- enabled flag (default false for external actions)

## Isolation and authorization rules
1. Every client-owned row must carry client_id; reject missing or unknown IDs.
2. Reads and writes must be filtered by authorized client IDs on the server side, never only in the browser.
3. Dedupe leads on client_id + normalized contact identifier; the same contact may exist under different clients where legitimately sourced.
4. Never store credentials, access tokens, API keys, or webhook secrets in Sheets or client-side HTML.
5. Default-deny for external actions. Publishing, outreach, ad spend, deletion, and scenario activation require explicit approval.
6. Keep Alpha Estate's existing production publishing path out of scope.
7. Log every permission change and approved execution; redact secrets and unnecessary personal data.
8. Offboarding disables agents and integrations for that client without deleting audit history.
9. Test cross-client access with synthetic Client A and Client B records; each must be unable to read or alter the other's data.
10. Do not claim compliance or secure isolation until the backend enforces and tests these rules.

## Suggested initial sheet layout
- Clients (existing central registry)
- ClientSettings (new)
- Leads (existing; add/validate client_id)
- Approvals (existing; add/validate client_id)
- AgentProfiles (new)
- Execution_Log (existing; add/validate client_id)
- Audit_Log (existing; add/validate client_id)

Do not add or change live columns until the current headers and mappings are reviewed. Existing rows should be backed up before any schema migration.

## Acceptance checklist for Phase 3.1
- [ ] Existing headers and mappings inventoried without changing data
- [ ] Stable client IDs and lifecycle values agreed
- [ ] Every client-owned row has a valid client_id
- [ ] Server-side authorization design documented
- [ ] Synthetic cross-client isolation tests pass
- [ ] No production scenario or publishing workflow changed
- [ ] Rollback plan prepared before any live schema migration

## Next step
Read-only inventory of the existing Sheets headers, followed by a proposed migration plan. Do not mutate the spreadsheet in this step.
