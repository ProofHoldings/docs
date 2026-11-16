---
title: API Reference
lastUpdated: "2026-10-29"
status: current
---

# API Reference

This page covers the core verification, proof, webhook and delegation endpoints. Every endpoint is in the OpenAPI document: [`/api/openapi.json`](https://api.proof.holdings/api/openapi.json).

## Authentication

Base URL: `https://api.proof.holdings`

Authentication: Bearer token via `Authorization` header

```
Authorization: Bearer pk_live_your_api_key_here
```

API keys are prefixed with `pk_live_` for production and `pk_test_` for testing.

---

## Test Mode (Sandbox)

Use test mode to build and test your integration without sending real messages, consuming quota, or incurring charges. Test mode is activated automatically when you authenticate with a **test API key** (`pk_test_*`).

### How It Works

| Aspect | Production (`pk_live_*`) | Test Mode (`pk_test_*`) |
|--------|--------------------------|-------------------------|
| Channel delivery | Real messages sent | Skipped — no real messages |
| Quota usage | Counted toward monthly limit | Not counted |
| Billing | Charged per proof | Free |
| Webhooks | Delivered normally | Delivered; verification webhooks carry `"test": true`, and so does a verification-request webhook once a test verification is behind it |
| Data isolation | Only sees production verifications | Only sees test verifications |
| Proof tokens | Valid, signed JWT | Signed JWT of the same format, under keys of a separate test tier once that tier is in place (see below); `POST /proofs/validate` answers `valid: false` |

`test-verify` completes a verification with no real check, so a test-mode token proves nothing about the identifier it names. `POST /proofs/validate` answers `valid: false` with `reason: "test_environment"` for a proof or delegation issued in test mode. `GET /proofs/delegation-token/{handle}` does not serve a test delegation. Revoking a test-mode proof revokes that proof and the delegations anchored on it, not the production records of the same domain.

Once the test tier is in place in an environment, test mode signs with keys of its own: a tier whose public keys are published only at `/.well-known/jwks-test.json`, never at `/.well-known/jwks.json`. Until then test-mode tokens are signed by production keys, as described in the next paragraph. Once it is in place, a verifier that reads the production key set — the SDKs' offline verification, the reference delegation verifier, or your own JWT library given that key set — does not know a test-mode token's `kid` and never answers valid for it, whether or not it checks revocation: the SDKs and the reference verifier answer `unknown_key`, or, within 30 seconds after an earlier unknown `kid`, a temporary "could not check" (`status_unavailable` in the SDKs, `jwks_unavailable` in the reference verifier). A test-mode token's `status.status_list.uri` names the test status list, `GET /proofs/status-list/test`, which carries each test token's real status (and `3`, unavailable, for every production slot).

A test-mode token whose `kid` names a production key, as every test token issued before the test tier was in place does, is not refused by its signature. For such a token the Token Status List marks its slot unavailable from the first rebuild after the token was issued, so the SDKs' offline verification, which reads the status list by default for a token that carries a slot, does not answer valid for it from then on. A list built before the token was issued still reads that slot as valid, and the served list and a verifier's cached copy can each be up to about a minute old, so for up to about two minutes after issuance an offline check can still answer valid. A signature-only check cannot tell such a token from a production one, and neither can an offline check of one that carries no status-list slot: verify online.

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
  "proof_expires_at": "2027-02-13T12:25:00Z",
  "test_mode": true
}
```

### Webhook Behavior

Verification webhooks sent in test mode carry `"test": true` in the payload. A verification-request webhook carries it when at least one of the request's verifications ran in test mode, so a test request that expires before anyone starts a verification arrives without it:

```json
{
  "event": "verification_request.completed",
  "test": true,
  "request_id": "507f1f77bcf86cd799439011",
  "status": "completed",
  "proofs": [...]
}
```

### Environment Isolation

Test and production verifications are kept apart. A `pk_test_` key can only see test verifications, and a `pk_live_` key can only see production verifications.

---

## Your First Verification

With a test key, a verification completes without sending a real message:

```bash
# 1. Create verification with test key (no real message sent)
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_test_..." \
  -H "Content-Type: application/json" \
  -d '{"type":"phone","channel":"whatsapp","identifier":"+37069199199"}'

# 2. Auto-verify (skip waiting for user action)
curl -X POST https://api.proof.holdings/api/v1/verifications/{id}/test-verify \
  -H "Authorization: Bearer pk_test_..."

