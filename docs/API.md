---
title: API Reference
lastUpdated: "2026-02-05"
status: current
---

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
| `POST` | `/api/v1/sessions` | Create phone verification session |
| `GET` | `/api/v1/sessions/:id` | Get session status |
| `POST` | `/api/v1/verification-requests` | Create multi-asset verification request |
| `GET` | `/api/v1/verification-requests` | List verification requests |
| `GET` | `/api/v1/verification-requests/:id` | Get verification request |
| `DELETE` | `/api/v1/verification-requests/:id` | Cancel verification request |
| `GET` | `/api/v1/verification-requests/by-reference/:referenceId` | Get by reference ID |
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
| `phone` | `whatsapp`, `telegram`, `viber`, `sms` |
| `email` | `email` |
| `domain` | `dns`, `http`, `email`, `auto` |
| `social` | `github`, `google`, `facebook`, `x`, `linkedin`, `instagram`, `youtube`, `tiktok` |
| `wallet` | `ethereum`, `solana`, `bitcoin` |
| `account` | `coinbase`, `kraken` |
### Example Request

```bash
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "type": "phone",
    "channel": "whatsapp",
    "identifier": "+37069199199",
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
  "channel": "whatsapp",
  "status": "pending",
  "identifier": "+37069199199",
  "challenge": {
    "code": "X7K2M9",
    "expires_at": "2026-02-04T11:00:00Z",
    "instruction": "Send X7K2M9 via WhatsApp",
    "deep_link": "https://wa.me/37069199199?text=X7K2M9"
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
  "channel": "whatsapp",
  "status": "verified",
  "identifier": "+37069199199",
  "verified_at": "2026-02-04T10:51:30Z",
  "proof": {
    "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_at": "2026-03-06T10:51:30Z"
  }
}
```

---

## Sessions (Phone-First Flow)

Sessions provide a phone-first verification flow where the user doesn't need to provide their phone number upfront. Instead, they send a message to your bot and the phone number is captured automatically.

### Create Session

`POST /api/v1/sessions`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channel` | string | Yes | `telegram` or `whatsapp` |
| `client_metadata` | object | No | Custom metadata for your reference |

```bash
curl -X POST https://api.proof.holdings/api/v1/sessions \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "telegram"
  }'
```

#### Response

```json
{
  "id": "sess_abc123",
  "channel": "telegram",
  "status": "pending",
  "deep_link": "https://t.me/bot?start=sess_abc123",
  "qr_code": "https://t.me/bot?start=sess_abc123",
  "expires_at": "2026-02-04T11:00:00Z",
  "instructions": "Open Telegram and tap the link to verify"
}
```

### Get Session

`GET /api/v1/sessions/:id`

Poll for session status. When verified, includes the phone number and proof token.

#### Response (Verified)

```json
{
  "id": "sess_abc123",
  "channel": "telegram",
  "status": "verified",
  "phone_number": "+15551234567",
  "verified_at": "2026-02-04T10:51:30Z",
  "verification_id": "507f1f77bcf86cd799439011",
  "proof": {
    "token": "eyJhbGciOiJSUzI1NiIs...",
    "expires_at": "2026-03-06T10:51:30Z"
  }
}
```

---

## Verification Requests (Multi-Asset)

Verification requests let you bundle multiple assets into a single verification flow. Useful for P2P trust, vendor onboarding, and KYB workflows.

### Create Verification Request

`POST /api/v1/verification-requests`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `assets` | array | Yes | Assets to verify (see below) |
| `reference_id` | string | No | Your external tracking ID |
| `action_type` | string | No | `verification`, `2fa`, `login`, `custom` |
| `action_context` | object | No | `{ title, description }` for display |
| `redirect_url` | string | No | Where to redirect after completion |
| `callback_url` | string | No | Webhook URL for completion notification |
| `expires_in` | number | No | Seconds until expiry (default 24h, max 7d) |
| `partial_ok` | boolean | No | Accept partial completion |

Each asset in the `assets` array:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `social`, `wallet` |
| `identifier` | string | No | Asset to verify (null = user provides) |
| `channel` | string | No | Preferred channel |
| `allowed_channels` | array | No | Restrict to specific channels |
| `required` | boolean | No | Whether this asset is required (default true) |

```bash
curl -X POST https://api.proof.holdings/api/v1/verification-requests \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "reference_id": "deal_12345",
    "assets": [
      { "type": "domain", "identifier": "example.com" },
      { "type": "email", "identifier": "ceo@example.com" }
    ],
    "action_type": "verification",
    "redirect_url": "https://your-app.com/verified"
  }'
```

#### Response

```json
{
  "id": "507f1f77bcf86cd799439011",
  "status": "pending",
  "reference_id": "deal_12345",
  "assets": [
    { "type": "domain", "identifier": "example.com", "status": "pending" },
    { "type": "email", "identifier": "ceo@example.com", "status": "pending" }
  ],
  "verification_url": "https://proof.holdings/verify/507f1f77bcf86cd799439011",
  "created_at": "2026-02-04T10:50:00Z",
  "expires_at": "2026-02-05T10:50:00Z"
}
```

### Get Verification Request

`GET /api/v1/verification-requests/:id`

### List Verification Requests

`GET /api/v1/verification-requests`

Query parameters: `page`, `limit` (max 100), `status`, `reference_id`

### Get by Reference ID

`GET /api/v1/verification-requests/by-reference/:referenceId`

### Cancel Verification Request

`DELETE /api/v1/verification-requests/:id`

### Verification Request Statuses

| Status | Description |
|--------|-------------|
| `pending` | Waiting for verifications to complete |
| `partial` | Some assets verified, others pending |
| `completed` | All required assets verified |
| `expired` | Request expired before completion |
| `cancelled` | Request was cancelled |

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
    "channel": "whatsapp",
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
