# Public Profile Discovery - API Quick Reference

## Base URL
```
http://localhost:3000/username
```

---

## Endpoints

### 1. Search Public Profiles

**GET** `/username/search`

Fuzzy search for public profiles with similarity scoring.

#### Parameters

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `query` | string | ✅ Yes | - | Search term (min 2 chars) |
| `limit` | number | ❌ No | 10 | Max results (1-100) |

#### Example Request

```bash
curl "http://localhost:3000/username/search?query=alice&limit=10"
```

#### Example Response

```json
{
  "profiles": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "alice",
      "publicKey": "GBXGQ55JMQ4L2B6E7S8Y9Z0A1B2C3D4E5F6G7H8I7YWR",
      "similarityScore": 95,
      "lastActiveAt": "2025-03-27T10:30:00Z",
      "createdAt": "2025-02-19T08:00:00Z"
    },
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "username": "alicen",
      "publicKey": "GCXHJ66KNR5M3C7F8T9A0B1C2D3E4F5G6H7I8J9K0LAS",
      "similarityScore": 85,
      "lastActiveAt": "2025-03-26T15:20:00Z",
      "createdAt": "2025-02-20T09:15:00Z"
    }
  ],
  "total": 2
}
```

#### Status Codes

- `200 OK` - Success
- `400 Bad Request` - Invalid query (too short or invalid format)

---

### 2. Get Trending Creators

**GET** `/username/trending`

Get trending creator profiles ranked by transaction volume.

#### Parameters

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `timeWindowHours` | number | ❌ No | 24 | Time window (1-720 hours) |
| `limit` | number | ❌ No | 10 | Max creators (1-100) |

#### Example Request

```bash
curl "http://localhost:3000/username/trending?timeWindowHours=24&limit=10"
```

#### Example Response

```json
{
  "creators": [
    {
      "id": "770e8400-e29b-41d4-a716-446655440002",
      "username": "toptrader",
      "publicKey": "GDXYZ123ABC456DEF789GHI012JKL345MNO678PQR901STU",
      "transactionVolume": 75000.50,
      "transactionCount": 200,
      "lastActiveAt": "2025-03-27T11:45:00Z",
      "createdAt": "2025-02-19T08:00:00Z"
    },
    {
      "id": "880e8400-e29b-41d4-a716-446655440003",
      "username": "activeuser",
      "publicKey": "GEABC456DEF789GHI012JKL345MNO678PQR901STU234VWX",
      "transactionVolume": 35250.75,
      "transactionCount": 95,
      "lastActiveAt": "2025-03-27T09:30:00Z",
      "createdAt": "2025-02-20T10:00:00Z"
    }
  ],
  "timeWindowHours": 24,
  "calculatedAt": "2025-03-27T12:00:00Z"
}
```

#### Status Codes

- `200 OK` - Success
- `400 Bad Request` - Invalid time window parameter

---

### 3. Toggle Public Profile Visibility

**POST** `/username/toggle-public`

Enable or disable public profile visibility.

#### Request Body

```json
{
  "username": "alice",
  "publicKey": "GBXGQ55JMQ4L2B6E7S8Y9Z0A1B2C3D4E5F6G7H8I7YWR",
  "isPublic": true
}
```

#### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `username` | string | ✅ Yes | Username to toggle |
| `publicKey` | string | ✅ Yes | Owner's Stellar public key |
| `isPublic` | boolean | ✅ Yes | Visibility state |

#### Example Request

```bash
curl -X POST "http://localhost:3000/username/toggle-public" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "publicKey": "GBXGQ55JMQ4L2B6E7S8Y9Z0A1B2C3D4E5F6G7H8I7YWR",
    "isPublic": true
  }'
```

#### Example Response (Success)

```json
{
  "ok": true
}
```

#### Status Codes

- `200 OK` - Successfully toggled
- `400 Bad Request` - Invalid username format
- `404 Not Found` - Username not found or wrong ownership
- `500 Internal Server Error` - Server error

