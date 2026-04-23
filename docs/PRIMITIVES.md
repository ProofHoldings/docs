---
title: Core Primitives
lastUpdated: "2026-03-19"
status: current
---

# proof.holdings Primitives

> The complete mental model in one page

---

## One Sentence

**proof.holdings answers: "Does this person control this digital asset right now?"**

Not identity. The control.

---

## The Abstraction

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ASSET ──────► CHALLENGE ──────► USER ACTION ──────► PROOF               │
│                                                                             │
│    "phone"       "Send X7K2M9     User sends         Signed JWT            │
│    "+37069199199" via WhatsApp"   the message        (offline-verifiable)  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

That's it. Four steps. Everything else is implementation detail.

---

## Three Primitives

Your entire integration uses three functions:

```
createChallenge(asset)  →  challenge
verifyProof(proof)      →  { valid, claims }
revokeProof(proofId)    →  { success }
```

> **Test mode:** All three primitives work identically with `pk_test_` keys. No real messages are sent, and you can auto-complete verifications with `POST /api/v1/verifications/:id/test-verify`. See [API Reference — Test Mode](/docs/api#test-mode-sandbox).

### 1. createChallenge

Ask proof.holdings to create a verification challenge.

```bash
POST /api/v1/verifications
{
  "type": "phone",
  "channel": "whatsapp",
  "identifier": "+37069199199"
}
```

Response:
```json
{
  "id": "abc123",
  "type": "phone",
  "channel": "whatsapp",
  "status": "pending",
  "identifier": "+37069199199",
  "challenge": {
    "code": "X7K2M9",
    "instruction": "Send X7K2M9 via WhatsApp"
  },
  "deep_link": "https://wa.me/37069199199?text=X7K2M9"
}
```

### 2. verifyProof

Verify a proof token. **Works offline** with JWKS public key.

```bash
# Online (via API)
POST /api/v1/proofs/validate
{ "proof_token": "eyJhbG..." }

# Offline (with public key)
jwt.verify(token, publicKey, { algorithms: ['RS256'] })
```

Response:
```json
{
  "valid": true,
  "verification": {
    "type": "phone",
    "channel": "whatsapp",
    "identifier": "+37069199199",
    "verified_at": "2026-02-04T10:00:00Z",
    "expires_at": "2026-03-06T10:00:00Z"
  }
}
```

### 3. revokeProof

Invalidate a proof (GDPR deletion, fraud, etc).

```bash
POST /api/v1/proofs/{id}/revoke
{ "reason": "user_request" }
```

---

## Asset Types

| Type | Channels | Example |
|------|----------|---------|
| `phone` | whatsapp, telegram, sms | +37069199199 |
| `email` | email | user@example.com |
| `domain` | dns, http, email, auto | example.com |
| `social` | github, google, facebook, x, linkedin, instagram, youtube, tiktok | @username |
| `wallet` | ethereum, solana, bitcoin | 0xAb5801... |
| `account` | coinbase, kraken | user@exchange |
| `telegram_bot` | telegram_bot_token | @mybusinessbot |
---

## The Innovation: Reverse OTP

Traditional OTP:
```
Server sends SMS to user  →  User enters code  →  Verified
     ($0.05-0.40)              (can be phished)
```

Reverse OTP (proof.holdings):
```
Server shows code  →  User sends message TO server  →  Verified
     (low cost)          (user-initiated = secure)
```

**Why it matters:**
- Low cost (no per-message fees — you pay per proof, not per SMS)
- Secure (user initiates, can't be phished)
- Reliable (no carrier delivery issues for messaging apps)
- Challenge code can be pre-populated via deep link (e.g., `https://wa.me/37069199199?text=X7K2M9`)

---

## Proof Token Structure

```json
{
  "iss": "proof.holdings",
  "sub": "verification_id",
  "iat": 1707048000,
  "exp": 1709640000,

  "proof_schema_version": "1.0",
  "type": "phone",
  "channel": "whatsapp",
  "identifier_hash": "sha256:1a84d7ea...",
  "verified_at": "2026-02-04T10:00:00Z",
  "user_id": "owner_id"
}
```

**Key properties:**
- `identifier_hash` — Privacy: raw identifier not exposed
- `exp` — Time-bounded: proofs expire
- Signed with RS256 — Offline-verifiable via JWKS

---

## Offline Verification

Proofs can be verified without calling our API:

```javascript
// 1. Fetch public key (cache for 24h)
const jwks = await fetch('https://api.proof.holdings/.well-known/jwks.json');
const publicKey = jwks.keys[0];

// 2. Check revocation list (cache for 5min)
const revoked = await fetch('https://api.proof.holdings/api/v1/proofs/revoked');

// 3. Verify locally
const payload = jwt.verify(token, publicKey, { algorithms: ['RS256'] });

// 4. Check not revoked
const isRevoked = revoked.revoked.some(r => r.proof_id === payload.sub);

// 5. Done - no API call needed
return !isRevoked && payload.exp > Date.now() / 1000;
```

---

## What proof.holdings IS

- ✅ Proof-of-control primitive
- ✅ Verification infrastructure
- ✅ Passwordless authentication building block
- ✅ Multi-asset verification API

## What proof.holdings is NOT

- ❌ Identity provider (we don't know WHO)
- ❌ KYC solution (we don't verify documents)
- ❌ Fraud prevention (we provide signals)
- ❌ Permanent ownership (proofs expire)

---

## Complete Flow Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  YOUR APP                        PROOF.HOLDINGS                    USER      │
│  ────────                        ──────────────                    ────      │
│                                                                              │
│     │                                  │                             │       │
│     │  POST /verifications             │                             │       │
│     │  { type: "phone",                │                             │       │
│     │    channel: "whatsapp",          │                             │       │
│     │    identifier: "+370..." }       │                             │       │
│     │ ─────────────────────────────►   │                             │       │
│     │                                  │                             │       │
│     │  { challenge: { code: "X7K2M9" },│                             │       │
│     │    deep_link: "wa.me/..." }      │                             │       │
│     │ ◄─────────────────────────────   │                             │       │
│     │                                  │                             │       │
│     │                                  │                             │       │
│     │  Show: "Send X7K2M9             │                             │       │
│     │         via WhatsApp"            │                             │       │
│     │ ─────────────────────────────────────────────────────────────► │       │
│     │                                  │                             │       │
│     │                                  │   User sends "X7K2M9"       │       │
│     │                                  │   via WhatsApp              │       │
│     │                                  │ ◄─────────────────────────  │       │
│     │                                  │                             │       │
│     │                                  │  (Webhook to your app)      │       │
│     │ ◄─────────────────────────────   │                             │       │
│     │  { event: "verified",            │                             │       │
│     │    proof_token: "eyJ..." }       │                             │       │
│     │                                  │                             │       │
│     │                                  │                             │       │
│     │  Verify proof (offline)          │                             │       │
│     │  jwt.verify(token, publicKey)    │                             │       │
│     │                                  │                             │       │
│     │  ✓ DONE                          │                             │       │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Endpoints Summary

| Primitive | Endpoint | Auth |
|-----------|----------|------|
| createChallenge | `POST /api/v1/verifications` | API Key |
| pollStatus | `GET /api/v1/verifications/:id` | API Key |
| verifyProof | `POST /api/v1/proofs/validate` | API Key |
| verifyProof (offline) | JWKS + local verify | None |
| revokeProof | `POST /api/v1/proofs/:id/revoke` | API Key |
| getRevocationList | `GET /api/v1/proofs/revoked` | None |
| getPublicKey | `GET /.well-known/jwks.json` | None |

---

## Quick Start

```bash
# 1. Create verification
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{"type":"phone","channel":"whatsapp","identifier":"+37069199199"}'

# 2. User sends challenge code via WhatsApp (or clicks deep_link)

# 3. Poll for completion (or use webhook)
curl https://api.proof.holdings/api/v1/verifications/{id} \
  -H "Authorization: Bearer pk_live_..."

# 4. Verify proof offline
curl https://api.proof.holdings/.well-known/jwks.json
# Then: jwt.verify(proof_token, publicKey, {algorithms: ['RS256']})
```

---

---

## Proof of Delegation

Proof of Delegation lets users authorize AI agents and automations to contact them. Before an agent can send approval requests via Telegram or WhatsApp, the user must explicitly consent.

### Authorization Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  YOUR APP               PROOF.HOLDINGS              USER        │
│                                                                  │
│     │  POST /authorizations    │                      │         │
│     │  { channel, scope }      │                      │         │
│     │ ──────────────────────►  │                      │         │
│     │                          │  "Allow approvals?"  │         │
│     │                          │ ──────────────────►  │         │
│     │                          │                      │         │
│     │                          │  User taps "Allow"   │         │
│     │                          │ ◄──────────────────  │         │
│     │                          │                      │         │
│     │  { status: "active",     │                      │         │
│     │    proof_token: "eyJ..." }                      │         │
│     │ ◄──────────────────────  │                      │         │
│     │                          │                      │         │
│     │  ✓ Agent can now send    │                      │         │
│     │    approval requests     │                      │         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Authorization Endpoints

| Action | Endpoint | Auth |
|--------|----------|------|
| Request authorization | `POST /api/v1/authorizations` | API Key |
| Get authorization | `GET /api/v1/authorizations/:id` | API Key |
| List authorizations | `GET /api/v1/authorizations` | API Key |
| Revoke authorization | `DELETE /api/v1/authorizations/:id` | API Key |
| Export authorizations | `GET /api/v1/authorizations/export` | API Key |

### Key Properties

- **Consent-based** — Users must explicitly allow agents to contact them
- **Revocable** — Users or API consumers can revoke authorization at any time
- **Scoped** — Authorizations define what the agent is allowed to do
- **Auditable** — Every authorization produces a proof token for compliance
- **GDPR-friendly** — Exportable consent records with revocation support

---

## Proof of Action (HITL)

Proof of Action is the human-in-the-loop (HITL) primitive. AI agents and automations request human approval before taking action. The human receives a message via Telegram or WhatsApp, decides to approve or deny, and every decision is cryptographically signed.

### HITL Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  AI AGENT              PROOF.HOLDINGS              HUMAN        │
│                                                                  │
│     │  POST /confirmations     │                      │         │
│     │  { hitl_id, message,     │                      │         │
│     │    timeout: 43200 }      │                      │         │
│     │ ──────────────────────►  │                      │         │
│     │                          │  "Deploy to prod?"   │         │
│     │                          │  [Approve] [Deny]    │         │
│     │                          │ ──────────────────►  │         │
│     │                          │                      │         │
│     │                          │  Human taps Approve  │         │
│     │                          │  + optional comment   │         │
│     │                          │ ◄──────────────────  │         │
│     │                          │                      │         │
│     │  { status: "approved",   │                      │         │
│     │    proof_token: "eyJ...",│                      │         │
│     │    response: {           │                      │         │
│     │      action: "approve",  │                      │         │
│     │      text: "Ship it!" } }│                      │         │
│     │ ◄──────────────────────  │                      │         │
│     │                          │                      │         │
│     │  ✓ Agent proceeds        │                      │         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Setup: HITL Config

Before sending approval requests, create a HITL config that defines the channels and timeout:

```bash
POST /api/v1/hitl
{
  "name": "Production Deploys",
  "channels": [
    { "type": "telegram", "config": { "chat_id": "123456789" } }
  ],
  "timeout_seconds": 43200
}
```

WhatsApp channels accept a phone number, a Business-Scoped User ID (BSUID), or both:

```bash
POST /api/v1/hitl
{
  "name": "Wire Approvals",
  "channels": [
    { "type": "whatsapp", "config": { "phone_number": "+15551234567" } }
  ]
}

# Or using a BSUID (Meta's privacy-preserving identifier):
{
  "channels": [
    { "type": "whatsapp", "config": { "bsuid": "US.abcdef1234567890" } }
  ]
}
```

### Request Approval

```bash
POST /api/v1/confirmations
{
  "hitl_id": "YOUR_HITL_CONFIG_ID",
  "message": "Deploy v2.1 to production?"
}
```

### Confirmation Endpoints

| Action | Endpoint | Auth |
|--------|----------|------|
| Create HITL config | `POST /api/v1/hitl` | API Key |
| List HITL configs | `GET /api/v1/hitl` | API Key |
| Get HITL config | `GET /api/v1/hitl/:id` | API Key |
| Update HITL config | `PATCH /api/v1/hitl/:id` | API Key |
| Archive HITL config | `DELETE /api/v1/hitl/:id` | API Key |
| Request approval | `POST /api/v1/confirmations` | API Key |
| Get confirmation | `GET /api/v1/confirmations/:id` | API Key |
| List confirmations | `GET /api/v1/confirmations` | API Key |

### Key Properties

- **Async** — Approvals via Telegram (inline keyboards) or WhatsApp (interactive buttons)
- **Configurable timeout** — Default 1 hour, max 24 hours; confirmation expires if no response
- **Comments** — Humans can add a comment with their approve/deny decision
- **Cryptographic proof** — Every decision produces an RS256-signed JWT proof token
- **First-response-wins** — Atomic resolution prevents race conditions
- **Webhooks** — `confirmation.approved` and `confirmation.denied` events fire on resolution

---

## That's It

Three primitives. Offline-verifiable proofs for everything.

```
Proof of Control:    createChallenge → user action → verifyProof
Proof of Delegation: requestAuthorization → user consents → proof token
Proof of Action:     createConfirmation → human decides → proof token
```

Everything else is implementation detail.

---

## Resources

- [API Reference](/docs/api) — Complete endpoint documentation
- [OpenAPI Spec](https://api.proof.holdings/api/openapi.json) — Machine-readable OpenAPI 3.0 specification
- [JWKS Public Keys](https://api.proof.holdings/.well-known/jwks.json) — For offline proof verification
- [API Status](https://api.proof.holdings/health) — Health check endpoint
- [GitHub Docs](https://github.com/ProofHoldings/docs) — Source of truth for all documentation
- [Security & Threat Model](/docs/threat-model) — Guarantees, non-guarantees, and attack vectors
