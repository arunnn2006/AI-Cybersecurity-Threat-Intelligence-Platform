# Authentication, ingestion, and detection

## Authentication and role enforcement

- Register with `POST /api/v1/auth/register`; self-registration always assigns the `analyst` role.
- Sign in with `POST /api/v1/auth/login` using either username or email.
- Send access tokens as `Authorization: Bearer <access_token>`.
- Rotate a refresh token with `POST /api/v1/auth/refresh`. A refresh token is single-use; replay is rejected.
- `POST /api/v1/auth/logout` revokes all refresh tokens belonging to the authenticated account.
- Password changes and account deactivation revoke that account's refresh tokens. Existing access tokens remain valid until their short expiry.
- Only administrators can list or edit other users and manage detection rules. Self-service profile changes cannot modify a role, administrator flag, or active status. The last active administrator cannot be demoted or deactivated.
- Production configuration must set a non-default `SECRET_KEY` with at least 32 characters.

Refresh tokens are stored as SHA-256 digests, not as bearer credentials or password hashes. Existing sessions from deployments that used the previous password-hash format must authenticate again after this version is deployed.

## Security-event ingestion

Submit events to `POST /api/v1/security-events/ingest` as an authenticated user. The payload includes an event ID, timezone-qualified ISO-8601 timestamp, event type, and optional user, IP, resource, status, metadata, and source-normalized object.

The API:

- rejects timezone-naive timestamps and invalid IP addresses;
- normalizes timestamps to UTC, IP addresses to canonical notation, event types/statuses to lowercase, and trims identifiers;
- constructs its canonical `normalized_event` from validated fields rather than trusting client-supplied normalized values;
- retains optional caller-provided normalized data under `normalized_event.source_data`;
- returns a conflict for repeated event IDs and rejects user IDs that do not refer to a local account.

## Detection rules

Rule management is administrator-only. Supported `rule_type` values are:

| Type | Required `config` key | Matching behavior |
| --- | --- | --- |
| `event_type` | `event_type` or `value` | Exact match against the normalized event type |
| `source_ip` | `source_ip` or `value` | Exact IP or membership in an IPv4/IPv6 CIDR network |
| `user_id` | `user_id` or `value` | Exact positive local user ID |
| `resource` | `resource` or `value` | Exact match against the resource |
| `contains` | `value` | Case-insensitive substring in event type or resource |

An optional `confidence` must be numeric and within `[0, 1]`. Invalid rule types/configuration are rejected during creation or update. Detection creation suppresses a repeated matching rule/event signature within the configured 15-minute window.

Run the backend suite, including authentication lifecycle and event/rule validation tests, from `backend`:

```powershell
python -m pytest app\tests -q
```