---

### 4. Rename Username (with redirect preservation)

**POST** `/username/rename`

Rename a username while preserving existing payment links. The previous
username is retained as a permanent redirect alias so that any payment link
or QR code that already references the old username continues to resolve to
the same Stellar public key. Self-custody is preserved: the rename only
affects the username-to-publicKey mapping, never the key itself.

#### Request Body

```json
{
  "currentUsername": "alice",
  "newUsername": "alice-co",
  "publicKey": "GBXGQ55JMQ4L2B6E7S8Y9Z0A1B2C3D4E5F6G7H8I7YWR",
  "idempotencyKey": "3f9c1e2a-7b4d-4c8e-9f01-2a3b4c5d6e7f"
}
```

#### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `currentUsername` | string | ✅ Yes | Username being renamed (normalized, lowercase) |
| `newUsername` | string | ✅ Yes | Desired new username (normalized, lowercase) |
| `publicKey` | string | ✅ Yes | Owner's Stellar public key (must match current owner) |
| `idempotencyKey` | string (UUID) | ✅ Yes | Client-generated key; replays return the original result |

#### Example Request

```bash
curl -X POST "http://localhost:3000/username/rename" \
  -H "Content-Type: application/json" \
  -d '{
    "currentUsername": "alice",
    "newUsername": "alice-co",
    "publicKey": "GBXGQ55JMQ4L2B6E7S8Y9Z0A1B2C3D4E5F6G7H8I7YWR",
    "idempotencyKey": "3f9c1e2a-7b4d-4c8e-9f01-2a3b4c5d6e7f"
  }'
```

#### Example Response (Success)

```json
{
  "ok": true,
  "username": "alice-co",
  "redirectFrom": "alice",
  "publicKey": "GBXGQ55JMQ4L2B6E7S8Y9Z0A1B2C3D4E5F6G7H8I7YWR",
  "renamedAt": "2025-03-27T12:05:00Z"
}
```

#### Status Codes

- `200 OK` - Rename applied (or idempotent replay of a prior success)
- `400 Bad Request` - Malformed username or missing idempotency key
- `401 Unauthorized` - Missing or invalid signature over the rename payload
- `403 Forbidden` - `publicKey` does not own `currentUsername`
- `404 Not Found` - `currentUsername` does not exist
- `409 Conflict` - `newUsername` already taken, or `currentUsername` is a reserved redirect alias
- `410 Gone` - Redirect alias expired (only when a non-permanent alias policy is configured)
- `503 Service Unavailable` - Dependency (DB/registry) unavailable; safe to retry with the same idempotency key

#### Redirect Semantics

- The old username becomes a **permanent redirect alias** to the new username.
- Payment links, QR codes, and share URLs that embed the old username keep
  resolving to the same `publicKey`; no funds are ever routed to a new key.
- Redirects are resolved server-side before any payment intent is created, so
  the financial invariant "username → publicKey is stable for the lifetime of
  a payment link" is preserved.
- A username that is currently a redirect alias cannot be re-registered by a
  different wallet (`409 Conflict`).

#### Idempotency & Rollback

- Every rename requires an `idempotencyKey`. Replaying the same key returns
  the original `200 OK` response without re-applying the change.
- Renames are applied atomically: the new mapping and the redirect alias are
  written in a single transaction. On dependency failure the transaction is
  rolled back and the old username remains fully active.
- If the new username write succeeds but the alias write fails, the whole
  transaction is rolled back — there is never a window where the old link
  stops resolving.

#### Observability

- Structured log fields: `event=username.rename`, `currentUsername`,
  `newUsername`, `publicKey` (hashed), `idempotencyKey`, `outcome`,
  `latencyMs`. No secrets or raw keys are logged.
- Metrics: `username_rename_total{outcome}`, `username_rename_latency_ms`,
  `username_redirect_resolve_total{hit|miss}`.

---