# 3. Validate the proof token as your integration validates live ones: a test-mode
#    token answers {"valid": false, "reason": "test_environment"}
```

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
| `GET` | `/api/v1/verifications/users` | List verified users |
| `GET` | `/api/v1/verifications/users/:externalUserId` | Get verified user details |
| `POST` | `/api/v1/verifications/domain` | Start domain verification for a customer |
| `POST` | `/api/v1/verifications/domain/:id/check` | Check domain verification for a customer |
| `POST` | `/api/v1/proofs/validate` | Validate proof token |
| `GET` | `/api/v1/proofs/:id/status` | Check proof status |
| `POST` | `/api/v1/proofs/:id/revoke` | Revoke a proof |
| `GET` | `/api/v1/proofs/revoked` | Get revocation list |
| `POST` | `/api/v1/delegations` | Create a delegation (mint token) |
| `GET` | `/api/v1/delegations` | List delegations |
| `GET` | `/api/v1/delegations/:id` | Get a delegation incl. its token |
| `POST` | `/api/v1/delegations/:id/revoke` | Revoke a delegation |
| `POST` | `/api/v1/delegations/:id/renew` | Renew a delegation in place (same handle, later expiry) |
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
| `GET` | `/api/v1/webhook-endpoints` | List webhook endpoints |
| `POST` | `/api/v1/webhook-endpoints` | Create webhook endpoint |
| `GET` | `/api/v1/webhook-endpoints/:id` | Get webhook endpoint |
| `PATCH` | `/api/v1/webhook-endpoints/:id` | Update webhook endpoint |
| `DELETE` | `/api/v1/webhook-endpoints/:id` | Delete webhook endpoint |
| `POST` | `/api/v1/webhook-endpoints/:id/rotate-secret` | Rotate an endpoint's signing secret |
| `GET` | `/api/v1/profiles/profiles/:profileId/callback-secret` | Read a sub-account's callback signing secret (dashboard session) |
| `POST` | `/api/v1/profiles/profiles/:profileId/callback-secret/rotate` | Rotate a sub-account's callback signing secret |
| `GET` | `/.well-known/jwks.json` | Public keys (ES256) |
| `GET` | `/.well-known/jwks-test.json` | Public keys of the test signing tier (test mode only) |
| `GET` | `/health` | Service status |

---

## Create Verification

`POST /api/v1/verifications`

Create a new verification challenge.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `telegram_bot` (`social`, `wallet` and `account` are coming soon: the API answers `400 channel_not_available`) |
| `channel` | string | Yes | Channel for verification (see table below) |
| `identifier` | string | Yes | The asset to verify |
| `client_metadata` | object | No | Custom metadata for your reference |
| `external_user_id` | string | No | Your own id for the user, used to group their verifications under Verified Users |
| `dns_provider` | object | For `auto` | DNS provider credentials; required when `channel` is `auto` |
| `bot_token` | string | For `telegram_bot_token` | The bot's token; required when `channel` is `telegram_bot_token` |
| `email_prefix` | string | No | For `type` `domain` with `channel` `email`: the mailbox the code goes to — `admin` (default), `administrator`, `webmaster`, `hostmaster` or `postmaster` |
| `proof_expiry_days` | integer | No | How long the proof lasts, 1–365 days; overrides your account's default |
| `profile_id` | string | No | The sub-account whose branding the messages carry; a sub-account key may name only its own |

### Type/Channel Compatibility

| Type | Valid Channels |
|------|----------------|
| `phone` | `whatsapp`, `telegram`, `sms` |
| `email` | `email` |
| `domain` | `dns`, `http`, `email`, `auto` |
| `social` | `github`, `google`, `facebook`, `x`, `linkedin`, `instagram`, `youtube`, `tiktok` (coming soon) |
| `wallet` | `ethereum`, `solana`, `bitcoin` (coming soon) |
| `account` | `coinbase`, `kraken` (coming soon) |
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
    "send_to": "+15557766633",
    "send_to_formatted": "+1 (555) 776-6633",
    "instruction": "Send 'X7K2M9' to +1 (555) 776-6633 on whatsapp",
    "deep_link": "https://wa.me/15557766633?text=Verify%20X7K2M9"
  },
  "created_at": "2026-02-04T10:50:00Z"
}
```

The person sends the code from the phone being verified to our verification number, `send_to`; `deep_link` opens WhatsApp on that number with the message filled in. For an email, or a domain checked by email, the response does not contain the code: we mail it to the address, and the person enters it through Submit Challenge Code below.

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
    "expires_at": "2027-02-04T10:51:30Z"
  }
}
```

---

## Submit Challenge Code

`POST /api/v1/verifications/:id/submit`

Submit the code we mailed for a verification with `type: email`, or `type: domain` with `channel: email`. Any other verification answers `400 unsupported_for_type`: a phone completes when the person messages us, and a domain over `dns`, `http` or `auto` completes through `POST /api/v1/verifications/:id/verify`.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | The code from the email |

```text
POST /api/v1/verifications/:id/submit
{ "code": "X7K2M9" }
```

### Response

When the code is right:

```json
{
  "id": "507f1f77bcf86cd799439011",
  "status": "verified",
  "attempts": 1,
  "max_attempts": 3,
  "verified_at": "2026-02-04T10:51:30Z",
  "proof": {
    "token": "eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_at": "2027-02-04T10:51:30Z"
  }
}
```

When it is wrong, `status` stays `pending` and the response adds `message` and `attempts_remaining`. When the last attempt is spent, `status` is `failed` and the response adds `message`.

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

The stream needs your API key in the `Authorization` header, so open it from your server, with an SSE client that can send headers. A browser's built-in `EventSource` cannot set headers: the key never reaches the API and the stream answers `401`.

### Usage

```bash
curl -N https://api.proof.holdings/api/v1/verifications/{id}/events \
  -H "Authorization: Bearer pk_live_..."
```

```text
id: 0
event: connected
data: {"verificationId":"507f1f77bcf86cd799439011"}

id: 1
event: status_changed
data: {"status":"verified","timestamp":"2026-02-04T10:51:30.000Z"}
```

Close the stream once `status` is no longer `pending`.

### Events

| Event | Description |
|-------|-------------|
| `connected` | Initial connection established |
| `status_changed` | Verification status updated |

A 30-second heartbeat keeps the connection alive through Cloudflare. Your account can hold up to 10 SSE connections at once.

---

## Verified users

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
  "pagination": { "page": 1, "limit": 20, "total": 42, "total_pages": 3 }
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
      "proof_expires_at": "2027-02-13T12:00:00Z",
      "created_at": "2026-02-13T11:50:00Z"
    }
  ]
}
```

---

## Domain verification for your customers

Verify your own domain with `POST /verifications` (type `domain`). Verify a domain for one of your customers with `POST /verifications/domain`, which keeps one record per customer domain.

### Start Domain Verification

`POST /api/v1/verifications/domain`

