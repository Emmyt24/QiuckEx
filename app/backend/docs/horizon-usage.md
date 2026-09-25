# Horizon Usage Guidelines

This document outlines how to fetch Stellar transaction data via Horizon in the QuickEx backend.

## Overview

The `HorizonService` provides a centralized way to interact with the Stellar Horizon API. It uses the `stellar-sdk` and implements in-memory caching to reduce latency and avoid rate limits.

## Fetching Payments

To get reliable amount and asset data, we fetch **operations** of type `payment`, `path_payment_strict_receive`, and `path_payment_strict_send` rather than raw transactions.

### Endpoint

`GET /transactions`

### Query Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `accountId` | string | Yes | Stellar public key (G...) |
| `asset` | string | No | Filter by asset (`XLM` or `CODE:ISSUER`) |
| `limit` | number | No | Max items to return (1-200, default 20) |
| `cursor` | string | No | Pagination token (`paging_token`) |

## Caching

Results are cached for **60 seconds** in memory using an LRU cache. The cache key includes the network, account ID, asset filter, limit, and cursor.

## Rate Limiting

If Horizon returns a `429 Too Many Requests` error, the backend will return a `503 Service Unavailable` response. Clients should implement their own backoff strategy and avoid aggressive polling.

## Transaction Simulation Hardening (Issue #214)

Transaction simulation must be validated against the **active network** and the **expected asset registry** before any result is trusted. Mismatches are rejected with stable error codes so callers can branch deterministically.

### Network matching

- The requested network (e.g. `PUBLIC`, `TESTNET`, `FUTURENET`) MUST equal the configured/active network for the running deployment.
- A mismatch returns `SIM_NETWORK_MISMATCH` and the simulation is not executed.
- Mainnet-only behavior is feature-gated: when the active network is not `PUBLIC`, mainnet-gated simulation paths return `SIM_FEATURE_GATED` instead of silently degrading.

### Asset matching

- Asset identifiers are compared against the expected asset registry using the canonical form:
  - Native: `XLM`
  - Issued: `CODE:ISSUER` (issuer validated with `StrKey.isValidEd25519PublicKey`)
  - Contract/SAC: `C...` contract address
- A mismatch (wrong issuer, wrong code, wrong SAC address, or unknown asset) returns `SIM_ASSET_MISMATCH`.
- Malformed asset identifiers return `SIM_ASSET_MALFORMED`.

### Stable error codes

| Code | Meaning |
| :--- | :--- |
| `SIM_NETWORK_MISMATCH` | Requested network does not match the active network. |
| `SIM_ASSET_MISMATCH` | Asset does not match the expected registry entry. |
| `SIM_ASSET_MALFORMED` | Asset identifier is malformed or unparseable. |
| `SIM_UNAUTHORIZED` | Caller is not authorized to simulate for the requested account. |
| `SIM_DUPLICATE` | Duplicate simulation request (idempotency key already seen). |
| `SIM_EXPIRED` | Simulation request or referenced ledger window has expired. |
| `SIM_DEPENDENCY_FAILURE` | Horizon or another upstream dependency failed. |
| `SIM_FEATURE_GATED` | Requested behavior is not enabled on the active network. |

### Idempotency, expiry, and degraded mode

- Simulation requests carry an idempotency key; replays return the original result or `SIM_DUPLICATE`.
- Requests older than the configured simulation TTL return `SIM_EXPIRED`.
- When Horizon is unavailable, simulation returns `SIM_DEPENDENCY_FAILURE` (mapped from upstream `503`) rather than a partial result.

### Observability

- Structured logs record `network`, `asset`, `errorCode`, and `latencyMs` for each simulation.
- Metrics track success rate, latency, and failure counts by `errorCode`.
- Logs MUST NOT include secrets, private keys, or unnecessary personal data.

## Example Usage

```typescript
// In a controller or service
const transactions = await this.horizonService.getPayments(
  'GD...',
  'USDC:GA...',
  20,
  '123456789'
);
```

## Best Practices

1.  **Always use the operations endpoint** for payment data.
2.  **Use pagination** instead of high limits to avoid long response times.
3.  **Validate account IDs** using `StrKey.isValidEd25519PublicKey`.
4.  **Validate network and asset identifiers** before trusting any simulation result (see above).
