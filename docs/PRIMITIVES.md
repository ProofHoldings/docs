# proof.holdings Primitives

> The complete mental model in one page

---

## One Sentence

**proof.holdings answers: "Does this person control this digital asset right now?"**

Not identity. Just control.

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
| `phone` | whatsapp, telegram, sms, viber | +37069199199 |
| `email` | email | user@example.com |
| `domain` | dns, http | example.com |
| `social` | github, google, twitter, ... | @username |
| `wallet` | ethereum, solana, bitcoin | 0xAb5801... |

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
- Low cost (Telegram/WhatsApp inbound is free; SMS inbound has carrier costs)
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

## That's It

Three primitives. Four steps. Offline-verifiable proofs.

```
createChallenge → user action → verifyProof
                                    ↓
                              revokeProof (if needed)
```

Everything else is implementation detail.

---

## Resources

- [API Explorer](https://api.proof.holdings/api/docs) — Interactive Swagger UI
- [OpenAPI Spec](https://api.proof.holdings/api/openapi.json) — Machine-readable API definition
- [JWKS Public Keys](https://api.proof.holdings/.well-known/jwks.json) — For offline proof verification
- [API Status](https://api.proof.holdings/health) — Health check endpoint
- [GitHub Docs](https://github.com/ProofHoldings/docs) — Source of truth for all documentation
- [Security & Threat Model](/docs/threat-model) — Guarantees, non-guarantees, and attack vectors
