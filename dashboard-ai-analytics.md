# AI Explanation, SOC Dashboard, Realtime Updates, and Analytics

## Incident explanations

`POST /api/v1/ai/analysis/{incident_id}` builds a transparent, deterministic explanation from the incident record, linked detections and their rule evidence, linked security events, and matched threat indicators. The result reports the evidence used, a concise rationale, potential impact, and analyst review recommendations.

The current implementation is an evidence-summary engine, not a call to a hosted language model. It does not send incident or telemetry data to an external provider. Explanations are advisory, mark `requires_human_review`, and return an empty `autonomous_actions` list. Both reading and generating an explanation enforce the same incident ownership/admin access boundary as incident detail.

## SOC dashboard

The React application loads its data from authenticated API routes. Sign-in and analyst registration use `/api/v1/auth/login` and `/api/v1/auth/register`; access and refresh tokens are held in tab-scoped `sessionStorage`, and expired access tokens are rotated through `/api/v1/auth/refresh`.

Set `VITE_API_BASE_URL` at build time to point the UI at an API base such as `https://soc.example/api/v1`. When unset, the development default is `http://localhost:8000/api/v1`.

Dashboard counts, severity distribution, recent notifications, threat indicators, and analytics are database-backed. There is no sample-data fallback: empty data is displayed as empty, and API failures are surfaced in the UI.

### Explicit development demo data

In development, an authenticated user can use **Load demo incidents** in the dashboard or call `POST /api/v1/ai/demo-data` to add twelve simulated incidents with linked simulated events, detections, and dashboard notifications. The operation is scoped to the requesting analyst, safe to repeat, and does not create active detection rules. Demo incident titles and evidence are labeled `[DEMO]` / `simulated`; the reserved documentation IP ranges are used instead of live indicators. This action is disabled outside development and is not a substitute for ingesting real telemetry.

## Realtime dashboard

Connect to `ws(s)://<api-host>/api/v1/ai/dashboard/ws` and send `{"access_token":"<access-token>"}` as the first JSON message. The server authenticates the access token before returning a `dashboard_update` snapshot. It checks for changed dashboard data every two seconds and sends updates only when the snapshot changes. Invalid or expired authentication closes the connection with code `4401`.

The legacy `/api/v1/ai/dashboard/stream` SSE route remains available for compatibility; the React application uses the authenticated WebSocket route.

## Search and analytics

`GET /api/v1/incidents/search` supports `q`, `severity`, `status`, `min_risk`, `max_risk`, `created_after`, `created_before`, `page`, and `page_size`. Results are bounded to 100 records per page and analyst results are restricted to incidents assigned to that user.

`GET /api/v1/ai/analytics?days=7` accepts a rolling window from 1 to 90 days and returns actual daily incident counts, severity/status breakdowns, event and detection totals, and anomaly-score counts. Non-admin users only receive analytics for their assigned incidents and own events/detections.

## Validation

Backend coverage for this functionality is in `backend/app/tests/test_ai_dashboard.py`. Build the frontend from the `frontend` directory with `npm run build`.