Start a domain verification flow. Returns DNS and HTTP instructions for the domain owner.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | Yes | Domain to verify (e.g., `team.management`) |
| `verification_method` | string | No | `manual_dns` (default) or `http_file` |

`customer_id` is accepted and ignored.

#### Response

```json
{
  "id": "507f1f77bcf86cd799439011",
  "domain": "team.management",
  "status": "pending",
  "verification_method": "manual_dns",
  "dns_record": {
    "type": "TXT",
    "name": "_proof.team.management",
    "value": "proof-verification=<token>"
  },
  "http_file": {
    "path": "https://team.management/.well-known/proof-holdings-verification.txt",
    "content": "proof-verification=<token>"
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
  "domain": "team.management",
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
  "expires_at": "2027-02-13T12:00:00Z",
  "is_expired": false
}
```

---

## Sender name

Every message our bots send for you names one of your sub-accounts: its business name, else its display name. If that sub-account has no name, the message carries Proof Holdings copy. The name is never taken from the request.

- A **session** (`POST /api/v1/sessions`) names the sub-account it is for: `profile_id`, else the API key's sub-account, else your primary.
- A **consent request** (`POST /api/v1/authorizations`) names the approval setup's sub-account, the same name the approval page shows. The response's `business_name` is the name used.

`business_name` on `POST /api/v1/authorizations` is deprecated and ignored. It is still accepted so existing integrations keep working, and a request that sends it gets a `Deprecation` response header. To send for several businesses, create one sub-account per business and use its API key.

---

## Sessions (Phone-First Flow)

Sessions provide a phone-first verification flow where the user doesn't need to provide their phone number upfront. Instead, they message our Telegram or WhatsApp bot, or text a short code by SMS to one of our numbers, and the number is captured from that message.

### Create Session

`POST /api/v1/sessions`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channel` | string | Yes | `telegram`, `whatsapp` or `sms` |
| `client_metadata` | object | No | Custom metadata for your reference |
| `profile_id` | string | No | The sub-account the session is for. Omitted: the API key's sub-account, else your primary |
| `context.action` | string | No | Picks the bot's wording: `login`, `2fa`, `recovery` or `verification` (default). Any other value answers `400 validation_error` |

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
  "id": "sess_8f14e45fceea167a5a36dedd4bea2543",
  "channel": "telegram",
  "status": "pending",
  "deep_link": "https://t.me/proof_holdings_bot?start=sess_8f14e45fceea167a5a36dedd4bea2543",
  "qr_code": "data:image/png;base64,...",
  "qr_text": "...",
  "expires_at": "2026-02-04T11:00:00Z",
  "instructions": "Open Telegram and tap the link to share your phone number"
}
```

`qr_code` is a PNG image of `deep_link` as a data URL, ready for an `<img src>`; `qr_text` is the same QR code drawn in text characters, for a terminal.

With `"channel": "sms"` the person texts a short code to one of our numbers, the same way SMS sign-in works. `deep_link` opens their messaging app on our default number with the message pre-filled, and `sms_dids` lists every number the code can be sent to, one per region, each with its own QR code. The person gets no reply SMS: read the result with `GET /api/v1/sessions/:id`. An SMS proof carries `confidence: "standard"`.

```json
{
  "id": "sess_8f14e45fceea167a5a36dedd4bea2543",
  "channel": "sms",
  "status": "pending",
  "deep_link": "sms:+19294568714?body=Verify%20ABC234",
  "qr_code": "data:image/png;base64,...",
  "qr_text": "...",
  "expires_at": "2026-02-04T11:00:00Z",
  "instructions": "Send the verification message by SMS to one of the numbers shown",
  "sms_code": "ABC234",
  "sms_message": "Verify ABC234",
  "sms_dids": {
    "americas": { "did": "+19294568714", "label": "Americas", "flag": "🇺🇸", "qr_code": "data:image/png;base64,...", "qr_text": "..." }
  }
}
```

### Get Session

`GET /api/v1/sessions/:id`

Poll for session status. When verified, includes the phone number and proof token (for SMS, with the `confidence` it was issued with). About a minute after an SMS session completes, a delayed spoof check can still act on its proof if the sender was not a real mobile line. That is the proof's own state, not the session's. A revocation is reported by `POST /api/v1/proofs/validate` and the `proof.revoked` webhook; a weaker signal instead lowers the confidence to `degraded`, which shows on `GET /api/v1/verifications/{verification_id}` (the session returns `verification_id`).

#### Response (Verified)

