# REST API Reference

**Base URL:** `https://api.orbitstream.xyz/api/v1`  
**Auth:** `Authorization: Bearer <token>` on all routes except `/auth/login`

---

## Authentication

### `POST /auth/login`

Wallet-signed login. Returns a JWT.

**Request**
```json
{
  "walletAddress": "GAXYZ...3A4B",
  "signature": "optional-ed25519-signature"
}
```

**Response**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "wallet": "GAXYZ...3A4B"
}
```

---

## Streams

### `POST /streams` — Create stream

**Request**
```json
{
  "recipient": "GBCD...7E2F",
  "tokenAddress": "CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA",
  "ratePerSecond": 347222,
  "totalDeposited": 1000000000,
  "durationSeconds": 2592000
}
```

**Response** — `Stream` object (see [Stream Schema](#stream-schema))

---

### `GET /streams` — List my streams

Query parameters:

| Param | Values | Description |
|-------|--------|-------------|
| `status` | `active`, `paused`, `cancelled`, `completed` | Filter by status |
| `role` | `sender`, `recipient` | Filter by wallet role |

**Response** — Array of `Stream` objects

---

### `GET /streams/:id` — Get stream

**Response** — `Stream` object

---

### `GET /streams/:id/claimable` — Claimable now

**Response**
```json
{
  "streamId": "550e8400-e29b-41d4-a716-446655440000",
  "claimable": 1234567,
  "earned": 9876543
}
```

---

### `PATCH /streams/:id/claim` — Record a claim

**Request**
```json
{
  "txHash": "abc123def456..."
}
```

**Response** — Updated `Stream` object

---

### `PATCH /streams/:id/pause` — Pause stream

No request body required. Caller must be the stream sender.

---

### `PATCH /streams/:id/resume` — Resume stream

No request body required. Caller must be the stream sender.

---

### `DELETE /streams/:id` — Cancel stream

No request body required. Caller must be the stream sender.

---

## Monitoring

### `GET /health`

**Response**
```json
{
  "status": "ok",
  "info": { "database": { "status": "up" } }
}
```

### `GET /metrics`

Prometheus text format. Metrics include:

- `orbitstream_streams_created_total` — total streams created
- `orbitstream_tokens_claimed_total` — total token units claimed
- `orbitstream_request_duration_seconds` — request latency histogram

---

## Stream Schema

```typescript
interface Stream {
  id:              string;           // UUID
  sender:          string;           // Stellar address
  recipient:       string;           // Stellar address
  tokenAddress:    string;           // Stellar asset contract
  ratePerSecond:   number;           // Token units per second
  totalDeposited:  number;
  totalClaimed:    number;
  contractStreamId: string | null;   // On-chain stream ID
  txHash:          string | null;    // Last transaction hash
  status:          'active' | 'paused' | 'cancelled' | 'completed';
  startTime:       number;           // Unix timestamp
  endTime:         number | null;    // null = open-ended
  createdAt:       string;           // ISO 8601
  updatedAt:       string;           // ISO 8601
}
```

---

## Error Responses

All errors follow this shape:

```json
{
  "statusCode": 404,
  "message": "Stream not found",
  "error": "Not Found"
}
```

| Status | Meaning |
|--------|---------|
| 400 | Bad request / validation error |
| 401 | Missing or invalid JWT |
| 403 | Caller is not the stream owner |
| 404 | Stream not found |
| 500 | Internal server error |