### 5. Report a Public Profile (Abuse Reporting)

**POST** `/username/report`

Submit an abuse report against a public profile. Reports are accepted only
from authenticated reporters and are recorded for moderator review. Reporting
never mutates the target profile or its `username → publicKey` mapping, so
self-custody and the financial invariants are preserved.

#### Request Body

```json
{
  "username": "alice",
  "category": "impersonation",
  "details": "Profile is impersonating a known creator.",
  "idempotencyKey": "9b2d4f6a-1c3e-4a5b-8d7f-0e1f2a3b4c5d"
}
```

#### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `username` | string | ✅ Yes | Target profile username (normalized, lowercase) |
| `category` | string | ✅ Yes | One of `spam`, `impersonation`, `fraud`, `harassment`, `other` |
| `details` | string | ❌ No | Free-text details (max 2000 chars) |
| `idempotencyKey` | string (UUID) | ✅ Yes | Client-generated key; replays return the original result |

#### Example Request

```bash
curl -X POST "http://localhost:3000/username/report" \
  -H "Authorization: Bearer <reporter-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "category": "impersonation",
    "details": "Profile is impersonating a known creator.",
    "idempotencyKey": "9b2d4f6a-1c3e-4a5b-8d7f-0e1f2a3b4c5d"
  }'
```

#### Example Response (Success)

```json
{
  "ok": true,
  "reportId": "c1a2b3c4-d5e6-4f70-8a9b-0c1d2e3f4a5b",
  "status": "open",
  "createdAt": "2025-03-27T12:10:00Z"
}
```

#### Status Codes

- `201 Created` - Report recorded (or idempotent replay of a prior success)
- `400 Bad Request` - Malformed payload, unknown `category`, or missing idempotency key
- `401 Unauthorized` - Missing or invalid reporter authentication
- `404 Not Found` - Target `username` does not exist
- `409 Conflict` - Duplicate report for the same target by the same reporter within the dedupe window
- `410 Gone` - Target profile no longer exists (deleted after lookup)
- `503 Service Unavailable` - Dependency (DB/moderation queue) unavailable; safe to retry with the same idempotency key

#### Idempotency & Deduplication

- Every report requires an `idempotencyKey`. Replaying the same key returns
  the original `201 Created` response without creating a second report.
- A reporter may not file more than one open report against the same target
  within the dedupe window; the second attempt returns `409 Conflict` with the
  existing `reportId`.
- Reports are written in a single transaction. On dependency failure the
  transaction is rolled back and no partial report is persisted.

#### Observability

- Structured log fields: `event=profile.report`, `reportId`, `username`,
  `category`, `reporterId` (hashed), `idempotencyKey`, `outcome`, `latencyMs`.
  Reporter identity and free-text details are never logged in raw form.
- Metrics: `profile_report_total{category,outcome}`,
  `profile_report_latency_ms`, `profile_report_dedupe_total{hit|miss}`.

---

### 6. List Abuse Reports (Moderation)

**GET** `/username/reports`

List abuse reports for moderator review. Requires a moderator or admin role.

#### Parameters

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `status` | string | ❌ No | `open` | Filter: `open`, `resolved`, `dismissed` |
| `limit` | number | ❌ No | 20 | Max results (1-100) |
| `cursor` | string | ❌ No | - | Opaque pagination cursor |

#### Example Request

```bash
curl "http://localhost:3000/username/reports?status=open&limit=20" \
  -H "Authorization: Bearer <moderator-token>"
```

#### Example Response

```json
{
  "reports": [
    {
      "reportId": "c1a2b3c4-d5e6-4f70-8a9b-0c1d2e3f4a5b",
      "username": "alice",
      "category": "impersonation",
      "status": "open",
      "createdAt": "2025-03-27T12:10:00Z"
    }
  ],
  "nextCursor": null
}
```

#### Status Codes