```json
{
  "id": "sess_8f14e45fceea167a5a36dedd4bea2543",
  "channel": "telegram",
  "status": "verified",
  "phone_number": "+37069199199",
  "verified_at": "2026-02-04T10:51:30Z",
  "verification_id": "507f1f77bcf86cd799439011",
  "proof": {
    "token": "eyJhbGciOiJFUzI1NiIs...",
    "expires_at": "2027-02-04T10:51:30Z"
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
| `action_type` | string | No | `verification`, `2fa`, `login`, `custom`, `confirmation`, `authorization` |
| `action_context` | object | No | `{ title, description }` for display |
| `redirect_url` | string | No | Where to redirect after completion |
| `callback_url` | string | No | Webhook URL for completion notification. When set, this request's events go here instead of to your webhook endpoints, signed with the sub-account's callback signing secret |
| `expires_in` | number | No | Seconds until expiry (default 604800 = 7 days, max 2592000 = 30 days) |
| `partial_ok` | boolean | No | Accept partial completion |
| `outcome` | string | No | `relying_party`: the chat reply after a proof waits for your decision (see [Your Decision After the Proof](#your-decision-after-the-proof)) |
| `outcome_timeout_seconds` | number | No | With `outcome`: the longest the reply waits, 3–15 seconds (default 10) |

Each asset in the `assets` array:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `telegram_bot` (`social` and `wallet` are coming soon, not available yet) |
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
      { "type": "domain", "identifier": "team.management" },
      { "type": "email", "identifier": "admin@team.management" }
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
    { "type": "domain", "identifier": "team.management", "status": "pending" },
    { "type": "email", "identifier": "admin@team.management", "status": "pending" }
  ],
  "verification_url": "https://proof.holdings/verify/507f1f77bcf86cd799439011",
  "created_at": "2026-02-04T10:50:00Z",
  "expires_at": "2026-02-11T10:50:00Z"
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

### Your Decision After the Proof

Some apps decide after the proof whether they accept the verified number: an allow list, a paid plan, any rule of their own. By default our bot replies in the chat as soon as the person proves the number ("Phone verified for …"), before your app has decided. If you then refuse, the chat says one thing and your page another.

Create the request with `"outcome": "relying_party"` and the bot holds that reply until you decide:

1. The person proves the number. The asset verifies and its `outcome` reads `{"status": "awaiting", "deadline_at": …}`. Your completion webhook and `GET /api/v1/verification-requests/:id` report this at once, as for any request.
2. Your app validates the proof, runs its own check and reports the result: `POST /api/v1/verification-requests/:id/assets/:assetIndex/outcome` with `{"accepted": true}`, or `{"accepted": false, "reason": "…"}`. The key needs `verifications:write`.
3. The bot sends the success message, or your `verification_refused` template with your `reason`. Our verification page shows the same.

```bash
curl -X POST https://api.proof.holdings/api/v1/verification-requests/507f1f77bcf86cd799439011/assets/0/outcome \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{ "accepted": false, "reason": "This number is not on the team.management allow list." }'
```

The window, `outcome_timeout_seconds` (3–15, default 10), is a maximum, not a delay: the reply goes out the moment you answer. While it waits, the chat shows the bot typing; on WhatsApp this also marks the person's message as read. If you do not answer in time, the person gets a neutral "✅ Verified. Return to <your business name> to continue." (platform copy, using your sub-account's name), and the asset's outcome reads `timed_out`. That text is never the success message.

The `reason` is plain text, at most 200 characters, shown to the person as you wrote it. The refusal message is the `verification_refused` template, editable per sub-account for Telegram and WhatsApp in the dashboard, on the Public page's Templates tab. Its variables are `{{business_name}}` and `{{reason}}`; the default reads "Your number was verified, but *<your business name>* has not accepted it." followed by your reason. SMS sends no chat reply, so an SMS-verified asset shows your decision on our verification page only.

While the outcome is awaited, and after a refusal, our verification page shows "Checking with <your business name>…" or the refusal and its reason, and does not redirect to your `redirect_url`. An acceptance or a timeout continues as before.

| `outcome.status` | Meaning |
|------------------|---------|
| `awaiting` | The asset verified; the reply is held until you answer or `deadline_at` passes |
| `accepted` | You accepted; the success message was sent |
| `refused` | You refused; the refusal was sent, with your `reason` |
| `timed_out` | No answer by `deadline_at`; the neutral text was sent |

The outcome call answers `409` with `outcome_not_awaited` (the request was not created with `outcome: "relying_party"`, or the asset has not verified yet), `outcome_already_set` (you answered before), or `outcome_window_closed` (the window passed first). If the chat cannot be reached when its message is due, we keep trying for up to a minute; after that the chat message is dropped, and our page and your read still show the outcome. A refused verification is still a completed proof and is billed like any other.

### How the Verified Identity Reaches You

The proof token never carries the number itself. It carries `identifier_hash`: the hex SHA-256 of the normalized identifier, unsalted (a phone in E.164, such as `+37255555555`). To check a token against a number you hold, normalize it and compare the hashes, or pass it as `identifier` to [Validate Proof](#validate-proof).

The number itself is on your authenticated read: `GET /api/v1/verification-requests/:id` returns each verified asset's `verified_identifier` unmasked. The public reads behind our verification page mask it (`+372•••••555`).

### Verification Request Statuses

| Status | Description |
|--------|-------------|
| `pending` | Waiting for verifications to complete |
| `partial` | Some assets verified, others pending |
| `completed` | Every required asset verified, or, with `partial_ok`, at least one required asset (any asset when none is required) |
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
    "expires_at": "2027-02-04T10:51:30Z"
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

Verify a token you fetched yourself, server to server (`GET /api/v1/verification-requests/:id/proofs`), never one a browser hands you: a valid signature proves we issued the token, not that it is about your request. A token from another account's request verifies too. Check that its `tenant_id` is your account's id, and that its `sub` is the request (or verification) you created.

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

The `revoked` array is capped and sorted newest first. When `has_more` is `true` the array was cut short, so an older revocation may be missing from it: check those proofs online with `POST /api/v1/proofs/validate`, or read the Token Status List the token's `status` claim points at.

```javascript
import jwt from 'jsonwebtoken';
import { createPublicKey } from 'node:crypto';

const jwks = await (await fetch('https://api.proof.holdings/.well-known/jwks.json')).json();
const list = await (await fetch('https://api.proof.holdings/api/v1/proofs/revoked')).json();

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

// Check not revoked: a revoked entry names the token's `sub` (`proof_id`) or its public handle
// (`public_proof_id`, the token's `proof_id` claim)
const isRevoked = list.revoked.some(
  r => (r.proof_id !== null && r.proof_id === payload.sub) ||
       (r.public_proof_id !== null && r.public_proof_id === payload.proof_id)
);

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
| `reason` | string | No | `user_request` (default), `fraud`, `key_compromise`, `asset_transferred`, `other`. A delegation proof (`ph_dlg_…`) refuses `asset_transferred`. |

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

