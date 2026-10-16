# Risk scoring, correlation, threat intelligence, and anomaly scoring

## Incident risk scoring

`POST /api/v1/incidents/{incident_id}/risk-score` recomputes the score from the incident severity, linked detection count, affected identity/source context, matched threat-intelligence confidence, and any persisted anomaly score from linked events. Scores are capped at 100. The response contains the component evidence and does not accept caller-supplied score, ML score, confidence, or reasons as overrides.

## Incident correlation

`POST /api/v1/incidents/correlate` requires at least one anchor (`affected_user_id`, `source_ip`, or both). It selects triggered detections that have not already been attached to an incident and match every supplied anchor. `window_minutes` is bounded to 5–1440 (default 60), and `min_detections` is bounded to 2–1000 (default 2). The time window is measured backward from the newest matching event, which supports delayed ingestion. Successful correlation attaches the detections and unique security events to the new incident and stores available threat-intelligence matches.

## Threat-intelligence feed

Administrators can add an individual indicator with `POST /api/v1/ai/threat-indicators`, or import a validated feed batch with `POST /api/v1/ai/threat-indicators/import`. Batches accept up to 1000 IP, FQDN, URL, MD5, SHA-1, or SHA-256 indicators. Values are canonicalized and deduplicated by `(indicator_type, value, source)`; repeated imports update confidence and observation bounds without lowering confidence. Invalid indicators reject the batch.

Security-event enrichment matches canonical source IPs, domains extracted from a resource host, exact URLs, and hashes in event metadata. Correlation links matches to incidents; risk scoring and AI evidence consume the resulting intelligence context. This provides feed ingestion and enrichment; no third-party feed credentials or vendor-specific polling is configured by default.

## Isolation Forest anomaly scoring

Every ingested security event is scored against up to the preceding 1000 events when at least 20 training events are available. A deterministic scikit-learn `IsolationForest` uses a fixed random seed and features for cyclical event time, failure status, 24-hour user/IP/event-type/resource frequencies, and linked detection count. The score is stored with its feature values, model version, sample count, and status in `anomaly_detections`.

Below 20 prior samples, the API records `insufficient_history` with no anomaly label; it does not treat cold-start data as normal. Administrators can recompute a score after more data arrives with `POST /api/v1/security-events/{event_id}/anomaly-score`, and inspect it with the corresponding GET endpoint. Scored anomaly confidence contributes to the incident risk score when the event is linked to an incident.

Run the complete verification suite from `backend`:

```powershell
python -m pytest app\tests -q
```