- `200 OK` - Success
- `400 Bad Request` - Invalid `status` filter or pagination cursor
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated principal lacks moderator/admin role
- `503 Service Unavailable` - Dependency (DB) unavailable; safe to retry

---

### 7. Resolve or Dismiss an Abuse Report (Moderation)

**POST** `/username/reports/{reportId}/resolve`

Apply a moderation decision to an open report. Requires a moderator or admin
role. State transitions are one-way: `open → resolved` or `open → dismissed`.
A report that is already `resolved` or `dismissed` cannot be transitioned
again.

#### Request Body

```json
{
  "decision": "resolved",
  "note": "Confirmed impersonation; profile hidden.",
  "idempotencyKey": "7e6d5c4b-3a2f-4e1d-9c8b-7a6f5e4d3c2b"
}
```

#### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `decision` | string | ✅ Yes | One of `resolved`, `dismissed` |
| `note` | string | ❌ No | Moderator note (max 2000 chars) |
| `idempotencyKey` | string (UUID) | ✅ Yes | Client-generated key; replays return the original result |

#### Example Request

```bash
curl -X POST "http://localhost:3000/username/reports/c1a2b3c4-d5e6-4f70-8a9b-0c1d2e3f4a5b/resolve" \
  -H "Authorization: Bearer <moderator-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "decision": "resolved",
    "note": "Confirmed impersonation; profile hidden.",
    "idempotencyKey": "7e6d5c4b-3a2f-4e1d-9c8b-7a6f5e4d3c2b"
  }'
```

#### Example Response (Success)

```json
{
  "ok": true,
  "reportId": "c1a2b3c4-d5e6-4f70-8a9b-0c1d2e3f4a5b",
  "status": "resolved",
  "resolvedAt": "2025-03-27T12:20:00Z"
}
```

#### Status Codes

- `200 OK` - Decision applied (or idempotent replay of a prior success)
- `400 Bad Request` - Malformed payload, unknown `decision`, or missing idempotency key
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated principal lacks moderator/admin role
- `404 Not Found` - `reportId` does not exist
- `409 Conflict` - Report is not in `open` state (already resolved or dismissed)
- `503 Service Unavailable` - Dependency (DB) unavailable; safe to retry with the same idempotency key

#### State Transitions

- `open → resolved` (decision `resolved`)
- `open → dismissed` (decision `dismissed`)
- Any other transition returns `409 Conflict`; terminal states are immutable.

#### Idempotency & Rollback

- Every decision requires an `idempotencyKey`. Replaying the same key returns
  the original `200 OK` response without re-applying the transition.
- The status update and the moderation audit record are written in a single
  transaction. On dependency failure the transaction is rolled back and the
  report remains `open`.

#### Observability

- Structured log fields: `event=profile.report.resolve`, `reportId`,
  `decision`, `moderatorId` (hashed), `idempotencyKey`, `outcome`, `latencyMs`.
- Metrics: `profile_report_resolve_total{decision,outcome}`,
  `profile_report_resolve_latency_ms`.

---

## Field Descriptions

### PublicProfile Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `username` | string | Normalized username (lowercase) |
| `publicKey` | string | Stellar public key (G...) |
| `similarityScore` | number | Search relevance (0-100), only in search results |
| `transactionVolume` | number | Total USD volume, only in trending results |
| `transactionCount` | number | Number of transactions, only in trending results |
| `lastActiveAt` | string (ISO 8601) | Last activity timestamp |
| `createdAt` | string (ISO 8601) | Profile creation timestamp |

### AbuseReport Object

| Field | Type | Description |
|-------|------|-------------|
| `reportId` | UUID | Unique report identifier |
| `username` | string | Reported profile username |
| `category` | string | `spam`, `impersonation`, `fraud`, `harassment`, or `other` |
| `status` | string | `open`, `resolved`, or `dismissed` |
| `createdAt` | string (ISO 8601) | Report creation timestamp |
| `resolvedAt` | string (ISO 8601) | Decision timestamp, present once terminal |