**Who can call it.** An API key with the `proofs:*` scope (the routes ask for `proofs:write`; a key is created with `proofs:read` or `proofs:*`, and only `proofs:*` grants it), or a signed-in dashboard session — the same dual-auth the delegation endpoints use. The scopes bind API keys only; a session owns the account outright. This applies to `/status`, `/revoke`, `/suspend` and `/unsuspend` alike.

**A second revoke means different things on the two resources, deliberately.**

| Resource | Repeat revoke | Why |
|----------|---------------|-----|
| Proof (`ph_ctl_…`, verification id) | `400 already_revoked` with the FIRST revocation's `revoked_at` | The code is published and read by integrators; changing it would break callers that branch on it |
| Delegation (`ph_dlg_…`) | `200` with the stored revocation, on this endpoint AND on `POST /api/v1/delegations/:id/revoke` | Idempotency is what the delegation contract has always advertised, in these docs and in all four SDKs |

Either way the answer never depends on WHICH address you used. A repeat call reports the revocation that actually happened: the original timestamp and the original reason, never this call's.

**Scopes on a delegation handle.** Any lifecycle act on a `ph_dlg_…` — revoke, suspend or unsuspend — additionally requires `delegations:write` on top of the route's own `proofs:write`, so the wider group is not a way around the narrower one. While a pause holds, a third party sees what it sees after a revocation: the token stops verifying and the status list says so. Acting on a domain control proof (`ph_ctl_…`) needs `proofs:write`, and `delegations:write` as well while delegations issued from it are live.

---

## Delegations

Proof of Delegation: a control-proven domain authorizes a typed artifact — a package (`purl`) or an https endpoint (`url`). Each create mints an independently revocable ES256 delegation token, signed with the same key and published through the same JWKS as proof tokens. Requires `delegations:*` API-key scopes; each mint is one billable proof.

### Create Delegation

`POST /api/v1/delegations`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `control_proof` | string | Yes | Public handle of YOUR verified domain proof (`ph_ctl_…`). The delegation's `principal` is the domain that proof covers. |
| `delegate` | object | Yes | The authorized artifact: `{ "type": "purl" \| "url", "value": "pkg:npm/your-mcp-server" \| "https://your-agent.example/api" }`. Versions, qualifiers, query strings and fragments are rejected — a delegation names the artifact, not a release. |
| `scope` | string[] | No | Deprecated. Accepted from SDKs released before delegation schema 2.0 and ignored: a delegation carries no scope, and responses return it as an empty list for delegations issued since. Removed in the next major version. |
| `expires_in` | number | No | Token lifetime in seconds. A value longer than the control proof's remaining lifetime is refused with `400 delegation_lifetime_exceeds_control_proof` — a delegation never outlives the domain proof under it. Omitted: the default proof lifetime, or the control proof's remaining lifetime if that is shorter. |

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
- `POST /api/v1/delegations/:id/renew` — renew in place (empty body). Signs a new token for the same handle with a later `exp` — a full delegation lifetime from now, capped at the expiry of the domain's verification — and keeps every other claim, `jti` included, adding `renewal_url` (where the current token is served) to a token issued before that claim existed. When that verification is due, or its end is what caps the delegation, it is checked again first, from the published record, and renewed with it. An already-expired delegation is renewed only on control shown now: a domain not on the Domains page answers `409 domain_recheck_required`. In test mode the domain is not checked again: a test-mode delegation renews only within its test control proof's lifetime, and past it needs a new delegation from a new test verification. Answers `200` with the delegation including its new `token`. The old token keeps verifying until its own `exp`, and `GET /api/v1/proofs/delegation-token/:handle` serves the new one. Billed like a create, behind the same balance check (`402`). Refusals are `409`: `delegation_revoked`, `delegation_suspended`, `delegation_not_renewable`, `unsupported_version` (schema 1.x), `control_proof_not_active`, `control_proof_revoked`, `control_proof_suspended`, `control_proof_replaced` (the domain was verified again under a new handle — issue a new delegation from it), `domain_evidence_missing` (the record is no longer published — add it again, then renew), `delegation_not_extendable` (the renewal would add less than a day), `renewal_in_progress` (another renewal of it is running — retry in a moment) and `delegation_changed` (a concurrent change — retry). `503 domain_renewal_unavailable` when the domain could not be checked. Needs `delegations:write`; a sub-account key gets `403 account_key_required`.

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

A published delegation attests exactly one thing: *the controller of the `principal` domain authorized this artifact*. It is **not** a statement that the server is safe, audited, or endorsed by Proof Holdings. Our own cards use this same publication format — `mcp/server.json` for `pkg:npm/@proof-holdings/mcp-server` and [`/.well-known/agent-card.json`](https://proof.holdings/.well-known/agent-card.json) for our A2A endpoint.

**Assemble the JSON above by hand.** `@proof-holdings/delegation-publisher`, a CLI that builds this exact shape from a token you already minted and validates the claim binding, is not on npm, so any command naming it fails. Before you publish, read the card back with `verifyPublishedDelegation` from `@proof-holdings/delegation-verifier` the way a third party will — a malformed or mismatched card is otherwise a silent verification failure for every reader.

### Verifying a Delegation

Verification needs **no account and no API key**. Two public surfaces carry it:

- **`POST /api/v1/proofs/validate`** — submit the delegation token as `proof_token`. This is the
  only surface that resolves the **cascade**: a delegation is worth no more than the domain
  control proof beneath it, and a revoked domain proof does not flip the delegation's own bit in
  the Token Status List.
- **`GET /api/v1/proofs/status-list`** — the signed IETF Token Status List the token's
  `status.status_list.uri` claim points at. Every verifier downloads the same list, so checking a
  token this way does not tell us which token you are checking.
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
the places a delegation can be published: the package's own file in its latest release (an npm
package's `package.json`, a Composer package's `composer.json` `extra` on Packagist, a Go module's
`proof-delegation.json` on the Go module proxy), the MCP registry entry the package names, an
endpoint's own and host cards, and the `_mcp` DNS pointer and `.well-known` cards of up to 5 domains
that hold a delegation for the artifact here. An npm `package.json` is read from the published
tarball, because the npm CLI removes every top-level field that begins with `_` — `_meta` included —
from the registry's own manifest. The MCP registry lists no Go modules or Composer packages, so for
those its row is `not_applicable`. The answer lists each place and what it carried:

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
      "location": "The package.json of postmark-mcp (latest published version)",
      "url": "https://registry.npmjs.org/postmark-mcp/latest",
      "result": "nothing_published",
      "reason": null,
      "detail": "Read; no delegation is published there.",
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

