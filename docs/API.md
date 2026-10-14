---
title: API Reference
lastUpdated: "2026-08-26"
status: current
---

# API Reference

Base URL: `https://api.proof.holdings`

Authentication: Bearer token via `Authorization` header

```
Authorization: Bearer pk_live_your_api_key_here
```

API keys are prefixed with `pk_live_` for production and `pk_test_` for testing.

> **Machine-readable spec:** The full OpenAPI 3.1 specification is available at [`/api/openapi.json`](https://api.proof.holdings/api/openapi.json) for SDK generation, contract testing, and AI coding assistants.

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
| `POST` | `/api/v1/delegations` | Create a delegation (mint token) |
| `GET` | `/api/v1/delegations` | List delegations |
| `GET` | `/api/v1/delegations/:id` | Get a delegation incl. its token |
| `POST` | `/api/v1/delegations/:id/revoke` | Revoke a delegation |
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
| `GET` | `/.well-known/jwks.json` | Public keys (ES256) |
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
  "proof_token": "eyJhbGciOiJFUzI1NiIs...",
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
| `type` | string | Yes | `phone`, `email`, `domain`, `social`, `wallet`, `telegram_bot` |
| `channel` | string | Yes | Channel for verification (see table below) |
| `identifier` | string | Yes | The asset to verify |
| `client_metadata` | object | No | Custom metadata for your reference |

### Type/Channel Compatibility

| Type | Valid Channels |
|------|----------------|
| `phone` | `whatsapp`, `telegram`, `sms` |
| `email` | `email` |
| `domain` | `dns`, `http`, `email`, `auto` |
| `social` | `github`, `google`, `facebook`, `x`, `linkedin`, `instagram`, `youtube`, `tiktok` |
| `wallet` | `ethereum`, `solana`, `bitcoin` |
| `account` | `coinbase`, `kraken` |
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
    "token": "eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9...",
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

Quick status check for a proof without validating the full JWT. Returns whether the proof is valid, suspended, revoked, or expired.

`:id` accepts the public proof handle (`ph_ctl_…`, the `proof_id` claim of a proof token, or `ph_dlg_…` for a delegation) or a verification id. A live proof reads `active` — the same word `POST /api/v1/proofs/:id/unsuspend` and `GET /api/v1/delegations/:id` use.

### Response

```json
{
  "proof_id": "ph_ctl_0123456789abcdef0123456789abcdef",
  "status": "active",
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
    "token": "eyJhbGciOiJFUzI1NiIs...",
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
| `expires_in` | number | No | Seconds until expiry (default 604800 = 7 days, max 2592000 = 30 days) |
| `partial_ok` | boolean | No | Accept partial completion |

Each asset in the `assets` array:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `social`, `wallet`, `telegram_bot` |
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

Proofs are signed with ES256 and can be verified without calling our API.

### 1. Fetch Public Key (cache 24h)

```bash
curl https://api.proof.holdings/.well-known/jwks.json
```

```json
{
  "keys": [{
    "kty": "EC",
    "crv": "P-256",
    "x": "f83OJ3D2xF1Bg8vub9tLe1gHMzV76e8Tus9uPHvRVEU",
    "y": "x_FEzRu9m36HLN_tue659LNpXW6pCyStikYjKIWI5a0",
    "alg": "ES256",
    "use": "sig",
    "kid": "proof-holdings-2026-01"
  }]
}
```

### 2. Fetch Revocation List

```bash
curl https://api.proof.holdings/api/v1/proofs/revoked
```

The list carries two lifetimes that answer different questions. Reuse a fetched list for up to 5 minutes — its cache lifetime (`Cache-Control: max-age=300`) — then fetch it again to see new revocations. If you cannot reach the API, the list's `signature` stays verifiable for 1 hour, so the copy you hold keeps working offline for that long; after that, fetch a fresh one.

### 3. Verify Locally

```javascript
import jwt from 'jsonwebtoken';
import { createPublicKey } from 'node:crypto';

// Pick the key the token names. During a rotation the set carries two — the current EC key
// and the retiring RSA one — so taking keys[0] would verify against the wrong key for every
// token minted under the other.
const { kid } = JSON.parse(
  Buffer.from(proofToken.split('.')[0], 'base64url').toString()
);
const jwk = jwks.keys.find(k => k.kid === kid);
const publicKey = createPublicKey({ key: jwk, format: 'jwk' });

// New tokens are ES256; the retiring key's algorithm stays accepted until its last token expires.
const payload = jwt.verify(proofToken, publicKey, { algorithms: ['ES256', 'RS256'] });

// Check not revoked
const isRevoked = revokedList.some(r => r.proof_id === payload.sub);

// Check not expired
const isValid = !isRevoked && payload.exp > Date.now() / 1000;
```

---

## Revoke Proof

`POST /api/v1/proofs/:id/revoke`

Revoke a proof (GDPR deletion, fraud, etc).

`:id` accepts the public proof handle (`ph_ctl_…` / `ph_dlg_…`) or a verification id — the same addresses `/status`, `/suspend` and `/unsuspend` take. The revocation reaches the registry, `GET /api/v1/proofs/:id/status`, the public `POST /api/v1/proofs/validate`, the revocation list and the Token Status List.

This revokes the **proof**, not the thing the proof is about: the verified asset, the consent or the delegated artifact keeps its own state and its own surface. Revoking a proof whose subject is not a verification (a domain asset, a request asset, a consent) works only through the handle — there is no other id for it.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | Yes | `user_request`, `fraud`, `key_compromise`, `asset_transferred`, `other` |

### Response

```json
{
  "success": true,
  "proof_id": "ph_ctl_0123456789abcdef0123456789abcdef",
  "proof_type": "verification",
  "verification_id": "507f1f77bcf86cd799439011",
  "revoked_at": "2026-02-04T12:00:00Z",
  "reason": "user_request"
}
```

`proof_id` and `proof_type` are present when the proof was addressed by handle; `verification_id` only when the proof belongs to a verification. A proof whose revocation cannot be written answers `503 proof_revocation_failed` rather than reporting a success it did not perform.

**Who can call it.** An API key with `proofs:write`, or a signed-in dashboard session — the same dual-auth the delegation endpoints use. The scopes bind API keys only; a session owns the account outright. This applies to `/status`, `/revoke`, `/suspend` and `/unsuspend` alike.

**A second revoke means different things on the two resources, deliberately.**

| Resource | Repeat revoke | Why |
|----------|---------------|-----|
| Proof (`ph_ctl_…`, verification id) | `400 already_revoked` with the FIRST revocation's `revoked_at` | The code is published and read by integrators; changing it would break callers that branch on it |
| Delegation (`ph_dlg_…`) | `200` with the stored revocation, on this endpoint AND on `POST /api/v1/delegations/:id/revoke` | Idempotency is what the delegation contract has always advertised, in these docs and in all four SDKs |

Either way the answer never depends on WHICH address you used — one delegation used to answer `200` through its own route and `400` here, and that is fixed. A repeat call reports the revocation that actually happened: the original timestamp and the original reason, never this call's.

**Scopes on a delegation handle.** Any lifecycle act on a `ph_dlg_…` — revoke, suspend or unsuspend — additionally requires `delegations:write` on top of the route's own `proofs:write`, so the wider group is not a way around the narrower one. Suspending used to need only `proofs:write`, on the argument that a pause is lifted by the same scope that applied it; what a third party sees while the pause holds is what it sees after a revocation (the token stops verifying, the status list says so), and being reversible bounds the cost of putting it back rather than the reach or the duration of the act. Acting on a domain control proof (`ph_ctl_…`) is unchanged and still needs `proofs:write` alone.

---

## Delegations

Proof of Delegation: a control-proven domain authorizes a typed artifact — a package (`purl`) or an https endpoint (`url`) — for a set of capability scopes. Each create mints an independently revocable ES256 delegation token, signed with the same key and published through the same JWKS as proof tokens. Requires `delegations:*` API-key scopes; each mint is one billable proof.

### Create Delegation

`POST /api/v1/delegations`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `control_proof` | string | Yes | Public handle of YOUR verified domain proof (`ph_ctl_…`). The delegation's `principal` is the domain that proof covers. |
| `delegate` | object | Yes | The authorized artifact: `{ "type": "purl" \| "url", "value": "pkg:npm/your-mcp-server" \| "https://your-agent.example/api" }`. Versions, qualifiers, query strings and fragments are rejected — a delegation names the artifact, not a release. |
| `scope` | string[] | Yes | 1–32 lowercase kebab-case capability tokens, ≤64 characters each (e.g. `send-email`). These are **not** API scopes — an API-style `resource:action` such as `payments:read` is rejected with `400 invalid_scope`. Verifiers compare by exact string match — there is no wildcard. |
| `expires_in` | number | No | Token lifetime in seconds. Capped by the control proof's own expiry — a delegation never outlives the domain proof under it. |

This bound holds after issuance too: if you later renew your domain proof for a shorter term
than it had before, the renewal call still succeeds, but whenever the requested duration would
drop the domain proof's expiry below an already-issued delegation's own `expires_at`, that
duration is not honoured — the domain proof is kept at (at least) the longest-lived delegation's
expiry instead, with no error returned. A delegate's token never silently outlives the domain
proof it depends on.

#### Response

```json
{
  "id": "60d5ecb74f421b2d8c9e1234",
  "delegation_id": "ph_dlg_0123456789abcdef0123456789abcdef",
  "principal": "yourdomain.com",
  "delegate": { "type": "purl", "value": "pkg:npm/your-mcp-server" },
  "scope": ["send-email"],
  "control_proof": "ph_ctl_0123456789abcdef0123456789abcdef",
  "status": "active",
  "environment": "production",
  "issued_at": "2026-02-04T12:00:00Z",
  "expires_at": "2027-02-04T12:00:00Z",
  "created_at": "2026-02-04T12:00:00Z",
  "token": "eyJhbGciOiJFUzI1NiIs..."
}
```

### Manage Delegations

- `GET /api/v1/delegations` — paginated list. Tokens are **not** included in lists. Every item carries `effective_status` and `is_valid` beside its stored `status` — **read `effective_status`**: `status` records only whether *you* revoked the delegation, so one whose domain control proof was revoked, suspended or expired — or whose own lifetime simply ran out — still reads `active` there while every verifier already rejects it. `effective_status` covers both clocks: a delegation is never valid for longer than the earlier of its own `expires_at` and the control proof's. The `?status=` filter matches the stored record only (the cascade spans two collections and cannot be filtered server-side without breaking pagination), so `?status=active` can legitimately return a delegation whose `effective_status` is `revoked`. If that cascade cannot be resolved the endpoint answers `503 delegation_status_unavailable` rather than serving stored statuses as if they were derived ones.
- `GET /api/v1/delegations/:id` — one delegation **including** its `token`: re-surfacing the publishable artifact costs nothing, re-minting is billed. Carries the same `effective_status`/`is_valid` pair, and the same 503 on an unreadable cascade.
- `POST /api/v1/delegations/:id/revoke` — revoke (optional body `{"reason": "user_request" | "fraud" | "key_compromise" | "other"}`). Idempotent: a repeat answers `200` with the stored revocation, here and through `POST /api/v1/proofs/:id/revoke` with the `ph_dlg_…` handle.

Create and revoke return the same delegation shape the reads return, including `effective_status` and `is_valid` — the shape of a delegation does not depend on which verb produced it, so there is no second GET to make.

`:id` accepts the internal id or the `ph_dlg_…` handle. A revoked delegation appears on the revocation list, the Token Status List and the owner lookup `GET /api/v1/proofs/:id/status` (authenticated, `proofs:read`; accepts `ph_dlg_…` too). Anyone without an account checks a delegation through the public `POST /api/v1/proofs/validate` — see [Verifying a delegation](#verifying-a-delegation). Revoking one delegation touches nothing else — the domain proof and every other delegation stay valid.

### Publishing Your Delegation

A delegation token is a bearer artifact designed to be published where agents already look. The namespaced key is `holdings.proof/delegation`; its value is `{ "token": "<delegation JWT>" }`.

**MCP server card (`server.json`)** — vendor-specific data belongs in the namespaced `_meta` field:

```json
{
  "name": "io.github.you/your-mcp-server",
  "_meta": {
    "holdings.proof/delegation": { "token": "eyJhbGciOiJFUzI1NiIs..." }
  }
}
```

When publishing through the official MCP registry, nest the same entry under `_meta["io.modelcontextprotocol.registry/publisher-provided"]` — readers should check both locations.

**A2A agent card (`agent-card.json`)** — an entry in `capabilities.extensions`:

```json
{
  "url": "https://your-agent.example/api",
  "capabilities": {
    "extensions": [{
      "uri": "https://proof.holdings/delegation",
      "required": false,
      "params": { "token": "eyJhbGciOiJFUzI1NiIs..." }
    }]
  }
}
```

**What a verifier checks:**

1. The token's ES256 signature against the issuer's JWKS (cached — no call at request time).
2. `principal` against the domain it expects, and `delegate` against the artifact identity it resolved **itself** — never against what the card says about itself: a copied token names the genuine artifact, not the impostor's.
3. Freshness via the token's `status.status_list` claim and the public validation endpoint below.

A published delegation attests exactly one thing: *the controller of the `principal` domain authorized this artifact for these scopes*. It is **not** a statement that the server is safe, audited, or endorsed by proof.holdings. Our own cards use this same publication format — `mcp/server.json` for `pkg:npm/@proof-holdings/mcp-server` and [`/.well-known/agent-card.json`](https://proof.holdings/.well-known/agent-card.json) for our A2A endpoint.

**Assemble the JSON above by hand.** `@proof-holdings/delegation-publisher`, a CLI that builds this exact shape from a token you already minted and validates the claim binding, is not on npm, so any command naming it fails. Before you publish, read the card back with `verifyPublishedDelegation` from `@proof-holdings/delegation-verifier` the way a third party will — a malformed or mismatched card is otherwise a silent verification failure for every reader.

### Verifying a Delegation

Verification needs **no account and no API key**. Two public surfaces carry it:

- **`POST /api/v1/proofs/validate`** — submit the delegation token as `proof_token`. This is the
  only surface that resolves the **cascade**: a delegation is worth no more than the domain
  control proof beneath it, and a revoked domain proof does not flip the delegation's own bit in
  the Token Status List.
- **`GET /api/v1/proofs/status-list`** — the signed IETF Token Status List the token's
  `status.status_list.uri` claim points at, for cache-friendly, herd-private revocation checks.
  The list is served with a `Cache-Control: max-age` just under a minute and carries the same
  `ttl` — the promise minus the issuer's own rebuild interval — so a revocation of the delegation
  itself reaches every verifier reading this channel **within a minute**; the validation endpoint
  above answers immediately. Every slot the issuer hands out is present in the list from the next
  fetch onward — a token is never issued pointing past the published array.

```json
{
  "valid": true,
  "token_type": "delegation",
  "proof_id": "ph_dlg_0123456789abcdef0123456789abcdef",
  "delegation": {
    "id": "ph_dlg_0123456789abcdef0123456789abcdef",
    "principal": "postmarkapp.com",
    "delegate": "pkg:npm/postmark-mcp",
    "scope": ["send-email"],
    "control_proof": "ph_ctl_fedcba9876543210fedcba9876543210",
    "issued_at": "2026-07-31T00:00:00.000Z",
    "expires_at": "2027-07-31T00:00:00.000Z"
  }
}
```

An invalid answer carries `valid: false` with `reason` — `revoked`, `suspended`, `expired`,
`unknown_delegation` (no registry record: delegations are minted fail-closed, so this means
invalid, not merely unknown) or `invalid`. Passing `identifier` alongside a delegation token
returns `identifier_unverifiable`: a delegation is about an artifact, not an identifier —
compare `delegate` instead.

**Reference verifier.** [`@proof-holdings/delegation-verifier`](https://www.npmjs.com/package/@proof-holdings/delegation-verifier)
walks the whole chain — signature → `principal` → `delegate` → status — with zero runtime
dependencies and **no default trust list**: you pass the issuers you accept. It reads the
delegation out of an MCP or A2A card for you, and it requires the artifact identity you resolved
independently, because that comparison is what defeats a copied token.

```ts
import { verifyPublishedDelegation } from '@proof-holdings/delegation-verifier';

const result = await verifyPublishedDelegation(serverCard, {
  trustedIssuers: [{
    issuer: 'proof.holdings',
    jwksUri: 'https://api.proof.holdings/.well-known/jwks.json',
    statusEndpoint: 'https://api.proof.holdings/api/v1/proofs/validate',
  }],
  delegate: { type: 'purl', value: 'pkg:npm/postmark-mcp' }, // what YOU are installing
  expectedPrincipal: 'postmarkapp.com',                      // who you expect behind it
});
```

Both pins matter and they close different attacks. `delegate` stops a genuine token being
**copied** into another package's card. `expectedPrincipal` stops one being **minted by the
wrong party**: issuing binds the artifact to nothing, so any domain owner can mint a valid
delegation naming someone else's package. A result without it carries `principalChecked: false`
and means "this domain claims the artifact", not "verified".

Our own MCP server ships it as the `verify_delegation` tool, so an agent can check another
server's card before installing it.

### Looking up an artifact

When you hold a package name rather than a card, ask where its delegation would be.
`GET /api/v1/artifacts/claim?type=purl&delegate=pkg:npm/postmark-mcp` — public, no API key — reads
the places a delegation can be published (the npm manifest, the MCP registry entry the package
names, an endpoint's own and host cards, and the `_mcp` DNS pointer and `.well-known` cards of up to
5 domains that hold a delegation for the artifact here). The npm manifest is read, but the npm CLI
removes every top-level field that begins with `_` — `_meta` included — when it publishes, so an npm
package's delegation is found in its MCP registry entry or on its domain's own cards. The answer
lists each place and what it carried:

```json
{
  "delegate": "pkg:npm/postmark-mcp",
  "delegate_type": "purl",
  "artifact_path": "/npm/postmark-mcp",
  "claim": "no_claim_found",
  "checked_at": "2026-10-08T12:00:00.000Z",
  "locations": [
    {
      "id": "npm_metadata",
      "location": "The npm registry's manifest for postmark-mcp (latest published version)",
      "url": "https://registry.npmjs.org/postmark-mcp/latest",
      "result": "nothing_published",
      "reason": null,
      "detail": "Read; it carries no delegation in any of the sanctioned slots.",
      "checked_at": "2026-10-08T12:00:00.000Z"
    }
  ],
  "delegations": [],
  "listing": null
}
```

`claim` is `claimed` only when a token read from one of those places is signed by this issuer,
names exactly this artifact and is in force; each such token is returned in `delegations[]`, with the
domain that signed it, so you can verify it yourself as above. A delegation that was issued but never
published is not a claim, and the answer does not reveal it. A place that could not be read is
`unchecked` with a `reason` — never reported as empty. Answers are reused for up to an hour (five
minutes when a place could not be read); `503 artifact_claim_busy` carries `Retry-After`.

The same answer is published three more ways:

- **A page per artifact** — `https://proof.holdings/npm/<name>`, `/pypi/<name>` and `/endpoint/<host><path>` — with its markdown at `/md/npm/<name>.md` and its JSON at `/api/content/artifact?type=purl&delegate=pkg:npm/<name>` on the same site.
- **A README badge**, live, so it changes when the delegation does: `[![Proof of Delegation](https://proof.holdings/badge/npm/<name>)](https://proof.holdings/npm/<name>)`.
- **A terminal check**, `npx @proof-holdings/proof-check pkg:npm/<name>`, which asks this endpoint where to look and verifies every token it is given locally, against the identifier you typed. Exit codes: `0` delegated, `1` no delegation found, `2` could not check. The package is not on npm until its first release.

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
  "by_status": { "pending": 3, "delivered": 142, "failed": 5 },
  "delivery_rate": "94.7%",
  "recent_failures_24h": 1
}
```

### Retry Failed Delivery

`POST /api/v1/webhook-deliveries/:id/retry`

Queue a fresh ladder of attempts for a delivery that has not been delivered: one with status `failed`, or a `pending` one no worker is holding. Attempt numbering continues after the attempts already spent, so a retried attempt never reuses the `X-Webhook-Id` of an attempt its `attempts` count already includes. Answers `400 already_delivered` for a delivered one and `503 webhook_enqueue_failed` when the retry could not be queued.

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

These describe a **verification**. A **proof** has its own, shorter vocabulary — `active`, `suspended`, `revoked`, `expired` — which is what `GET /api/v1/proofs/:id/status`, the suspend/unsuspend pair and the delegation surfaces report. A verification that never produced a live proof (`pending`, `failed`, `cancelled`) surfaces its own status there.

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
| **Message Templates** | `GET /api/v1/templates`, `PUT /api/v1/templates/:channel/:messageType` | [Message Templates](/docs/templates) |
