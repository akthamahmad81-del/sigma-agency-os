# Sigma Agency OS — Secure Dashboard Bridge

## Status
Planning artifact on the isolated `feat/secure-dashboard-bridge` branch. This does not deploy or change the production publisher, Google Apps Script deployment, Google Sheets data, or Make scenarios.

## Current verified state
- `index.html` is a static RTL dashboard prototype with hard-coded demo status and no live API connection.
- The existing Apps Script project reads summary data from a Google spreadsheet and serves an `Index` HTML file; its current `doGet()` is not a JSON API.
- Make has separate inactive test scenarios. They must remain inactive unless separately approved.
- No dashboard secret, API key, webhook URL, or Google password belongs in browser code.

## Target architecture
1. **Dashboard (GitHub Pages):** presentation only. It may call only a deliberately exposed, read-only API endpoint. No Google credentials or privileged secrets in the browser.
2. **Read-only bridge:** a separate Apps Script project/deployment or equivalent backend, isolated from the existing publishing deployment. It returns a minimal JSON summary and uses an explicit allowlist of actions.
3. **Google Sheets:** source of truth. Read-only dashboard endpoints should read only the required columns/counts and never mutate rows.
4. **Make:** orchestration layer. No scenario is activated by the dashboard. Each future action must go through an approval gate and an explicitly allowlisted scenario.
5. **Audit:** record read/action requests, actor, request ID, decision, timestamps and outcome; never record secrets.

## Non-negotiable safety requirements
- Do not edit or redeploy the current publishing system.
- Do not run, activate, deactivate, or modify existing Make scenarios as part of dashboard connection work.
- Fail closed when identity, approval, request ID, or payload validation is missing.
- Use separate test spreadsheet/project for write tests; read-only integration tests only against production data.
- Never accept a Google password in a custom form. Apps Script cannot validate a Google password.
- CORS is not authentication. A public Apps Script web-app URL plus a shared secret embedded in static JavaScript is not a secure bridge.
- Sensitive actions require authenticated identity, recent re-authentication where appropriate, single-use time-limited approval bound to the exact request/payload, and an audit trail.
- Do not return personal lead details to the public dashboard unless the viewer is authenticated and authorized.

## Read-only API contract (proposed)
`GET /api/v1/dashboard-summary`
- Response: `{ "connected": true, "totalLeads": 0, "leadStatuses": {}, "totalClients": 0, "totalApprovals": 0, "generatedAt": "ISO-8601" }`
- Must use an authenticated/authorized backend route; the route is not implemented yet.
- Errors return a generic status/message and do not leak spreadsheet IDs, stack traces, or internal credentials.
- No write methods are part of v1.

## Implementation sequence
1. Review deployment access settings and decide whether to create a separate Apps Script project. Do not change the current deployment.
2. Implement and test a read-only JSON endpoint in an isolated project, preferably against a test spreadsheet first.
3. Verify identity/access controls and response shape without exposing lead-level data.
4. Add the dashboard client with loading, error, stale-data and disconnected states.
5. Run a read-only integration test, record evidence, and only then consider a separately approved deployment.
6. Keep any write/action endpoints out of v1; design them later around approval and idempotency.

## Definition of done
- Dashboard reads a real summary from an isolated, explicitly authorized endpoint.
- No secrets are shipped to GitHub Pages.
- No production publisher or Make scenario was changed or activated.
- Error and unauthorized states are visible and fail closed.