- **A page per artifact** — `https://proof.holdings/npm/<name>`, `/pypi/<name>`, `/golang/<module path>`, `/composer/<vendor>/<name>` and `/endpoint/<host><path>` — with its markdown at `/md/npm/<name>.md` and its JSON at `/api/content/artifact?type=purl&delegate=pkg:npm/<name>` on the same site.
- **A README badge**, live, so it changes when the delegation does: `[![Proof of Delegation](https://proof.holdings/badge/npm/<name>)](https://proof.holdings/npm/<name>)`.
- **A terminal check**, `npx @proof-holdings/proof-check pkg:npm/<name>`, which asks this endpoint where to look and verifies every token it is given locally, against the identifier you typed. Exit codes: `0` delegated, `1` no delegation found, `2` could not check.

---

## Webhook Endpoints

An endpoint belongs to the account and receives the events of the sub-accounts it serves: `scope: "all"` (every sub-account, present and future) or `scope: "selected"` with `profile_ids`. Every event body carries `profile_id`, the sub-account it belongs to — `null` for an account-wide event (account deletion, proof events for delegations and domains, and `delegation.reissued`), which only `all` endpoints receive. An account may have up to 20 endpoints. API keys need `webhooks:read` / `webhooks:write`; a dashboard session confirms writes with a second factor. The events are listed under Webhook Event Types below.

### Create Webhook Endpoint

`POST /api/v1/webhook-endpoints`

```json
{
  "url": "https://team.management/webhooks/proof",
  "events": ["verification_request.*", "proof.*"],
  "scope": "selected",
  "profile_ids": ["65f1a2b3c4d5e6f7a8b9c0d1"],
  "description": "Billing service"
}
```

`events` takes exact event types, `<family>.*` or `*`; an empty list means every event. The response carries the endpoint's signing `secret` — the only read that does, apart from `POST /api/v1/webhook-endpoints/:id/rotate-secret`; every other read returns `secret_set`.

`GET`, `PATCH` and `DELETE /api/v1/webhook-endpoints/:id` read, change (`url`, `events`, `scope`, `profile_ids`, `status: "active" | "disabled"`, `description`) and delete one. Deliveries already queued keep the URL and secret they were queued with.

### Request Events and callback_url

A verification request with a `callback_url` sends its events there only, signed with its sub-account's callback signing secret (`GET /api/v1/profiles/profiles/:profileId/callback-secret`, rotate with `POST …/rotate`). A request without one sends its `verification_request.*` events to its sub-account's matching endpoints. Nothing is delivered twice.

### The Settings Webhook Block (deprecated)

The `webhook` block of `GET`/`PATCH /api/v1/me/settings` still works as your account's first endpoint — the one an account's settings URL became — and every response carrying it has `Deprecation` and `Sunset` headers. After the sunset date it will be refused with `400 webhook_moved`; use `/api/v1/webhook-endpoints`.

## Webhook Deliveries

Monitor and manage webhook deliveries for your account: one row per event per endpoint or callback URL.

An API key reads and retries the deliveries of its own environment: a test key the test ones, a live key the live ones. A dashboard session has no key, so on these four operations it may name the environment with the query parameter `environment` (`production` or `test`; absent, `production`; any other value answers `400 validation_error`). With an API key the parameter is not read.

### List Webhook Deliveries

`GET /api/v1/webhook-deliveries`

Query parameters: `page`, `limit` (max 100), `status` (`pending`, `delivered`, `failed`), `event_type`, `verification_request_id`, `endpoint_id`, `profile_id`

#### Response

```json
{
  "data": [
    {
      "id": "507f1f77bcf86cd799439011",
      "verification_request_id": "507f1f77bcf86cd799439012",
      "endpoint_id": "507f1f77bcf86cd799439013",
      "profile_id": "65f1a2b3c4d5e6f7a8b9c0d1",
      "url": "https://team.management/webhooks/proof",
      "event_type": "verification_request.completed",
      "status": "delivered",
      "attempts": 1,
      "max_attempts": 6,
      "response_status": 200,
      "error_message": null,
      "last_attempt_at": "2026-02-13T12:00:01Z",
      "next_retry_at": null,
      "delivered_at": "2026-02-13T12:00:01Z",
      "created_at": "2026-02-13T12:00:00Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 1, "total_pages": 1 }
}
```

### Get Webhook Delivery

`GET /api/v1/webhook-deliveries/:id`

Returns full delivery details including the stored payload and the response body. The stored payload holds every `proof_token` as `null`; the body POSTed to your endpoint carries the token, read from its record when each attempt is sent.

Every attempt of one delivery (one `X-Proof-Delivery` value) sends the token the delivery was queued with, or `proof_token: null`: a retried delivery carries the same `proof_token` as the first attempt, or `proof_token: null`, never a different token. When a token is sent as `null`, `token_omitted_reason` says why:

