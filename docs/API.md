# API Reference

Base URL: `https://api.proof.holdings`

Authentication: Bearer token via `Authorization` header

```
Authorization: Bearer pk_live_your_api_key_here
```

API keys are prefixed with `pk_live_` for production and `pk_test_` for testing.

---

## Endpoints Overview

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/api/v1/verifications` | Create verification challenge |
| `GET` | `/api/v1/verifications/:id` | Get verification status |
| `POST` | `/api/v1/verifications/:id/verify` | Trigger DNS/HTTP check |
| `GET` | `/api/v1/verifications` | List verifications |
| `POST` | `/api/v1/proofs/validate` | Validate proof token |
| `POST` | `/api/v1/proofs/:id/revoke` | Revoke a proof |
| `GET` | `/api/v1/proofs/revoked` | Get revocation list |
| `GET` | `/api/v1/proofs/:id/status` | Check if proof is revoked |
| `GET` | `/.well-known/jwks.json` | Public keys (RS256) |
| `GET` | `/health` | Service status |

---

## Create Verification

`POST /api/v1/verifications`

Create a new verification challenge.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `social`, `wallet` |
| `channel` | string | Yes | Channel for verification (see table below) |
| `identifier` | string | Yes | The asset to verify |
| `client_metadata` | object | No | Custom metadata for your reference |

### Type/Channel Compatibility

| Type | Valid Channels |
|------|----------------|
| `phone` | `telegram`, `sms`, `whatsapp` |
| `email` | `email` |
| `domain` | `dns`, `http`, `email`, `auto` |
| `social` | `github`, `google`, `discord`, etc. (coming soon) |
| `wallet` | `ethereum`, `solana`, `bitcoin` (coming soon) |

### Example Request

```bash
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "type": "phone",
    "channel": "telegram",
    "identifier": "+15551234567",
    "client_metadata": {
      "user_id": "usr_123",
      "action": "login"
    }
  }'
```

### Response

```json
{
  "id": "507f1f77bcf86cd799439011",
  "type": "phone",
  "channel": "telegram",
  "status": "pending",
  "identifier": "+15551234567",
  "challenge": {
    "code": "X7K2M9",
    "expires_at": "2026-02-04T11:00:00Z",
    "send_to": "@proof_holdings_bot",
    "instruction": "Send the code to our Telegram bot",
    "deep_link": "https://t.me/proof_holdings_bot?start=X7K2M9"
  },
  "created_at": "2026-02-04T10:50:00Z"
}
```

---

## Get Verification

`GET /api/v1/verifications/:id`

Poll for verification status.

### Response (Verified)

```json
{
  "id": "507f1f77bcf86cd799439011",
  "type": "phone",
  "channel": "telegram",
  "status": "verified",
  "identifier": "+15551234567",
  "verified_at": "2026-02-04T10:51:30Z",
  "proof": {
    "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_at": "2026-03-06T10:51:30Z"
  }
}
```

---

## Validate Proof

`POST /api/v1/proofs/validate`

Validate a proof token via API.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `proof_token` | string | Yes | The JWT proof token |
| `identifier` | string | No | Optional identifier to verify against |

### Response (Valid)

```json
{
  "valid": true,
  "verification": {
    "id": "507f1f77bcf86cd799439011",
    "type": "phone",
    "channel": "telegram",
    "verified_at": "2026-02-04T10:51:30Z",
    "expires_at": "2026-03-06T10:51:30Z"
  }
}
```

### Response (Invalid)

```json
{
  "valid": false,
  "reason": "expired",
  "message": "jwt expired"
}
```

---

## Offline Verification

Proofs are signed with RS256 and can be verified without calling our API.

### 1. Fetch Public Key (cache 24h)

```bash
curl https://api.proof.holdings/.well-known/jwks.json
```

```json
{
  "keys": [{
    "kty": "RSA",
    "n": "0vx7agoebGcQ...",
    "e": "AQAB",
    "alg": "RS256",
    "use": "sig",
    "kid": "proof-holdings-2026-01"
  }]
}
```

### 2. Fetch Revocation List (cache 5min)

```bash
curl https://api.proof.holdings/api/v1/proofs/revoked
```

### 3. Verify Locally

```javascript
import jwt from 'jsonwebtoken';

// Verify signature
const payload = jwt.verify(proofToken, publicKey, { algorithms: ['RS256'] });

// Check not revoked
const isRevoked = revokedList.some(r => r.proof_id === payload.sub);

// Check not expired
const isValid = !isRevoked && payload.exp > Date.now() / 1000;
```

---

## Revoke Proof

`POST /api/v1/proofs/:id/revoke`

Revoke a proof (GDPR deletion, fraud, etc).

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | Yes | `user_request`, `fraud`, `key_compromise`, `asset_transferred`, `other` |

### Response

```json
{
  "success": true,
  "verification_id": "507f1f77bcf86cd799439011",
  "status": "revoked",
  "revoked_at": "2026-02-04T12:00:00Z"
}
```

---

## Verification Statuses

| Status | Description |
|--------|-------------|
| `pending` | Challenge created, waiting for user |
| `verified` | Successfully verified, proof available |
| `failed` | Max attempts exceeded |
| `expired` | Challenge expired |
| `revoked` | Proof was revoked |

---

## Error Responses

All errors return:

```json
{
  "error": {
    "code": "error_code",
    "message": "Human-readable message"
  }
}
```

| Code | HTTP | Description |
|------|------|-------------|
| `validation_error` | 400 | Invalid request parameters |
| `invalid_identifier` | 400 | Identifier format is invalid |
| `invalid_channel_for_type` | 400 | Channel doesn't match type |
| `unauthorized` | 401 | Invalid or missing API key |
| `not_found` | 404 | Resource not found |
| `challenge_expired` | 400 | Verification challenge expired |
| `rate_limited` | 429 | Too many requests |

---

## Rate Limits

- 100 requests/minute per API key
- Webhook deliveries: automatic retry with exponential backoff
