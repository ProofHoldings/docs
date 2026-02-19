---
title: API Reference
lastUpdated: "2026-02-18"
status: current
---

# API Reference

Base URL: `https://api.proof.holdings`

Authentication: Bearer token via `Authorization` header

```
Authorization: Bearer pk_live_your_api_key_here
```

API keys are prefixed with `pk_live_` for production and `pk_test_` for testing.

> **Machine-readable spec:** The full OpenAPI 3.0 specification is available at [`/api/openapi.json`](https://api.proof.holdings/api/openapi.json) for SDK generation, contract testing, and AI coding assistants.

---

## Endpoints Overview

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/api/v1/verifications` | Create verification challenge |
| `GET` | `/api/v1/verifications` | List verifications |
| `GET` | `/api/v1/verifications/:id` | Get verification status |
| `POST` | `/api/v1/verifications/:id/submit` | Submit challenge code |
| `POST` | `/api/v1/verifications/:id/verify` | Trigger DNS/HTTP check |
| `POST` | `/api/v1/verifications/:id/resend` | Resend verification (email only) |
| `POST` | `/api/v1/verifications/:id/test-verify` | Auto-verify (test mode only) |
| `GET` | `/api/v1/verifications/:id/events` | Stream status updates (SSE) |
| `GET` | `/api/v1/verifications/users` | List verified users (B2B) |
| `GET` | `/api/v1/verifications/users/:externalUserId` | Get verified user details (B2B) |
| `POST` | `/api/v1/verifications/domain` | Start domain verification (B2B) |
| `POST` | `/api/v1/verifications/domain/:id/check` | Check domain verification (B2B) |
| `POST` | `/api/v1/proofs/validate` | Validate proof token |
| `GET` | `/api/v1/proofs/:id/status` | Check proof status |
| `POST` | `/api/v1/proofs/:id/revoke` | Revoke a proof |
| `GET` | `/api/v1/proofs/revoked` | Get revocation list |
| `POST` | `/api/v1/sessions` | Create phone verification session |
| `GET` | `/api/v1/sessions/:id` | Get session status |
| `POST` | `/api/v1/verification-requests` | Create multi-asset verification request |
| `GET` | `/api/v1/verification-requests` | List verification requests |
| `GET` | `/api/v1/verification-requests/:id` | Get verification request |
| `GET` | `/api/v1/verification-requests/by-reference/:referenceId` | Get by reference ID |
| `DELETE` | `/api/v1/verification-requests/:id` | Cancel verification request |
| `GET` | `/api/v1/webhook-deliveries` | List webhook deliveries |
| `GET` | `/api/v1/webhook-deliveries/:id` | Get webhook delivery details |
| `GET` | `/api/v1/webhook-deliveries/stats` | Get delivery statistics |
| `POST` | `/api/v1/webhook-deliveries/:id/retry` | Retry failed delivery |
| `GET` | `/.well-known/jwks.json` | Public keys (RS256) |
| `GET` | `/health` | Service status |

---

## Test Mode (Sandbox)

Use test mode to build and test your integration without sending real messages, consuming quota, or incurring charges. Test mode is activated automatically when you authenticate with a **test API key** (`pk_test_*`).

### How It Works

| Aspect | Production (`pk_live_*`) | Test Mode (`pk_test_*`) |
|--------|--------------------------|-------------------------|
| Channel delivery | Real messages sent | Skipped — no real messages |
| Quota usage | Counted toward monthly limit | Not counted |
| Billing | Charged per proof | Free |
| Webhooks | Delivered normally | Delivered with `"test": true` flag |
| Data isolation | Only sees production verifications | Only sees test verifications |
| Proof tokens | Valid, signed JWT | Valid, signed JWT (identical format) |

### Auto-Verify Endpoint

`POST /api/v1/verifications/:id/test-verify`

Instantly complete a verification without user action. Only available with test API keys (`pk_test_*`). Returns `403` for production keys.

```json
{
  "id": "507f1f77bcf86cd799439011",
  "type": "phone",
  "channel": "whatsapp",
  "status": "verified",
  "identifier": "+37069199199",
  "verified_at": "2026-02-13T12:25:00Z",
  "proof_token": "eyJhbGciOiJSUzI1NiIs...",
  "proof_expires_at": "2026-03-15T12:25:00Z",
  "test_mode": true
}
```

### Example: Full Test Flow

```bash
# 1. Create verification with test key (no real message sent)
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_test_..." \
  -H "Content-Type: application/json" \
  -d '{"type":"phone","channel":"whatsapp","identifier":"+37069199199"}'

# 2. Auto-verify (skip waiting for user action)
curl -X POST https://api.proof.holdings/api/v1/verifications/{id}/test-verify \
  -H "Authorization: Bearer pk_test_..."

# 3. Use the proof token exactly as you would in production
```

### Webhook Behavior

Test mode webhooks are delivered normally but include `"test": true` in the payload:

```json
{
  "event": "verification_request.completed",
  "test": true,
  "request_id": "req_abc123",
  "status": "completed",
  "proofs": [...]
}
```

### Environment Isolation

Test and production data are completely isolated. A `pk_test_` key can only see test verifications, and a `pk_live_` key can only see production verifications.

---

## Create Verification

`POST /api/v1/verifications`

Create a new verification challenge.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `social`, `wallet`, `waba`, `telegram_bot` |
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
| `waba` | `waba_otp` |
| `telegram_bot` | `telegram_bot_token` |
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

## Submit Challenge Code

`POST /api/v1/verifications/:id/submit`

Submit an OTP challenge code for email/phone verification.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `challenge_code` | string | Yes | The 6-character challenge code |
| `source_identifier` | string | No | Phone/email that sent the code (for validation) |

### Response

Returns the updated verification object (same as Get Verification).

---

## Resend Verification

`POST /api/v1/verifications/:id/resend`

Resend the verification challenge. Currently only supported for email verifications. The verification must be in `pending` status.

### Response

```json
{
  "success": true,
  "message": "Verification email sent",
  "expires_at": "2026-02-13T12:10:00Z"
}
```

---

## Verification Events (SSE)

`GET /api/v1/verifications/:id/events`

Subscribe to real-time verification status updates via Server-Sent Events. Use this instead of polling `GET /verifications/:id`.

### Usage

```javascript
const source = new EventSource(
  'https://api.proof.holdings/api/v1/verifications/{id}/events',
  { headers: { 'Authorization': 'Bearer pk_live_...' } }
);

source.addEventListener('connected', (e) => {
  console.log('Connected:', JSON.parse(e.data));
});

source.addEventListener('status_changed', (e) => {
  const update = JSON.parse(e.data);
  if (update.status === 'verified') {
    source.close();
  }
});
```

### Events

| Event | Description |
|-------|-------------|
| `connected` | Initial connection established |
| `status_changed` | Verification status updated |

A 30-second heartbeat keeps the connection alive through Cloudflare. Maximum concurrent SSE connections per tenant is configurable (default: 10).

---

## Verified Users (B2B)

### List Verified Users

`GET /api/v1/verifications/users`

List verified users grouped by `external_user_id`. Only includes verifications where `external_user_id` was set during creation.

Query parameters: `page`, `limit` (max 100)

#### Response

```json
{
  "data": [
    {
      "external_user_id": "user_12345",
      "verification_count": 3,
      "types_verified": ["phone", "email"],
      "verifications": [
        { "id": "507f...", "type": "phone", "channel": "whatsapp", "identifier": "+3706***9199", "verified_at": "2026-02-13T12:00:00Z" }
      ],
      "first_verified_at": "2026-01-10T08:00:00Z",
      "last_verified_at": "2026-02-13T12:00:00Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 42, "pages": 3 }
}
```

### Get Verified User

`GET /api/v1/verifications/users/:externalUserId`

Get all verifications for a specific external user.

#### Response

```json
{
  "external_user_id": "user_12345",
  "verification_count": 3,
  "types_verified": ["phone", "email"],
  "verifications": [
    {
      "id": "507f...",
      "type": "phone",
      "channel": "whatsapp",
      "identifier": "+3706***9199",
      "status": "verified",
      "verified_at": "2026-02-13T12:00:00Z",
      "has_proof": true,
      "proof_expires_at": "2026-03-15T12:00:00Z",
      "created_at": "2026-02-13T11:50:00Z"
    }
  ]
}
```

---

## Domain Verification (B2B)

### Start Domain Verification

`POST /api/v1/verifications/domain`

Start a domain verification flow. Returns DNS and HTTP instructions for the domain owner.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | Yes | Domain to verify (e.g., `example.com`) |
| `customer_id` | string | No | Optional customer identifier |
| `verification_method` | string | No | `manual_dns` (default) or `http_file` |

#### Response

```json
{
  "id": "507f1f77bcf86cd799439011",
  "domain": "example.com",
  "status": "pending",
  "verification_method": "manual_dns",
  "dns_record": {
    "type": "TXT",
    "name": "_proof.example.com",
    "value": "proof-verify=abc123"
  },
  "http_file": {
    "path": "/.well-known/proof-verify.txt",
    "content": "proof-verify=abc123"
  },
  "provider": {
    "detected": "cloudflare",
    "name": "Cloudflare"
  }
}
```

### Check Domain Verification

`POST /api/v1/verifications/domain/:id/check`

Check if DNS TXT record or HTTP file has been set up correctly.

#### Response

```json
{
  "id": "507f1f77bcf86cd799439011",
  "domain": "example.com",
  "status": "verified",
  "verified": true,
  "verified_at": "2026-02-13T12:00:00Z",
  "check_count": 1
}
```

---

## Proof Status

`GET /api/v1/proofs/:id/status`

Quick status check for a proof without validating the full JWT. Returns whether the proof is valid, revoked, or expired.

### Response

```json
{
  "proof_id": "507f1f77bcf86cd799439011",
  "status": "verified",
  "is_valid": true,
  "is_revoked": false,
  "revoked_at": null,
  "revoked_reason": null,
  "expires_at": "2026-03-15T12:00:00Z",
  "is_expired": false
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
  "phone_number": "+37069199199",
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
| `type` | string | Yes | `phone`, `email`, `domain`, `social`, `wallet`, `waba`, `telegram_bot` |
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

## Webhook Deliveries

Monitor and manage webhook deliveries for your account. Webhooks are sent when verification requests complete, partially complete, or expire.

### List Webhook Deliveries

`GET /api/v1/webhook-deliveries`

Query parameters: `page`, `limit` (max 100), `status` (`pending`, `delivered`, `failed`), `event_type`, `verification_request_id`

#### Response

```json
{
  "data": [
    {
      "id": "507f1f77bcf86cd799439011",
      "verification_request_id": "507f1f77bcf86cd799439012",
      "event_type": "verification_request.completed",
      "url": "https://yourapp.com/webhook",
      "status": "delivered",
      "attempts": 1,
      "max_attempts": 3,
      "response_status": 200,
      "created_at": "2026-02-13T12:00:00Z",
      "updated_at": "2026-02-13T12:00:01Z"
    }
  ],
  "pagination": { "hasMore": false }
}
```

### Get Webhook Delivery

`GET /api/v1/webhook-deliveries/:id`

Returns full delivery details including request payload and response body.

### Get Delivery Statistics

`GET /api/v1/webhook-deliveries/stats`

```json
{
  "total": 150,
  "delivered": 142,
  "failed": 5,
  "pending": 3,
  "success_rate": 94.67
}
```

### Retry Failed Delivery

`POST /api/v1/webhook-deliveries/:id/retry`

Retry a failed webhook delivery. Only available for deliveries with status `failed`.

### Webhook Event Types

| Event | Description |
|-------|-------------|
| `verification_request.completed` | All required assets verified |
| `verification_request.partial` | Some assets verified (when `partial_ok` is true) |
| `verification_request.expired` | Request expired before completion |
| `verification.completed` | Individual verification completed |

### Webhook Signature

Webhooks include an HMAC-SHA256 signature in the `X-Proof-Signature` header. Verify this against your webhook secret to ensure authenticity.

```javascript
const crypto = require('crypto');

function verifyWebhook(payload, signature, secret) {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expected)
  );
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

---

## See Also

| Feature | Endpoints | Documentation |
|---------|-----------|---------------|
| **Smart Reuse** | `GET /api/v1/verify/request/:id/proofs`, `POST .../auto-verify` | [Smart Reuse](/docs/smart-reuse) |
| **Multi-Profile** | `GET/POST /api/v1/me/profiles`, `PATCH/DELETE .../profiles/:id` | [Multi-Profile System](/docs/profiles) |
| **Message Templates** | `GET/POST /api/v1/me/projects`, `PUT .../templates/:channel/:type` | [Message Templates](/docs/templates) |