| Value | Meaning |
|-------|---------|
| `source_gone` | The record holding the token no longer exists, or (for a delivery stored before tokens were left out of history) could not be identified. |
| `token_absent` | The record holds no token, or held none when the delivery was queued. |
| `token_changed` | The record's token is no longer the one the delivery was queued with (re-issued or changed since). The delivery does not carry the new token. |

`token_omitted_reason` is `null` when every token was sent.

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

```json
{
  "success": true,
  "already_queued": false,
  "message": "Webhook delivery queued for retry",
  "delivery_id": "507f1f77bcf86cd799439011",
  "next_retry_at": "2026-02-13T12:05:00Z"
}
```

`already_queued` is `true` when a worker already holds an attempt for the delivery: the call queued nothing, `next_retry_at` is the attempt already scheduled, and `message` reads "Webhook delivery is already queued for retry". It is `false` when the call queued a fresh ladder. The field is present on servers from this release; where it is absent, treat it as `false`.

### Webhook Event Types

| Event | Description |
|-------|-------------|
| `verification_request.completed` | The request completed: every required asset verified, or, with `partial_ok`, at least one required asset (any asset when none is required) |
| `verification_request.partial` | Declared and accepted as a subscription filter, but not sent: a request with `partial_ok` completes with `verification_request.completed` |
| `verification_request.expired` | Request expired before completion |
| `verification.completed` | A direct verification (`POST /verifications`) or a phone-first session completed with a proof — once per verification id; a multi-channel group fires it once, for the winning channel. Not sent for the verifications behind a verification request's assets: the request reports those proofs in its own event to its `callback_url`, when it has one (`verification_request.*`, or `confirmation.approved` for a request with `action_type: confirmation`). A request without a `callback_url` sends its `verification_request.*` events to its sub-account's webhook endpoints instead. |
| `confirmation.approved` | An approval request was approved |
| `confirmation.denied` | An approval request was denied, or timed out (a timeout noticed long after it happened closes without this event) |
| `authorization.approved` | A person allowed you to send them approval requests |
| `authorization.denied` | A person refused that consent |
| `proof.revoked` | A delegation was revoked, or a phone proof was revoked by the SMS sender check |
| `proof.suspended` | A delegation was suspended |
| `proof.unsuspended` | A suspended delegation was restored |
| `account.deletion_scheduled` | Your account was scheduled for deletion |
| `account.deletion_cancelled` | That deletion was cancelled |
| `proof.reissued` | A stored proof token was re-signed under a newer signing key. The body is a reference, never the token: `proof_id`, `kid`, `reissued_at` and `token_url` (the authenticated token pull) — fetch the new token there. Not sent for a proof of the account's own verified assets |
| `delegation.reissued` | A delegation's token was re-signed under a newer signing key: `handle`, `kid`, `reissued_at` and `token_url` (the public delegation pull). Account-level; an endpoint subscribed to `proof.*` does not receive it |

The `verification.completed` body is flat. It goes to every active endpoint serving the verification's sub-account that subscribes to `verification.completed`, `verification.*` or `*`, or sets no event filter (all events):

```json
{
  "event": "verification.completed",
  "profile_id": "65f1a2b3c4d5e6f7a8b9c0d2",
  "verification_id": "65f1a2b3c4d5e6f7a8b9c0d1",
  "type": "phone",
  "channel": "sms",
  "identifier": "+15551234567",
  "status": "verified",
  "environment": "production",
  "external_user_id": "user_123",
  "client_metadata": { "order_id": "42" },
  "verified_at": "2026-10-25T10:30:00.000Z",
  "proof_token": "eyJhbGciOiJFUzI1NiIs...",
  "proof_expires_at": "2027-10-25T10:30:00.000Z",
  "confidence": "standard"
}
```

`confidence` is present only when the channel sets one (SMS). A test-mode verification carries `"environment": "test"` and `"test": true`. A phone-first session's body also carries `session_id` (the `sess_…` id `POST /sessions` returned): correlate the session by it, since its `verification_id` is created when it completes.

### Webhook Signature

Every delivery carries an `X-Proof-Timestamp` header (Unix seconds) and an `X-Proof-Signature` header: the hex HMAC-SHA256, keyed with your webhook secret, of the timestamp, a `.`, and the raw request body exactly as received. Verify it against the raw bytes, before parsing the JSON, and refuse a timestamp that is not recent. The sample also refuses an empty secret (anyone could compute a signature with it), a missing one, a body that is not a `Buffer`, and a signature that is not a string.

```javascript
const crypto = require('crypto');

const MAX_AGE_SECONDS = 300;

function verifyWebhook(rawBody, timestamp, signature, secret) {
  if (typeof secret !== 'string' || secret.length === 0) return false;
  if (!Buffer.isBuffer(rawBody)) return false;

  const age = Math.abs(Date.now() / 1000 - Number(timestamp));
  if (!Number.isFinite(age) || age > MAX_AGE_SECONDS) return false;

  const expected = crypto
    .createHmac('sha256', secret)
    .update(`${timestamp}.${rawBody}`)
    .digest('hex');

  if (typeof signature !== 'string') return false;
  const a = Buffer.from(signature, 'utf8');
  const b = Buffer.from(expected, 'utf8');
  return a.length === b.length && crypto.timingSafeEqual(a, b);
}

// rawBody: the request body as a Buffer, e.g. from express.raw({ type: 'application/json' })
// verifyWebhook(rawBody, req.headers['x-proof-timestamp'], req.headers['x-proof-signature'], secret)
```

