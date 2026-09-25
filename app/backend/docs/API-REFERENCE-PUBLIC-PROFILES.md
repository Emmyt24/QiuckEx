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
| `lastActiveAt` | ISO 8601 datetime | Last activity timestamp |
| `createdAt` | ISO 8601 datetime | Registration timestamp |

---

## Usage Notes

### Search Tips
- Minimum 2 characters required
- Case-insensitive (automatically normalized)
- Supports partial matches and typos
- Results ranked by similarity score
- Only includes profiles with `is_public=true`

### Trending Algorithm
- Based on actual payment transaction volume
- Counts both sender and receiver activity
- Volume measured in USD
- Only public profiles appear in results
- Real-time calculation (no caching)

### Privacy Controls
- Profiles are **private by default** (opt-in)
- Only wallet owners can toggle visibility
- Changes take effect immediately
- Hidden profiles won't appear in search or trending

### Rename & Redirects
- Renames preserve all existing payment links via permanent redirect aliases.
- Redirect aliases resolve to the same `publicKey`; they never change custody.
- Aliases are excluded from search and trending results.
- Renames are feature-gated: disabled on mainnet until the redirect registry
  migration is complete (see `docs/CAPABILITY-MAP.md`).

---

## Rate Limits

All endpoints are subject to rate limiting:
- Default: 10 requests/minute
- With API key: Higher limits apply
- Exceeding limits returns `429 Too Many Requests`

---

## Error Responses

### 400 Bad Request
```json
{
  "code": "USERNAME_INVALID_FORMAT",
  "message": "Search query must be at least 2 characters",
  "field": "query"
}
```

### 404 Not Found
```json
{
  "code": "USERNAME_NOT_FOUND",
  "message": "Username not found or does not belong to this wallet"
}
```

### 409 Conflict
```json
{
  "code": "USERNAME_ALREADY_TAKEN",
  "message": "New username is already registered or reserved as a redirect alias"
}
```

### 429 Too Many Requests
```json
{
  "statusCode": 429,
  "error": "Too Many Requests",
  "message": "Rate limit exceeded"
}
```

---

## Swagger Documentation

Interactive API documentation available at:
```
http://localhost:3000/api#/usernames
```

Features:
- Try it out directly in browser
- See all request/response schemas
- Download OpenAPI spec

---

## Code Examples

### JavaScript/Node.js

```javascript
// Search for profiles
async function searchProfiles(query, limit = 10) {
  const response = await fetch(
    `http://localhost:3000/username/search?query=${query}&limit=${limit}`
  );
  return await response.json();
}

// Get trending creators
async function getTrendingCreators(timeWindowHours = 24, limit = 10) {
  const response = await fetch(
    `http://localhost:3000/username/trending?timeWindowHours=${timeWindowHours}&limit=${limit}`
  );
  return await response.json();
}

// Toggle public profile
async function togglePublicProfile(username, publicKey, isPublic) {
  const response = await fetch(
    'http://localhost:3000/username/toggle-public',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, publicKey, isPublic }),
    }
  );
  return await response.json();
}

// Rename username (preserves existing payment links via redirect alias)
async function renameUsername(currentUsername, newUsername, publicKey, idempotencyKey) {
  const response = await fetch(
    'http://localhost:3000/username/rename',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ currentUsername, newUsername, publicKey, idempotencyKey }),
    }
  );
  return await response.json();
}
```

### Python

```python
import requests

BASE_URL = "http://localhost:3000"

def search_profiles(query, limit=10):
    response = requests.get(
        f"{BASE_URL}/username/search",
        params={"query": query, "limit": limit}
    )
    return response.json()

def get_trending_creators(time_window_hours=24, limit=10):
    response = requests.get(
        f"{BASE_URL}/username/trending",
        params={"timeWindowHours": time_window_hours, "limit": limit}
    )
    return response.json()

def toggle_public_profile(username, public_key, is_public):
    response = requests.post(
        f"{BASE_URL}/username/toggle-public",
        json={"username": username, "publicKey": public_key, "isPublic": is_public}
    )
    return response.json()

def rename_username(current_username, new_username, public_key, idempotency_key):
    response = requests.post(
        f"{BASE_URL}/username/rename",
        json={
            "currentUsername": current_username,
            "newUsername": new_username,
            "publicKey": public_key,
            "idempotencyKey": idempotency_key,
        },
    )
    return response.json()
```

### cURL Examples

```bash
# Search
curl "http://localhost:3000/username/search?query=alice&limit=5"

# Trending (last 7 days)
curl "http://localhost:3000/username/trending?timeWindowHours=168&limit=20"

# Enable public profile
curl -X POST "http://localhost:3000/username/toggle-public" \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","publicKey":"GBXG...","isPublic":true}'

# Disable public profile
curl -X POST "http://localhost:3000/username/toggle-public" \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","publicKey":"GBXG...","isPublic":false}'

# Rename username (old link keeps resolving via redirect alias)
curl -X POST "http://localhost:3000/username/rename" \
  -H "Content-Type: application/json" \
  -d '{"currentUsername":"alice","newUsername":"alice-co","publicKey":"GBXG...","idempotencyKey":"3f9c1e2a-7b4d-4c8e-9f01-2a3b4c5d6e7f"}'
```