Every attempt of one delivery carries the same `X-Proof-Delivery` value (`X-Webhook-Id` changes per attempt). A retried delivery carries the same `proof_token` as the first attempt, or `proof_token: null` with a reason you can read from [Get Webhook Delivery](#get-webhook-delivery), never a different token.

---

## Sub-accounts and keys

An account holds one or more sub-accounts. Each sub-account has its own branding, message templates, verification requests, verifications, phone sessions, approval setups and the confirmations and consents sent through them, and circles. What you prove or set up once — phones, emails and domains, and the delegations issued from a domain — belongs to the account.

| Credential | Reaches |
|------------|---------|
| API key bound to a sub-account | That sub-account only |
| Account-wide API key | Every sub-account of the account |
| Dashboard session | Every sub-account of the account |

A key bound to a sub-account:

- **Creates in its own sub-account.** A request body or query naming another sub-account (`profile_id`, `public_profile_id`, `business_profile_id`) returns `403 profile_mismatch`.
- **Sees only its own sub-account's records.** A record of another sub-account answers exactly like an id that does not exist — the endpoint's own `404` — on every read and every change, and nothing is sent or charged. Lists, `/api/v1/me/search`, `/api/v1/me/assets/search` and `/api/v1/me/summary` contain only its own sub-account's records. A record that carries no sub-account — an older record, made before its kind recorded one — belongs to the primary sub-account, and a record created by one of the sub-account's keys stays the sub-account's, also when that key was bound later.
- **Reads account-level things but does not change them.** It can read settings, usage, billing, the list of sub-accounts, and the account's phones, emails, domains, delegations and Lookalike Watch. Every account-level change returns `403 account_key_required`: account settings (including the deprecated webhook block in `/api/v1/me/settings`), billing, creating or deleting a sub-account or choosing the primary, starting, checking or removing a domain verification, issuing or revoking a delegation, revoking, suspending or unsuspending a proof of a shared asset, Lookalike Watch, ending a sign-in session and retrying a webhook delivery. Use an account-wide key or the dashboard for these. A domain proof the sub-account made itself — a customer's domain verified through its own key — is the sub-account's own: its key may revoke, suspend and unsuspend it, unless the domain is also one of the account's own domains or delegations are live under that proof.
- **Changes its own sub-account's profile.** `PUT /api/v1/profiles/me` and `PATCH /api/v1/me/profiles/:profileId` work on its own sub-account; `GET /api/v1/profiles/me` returns it. Any other sub-account, or making its own the primary, returns `403 account_key_required` — as does the deletion preview of another sub-account.
- **Manages only its own sub-account's webhook endpoints.** It sees and changes only endpoints scoped to exactly its own sub-account (any other answers `404`), may create one so scoped, and may not widen one (`403 profile_mismatch`). It may rotate its own sub-account's callback signing secret.
- **Sees its own sub-account's webhook deliveries.** Those sent to its callback URLs and to the endpoints it can see — nothing sent to an endpoint that serves other sub-accounts too.

Two limits apply to the whole account rather than to each sub-account:

- Approval setup names and verification request `reference_id`s are unique per account. A `reference_id` already used in another sub-account returns `409 duplicate_reference_id` without naming that request.
- Anyone can start a sign-in (`POST /api/v1/auth/sessions`), with or without a bound key, and once a person approves it the result is a full account session. Approve only sign-ins you started yourself.

| Code | Status | Meaning |
|------|--------|---------|
| `profile_mismatch` | 403 | A key bound to a sub-account named another sub-account |
| `account_key_required` | 403 | A key bound to a sub-account attempted an account-level change, a change to another sub-account, or the deletion preview of another sub-account |

---

## Verification Statuses

| Status | Description |
|--------|-------------|
| `pending` | Challenge created, waiting for user |
| `verified` | Successfully verified, proof available |
| `failed` | Max attempts exceeded |
| `expired` | Challenge expired |
| `cancelled` | Another channel of the same multi-channel phone verification completed first |
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
| `channel_not_available` | 400 | The channel is not available yet (social, wallet and account verification are coming soon) |
| `missing_api_key` | 401 | No usable API key in the `Authorization` header |
| `invalid_api_key` | 401 | The API key is unknown, revoked or expired |
| `not_found`, `<resource>_not_found` | 404 | Resource not found; many endpoints name it, e.g. `domain_not_found` |
| `challenge_expired` | 400 | Verification challenge expired |
| `rate_limit_exceeded` | 429 | Too many requests (see Rate Limits) |

---

## Rate Limits

Each account has a per-second request allowance set by its plan. All of the account's API keys
share it, and most routes an API key reaches draw on it:

| Plan | Requests per second |
|------|---------------------|
| Pay as you go | 10 |
| €9 | 20 |
| €29 | 50 |
| €79 | 100 |
| Custom | 500 |

- Short bursts of up to 3× the allowance are absorbed; beyond that the API answers `429 rate_limit_exceeded` with a `Retry-After` header. A response from a route that applies the allowance carries `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset`.
- Most routes also have a limit per IP address, checked before the API key is read, and several routes can share one. Requests from a single address can reach it before your plan's allowance. Its 429 carries the same `rate_limit_exceeded` code with a message of its own, and any `X-RateLimit-*` headers it sets describe that limit rather than your plan's.
- Webhook deliveries: automatic retry with exponential backoff

---

## See Also

| Feature | Endpoints | Documentation |
|---------|-----------|---------------|
| **Smart Reuse** | `GET /api/v1/verify/request/:id/proofs`, `POST .../auto-verify` | [Smart Reuse](/docs/smart-reuse) |
| **Multi-Profile** | `GET/POST /api/v1/me/profiles`, `PATCH/DELETE .../profiles/:id` | [Multi-Profile System](/docs/profiles) |
| **Message Templates** | `GET /api/v1/templates`, `PUT /api/v1/templates/:channel/:messageType` | [Message Templates](/docs/templates) |
