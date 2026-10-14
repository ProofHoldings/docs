---
title: Core Primitives
lastUpdated: "2026-08-26"
status: current
---

# proof.holdings Primitives

> The complete mental model in one page

---

## One Sentence

**proof.holdings answers: "Does this person control this digital asset right now?"**

Not identity. The control. That question — asked of phones, consents, decisions, and identities — grows into seven verification primitives, each minting the same offline-verifiable proof token.

---

## The Seven Primitives

Four are live today; three are coming soon. Every primitive mints an ES256-signed JWT bearer proof token.

| # | Primitive | Status | What it proves |
|---|-----------|--------|----------------|
| 1 | [Proof of Control](/verify) | live | Control of a phone, email, or domain — social accounts and wallets coming soon |
| 2 | [Proof of Delegation](/delegation) | live | A control-proven domain authorized this artifact — an MCP server, package or endpoint — for named scopes, revocably |
| 3 | [Proof of Action](/primitives/action) | live | A human approved this specific irreversible action |
| 4 | [Proof of Me](/primitives/me) | live | The person reaching out really is you — reverse verification via trusted Circle members ([docs](/docs/proof-me)) |
| 5 | [Proof of Agreement](/primitives/agreement) | coming soon | Group conversations preserved as verifiable records — no API endpoints yet |
| 6 | [Proof of Development](/primitives/development) | coming soon | Verifiable development provenance via team.management — no API endpoints yet |
| 7 | [Proof of Enforcement](/primitives/enforcement) | coming soon | Mandated processes demonstrably ran — no API endpoints yet |

This page covers the live primitives' mechanics. Proof of Me has its own deep-dive at [/docs/proof-me](/docs/proof-me).

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

## Core API Operations

Your entire Proof of Control integration uses three functions:

```
createChallenge(asset)  →  challenge
verifyProof(proof)      →  { valid, claims }
revokeProof(proofId)    →  { success }
```

> **Test mode:** All three operations work identically with `pk_test_` keys. No real messages are sent, and you can auto-complete verifications with `POST /api/v1/verifications/:id/test-verify`. See [API Reference — Test Mode](/docs/api#test-mode-sandbox).

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
jwt.verify(token, publicKey, { algorithms: ['ES256', 'RS256'] })
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

## Proof of Control

The first live primitive ([overview page](/verify)): verify that a person or business controls an asset they claim. The challenge/verify/revoke operations above, the asset types and Reverse OTP below, are its mechanics. Reverse OTP applies to phones — the user sends the message TO Proof; email uses magic link/OTP, domains use DNS/HTTP/admin email; social (OAuth) and wallets (signing) are coming soon.

### Asset Types

| Type | Channels | Example |
|------|----------|---------|
| `phone` | whatsapp, telegram, sms | +37069199199 |
| `email` | email | user@example.com |
| `domain` | dns, http, email, auto | example.com |
| `social` (coming soon) | github, google, facebook, x, linkedin, instagram, youtube, tiktok | @username |
| `wallet` (coming soon) | ethereum, solana, bitcoin | 0xAb5801... |
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
- Signed with ES256 — Offline-verifiable via JWKS

---

## Offline Verification

Proofs can be verified without calling our API:

```javascript
// 1. Fetch the key set (cache for 24h)
const jwks = await fetch('https://api.proof.holdings/.well-known/jwks.json').then(r => r.json());

// 2. Check revocation list (cache for 5min)
const revoked = await fetch('https://api.proof.holdings/api/v1/proofs/revoked').then(r => r.json());

// 3. Verify locally with the key the token names — a rotation puts two in the set
const { kid } = JSON.parse(Buffer.from(token.split('.')[0], 'base64url').toString());
const publicKey = createPublicKey({ key: jwks.keys.find(k => k.kid === kid), format: 'jwk' });
const payload = jwt.verify(token, publicKey, { algorithms: ['ES256', 'RS256'] });

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

| Operation | Endpoint | Auth |
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
# Then: jwt.verify(proof_token, publicKey, {algorithms: ['ES256', 'RS256']})
```

---

---

## Proof of Delegation

Proof of Delegation is a signed statement that whoever controls a domain authorized a named artifact for a listed set of capabilities — nothing more. It says nothing about the artifact, its safety, or the business behind it.

The **principal** is a domain proven through Proof of Control, addressed by its `ph_ctl_*` handle. The **delegate** is a typed identifier — an `https` URL or a package URL (`purl`) — so the token points at exactly one package, container or endpoint. The **scope** is an explicit list of lowercase capability tokens (`send-email`, not `payments:read` — an API-scope form is rejected as `invalid_scope`). The delegation mints an ES256 token you publish alongside the artifact, and anyone can verify that token offline.

What a verified result establishes is bounded and worth stating plainly: **the domain you expected authorized this artifact for these scopes** — not that the artifact is safe, audited or endorsed, and not that another domain has not published its own delegation over the same artifact. Any domain controller can mint a valid delegation naming someone else's package, so a verifier that does not pin the expected domain itself learns only that *some* domain made a claim.

Revoking is immediate at the API and reaches every verifier reading the published status list within a minute. Revoking one delegation leaves the domain proof and every sibling delegation intact; revoking the domain's control proof cascades to every delegation issued from it.

### Delegation Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  YOU                    PROOF.HOLDINGS           ANY VERIFIER    │
│                                                                  │
│     │  prove the domain        │                      │          │
│     │ ──────────────────────►  │  ph_ctl_… handle     │          │
│     │ ◄──────────────────────  │                      │          │
│     │                          │                      │          │
│     │  POST /delegations       │                      │          │
│     │  { control_proof,        │                      │          │
│     │    delegate, scope }     │                      │          │
│     │ ──────────────────────►  │                      │          │
│     │  { token: "eyJ..." }     │                      │          │
│     │ ◄──────────────────────  │                      │          │
│     │                          │                      │          │
│     │  publish the token on the artifact ───────────► │          │
│     │                          │  status list  ─────► │  offline │
│     │                          │                      │  verify  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Delegation Endpoints

| Action | Endpoint | Auth |
|--------|----------|------|
| Issue a delegation | `POST /api/v1/delegations` | API Key |
| List your delegations | `GET /api/v1/delegations` | API Key |
| Get a delegation (incl. token) | `GET /api/v1/delegations/:id` | API Key |
| Revoke a delegation | `POST /api/v1/delegations/:id/revoke` | API Key |

### Key Properties

- **Domain-anchored** — Every delegation is issued from a control proof you already hold
- **Typed** — The delegate is a `url` or a `purl`, so the token names one artifact, not a description of one
- **Scoped** — Capability tokens state what was authorized
- **Expiring** — A delegation can never outlive the control proof beneath it
- **Revocable** — Per delegation, without touching the domain proof or its siblings
- **Publishable** — The token is meant to be published on the artifact and verified offline

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
  "message": "{\"v\":1,\"alg\":\"RSA-OAEP-256+AES-256-GCM\",\"ek\":\"...\",\"iv\":\"...\",\"ct\":\"...\",\"tag\":\"...\",\"kid\":\"...\"}"
}
```

`message` is the ciphertext envelope, never the sentence: the approver decrypts in
their browser, so plaintext is refused with `400 message_not_encrypted`. Build the
envelope with an SDK (`encryptMessage` / `encrypt_message`) using the config's public
key, or let the JS helper `confirmations.createAndWait` do both steps — that helper
builds a v1 (owner-only) envelope, so a config with ENROLLED APPROVERS needs a v2 one
(`encryptMessageV2` with the keys from `GET /api/v1/hitl/:id/approver-keys`), otherwise
the call is refused with `400 envelope_missing_approvers`.

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
- **Cryptographic proof** — Every decision produces an ES256-signed JWT proof token
- **First-response-wins** — Atomic resolution prevents race conditions
- **Webhooks** — `confirmation.approved` and `confirmation.denied` events fire on resolution; the body names its event in `event` (the same value as the `X-Proof-Event` header)

### Agent Contact Authorizations (consent)

Before an agent may send approval requests to a person, that person has to allow it. That consent is an **authorization** — a scoped, revocable grant the user approves on their own channel. It lives here, with the approvals it gates, and not under Proof of Delegation: its scope union is `confirmations | identity_challenge`, so it serves this primitive and Proof of Me. (This documentation described it as the delegation primitive until 2026-08 — the two are separate objects on separate routes.)

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

| Action | Endpoint | Auth |
|--------|----------|------|
| Request authorization | `POST /api/v1/authorizations` | API Key |
| Get authorization | `GET /api/v1/authorizations/:id` | API Key |
| List authorizations | `GET /api/v1/authorizations` | API Key |
| Revoke authorization | `DELETE /api/v1/authorizations/:id` | API Key |
| Export authorizations | `GET /api/v1/authorizations/export` | API Key |

- **Consent-based** — Users must explicitly allow agents to contact them
- **Revocable** — Users or API consumers can revoke authorization at any time
- **Scoped** — Authorizations define what the agent is allowed to do
- **Auditable** — Every authorization produces a proof token for compliance
- **GDPR-friendly** — Exportable consent records with revocation support
- **Webhooks** — `authorization.approved` and `authorization.denied` fire when the person answers, so you learn the decision without polling. The body carries `authorization_id`, `status`, `channel` and the proof token — not the person's phone or chat id
- **Replying in words** — On Telegram or WhatsApp, a person who types "Allow" instead of tapping gets the Allow/Deny buttons sent again (at most once every 10 minutes); a typed word is never taken as consent

---

## Proof of Me

Proof of Me (live — [overview page](/primitives/me)) is anti-impersonation **reverse verification**. Instead of proving your own identity, you let trusted contacts confirm that a person reaching out to them is really you: enroll named **members** on a Circle; when someone suspicious contacts a member claiming to be you, one tap sends you a CONFIRM on a **different channel** than the impersonator used; you approve or deny and the member is told the result. Scheduled drills keep members trained.

Endpoints (the member surface under `/api/v1/circles`, plus `/api/v1/identity-challenges`), SDK usage, and the full flow live in the dedicated [Proof of Me docs](/docs/proof-me).

---

## Coming Soon

Three more primitives are on the way — no API endpoints yet; their overview pages carry the vision and a notify option:

- **[Proof of Agreement](/primitives/agreement)** (coming soon) — continuous witnessing for group conversations: every message in a group that opts in becomes a verifiable record. Starts only when every member agrees.
- **[Proof of Development](/primitives/development)** (coming soon) — verifiable development provenance via the team.management integration: every completed task issues a signed token recording how, what, why, when, by whom.
- **[Proof of Enforcement](/primitives/enforcement)** (coming soon) — proof that mandated protocols and processes actually ran across teams.

---

## That's It

Seven primitives — four live today, three coming soon. Offline-verifiable proofs for everything.

```
Proof of Control:    createChallenge → user action → verifyProof
Proof of Delegation: proven domain → createDelegation over url/purl → published token
Proof of Action:     createConfirmation → human decides → proof token
Proof of Me:         enroll Circle members → challenge → cross-channel CONFIRM → proof token
Coming soon:         Agreement · Development · Enforcement
```

Everything else is implementation detail.

---

## Resources

- [API Reference](/docs/api) — Complete endpoint documentation
- [OpenAPI Spec](https://api.proof.holdings/api/openapi.json) — Machine-readable OpenAPI 3.1 specification
- [JWKS Public Keys](https://api.proof.holdings/.well-known/jwks.json) — For offline proof verification
- [API Status](https://api.proof.holdings/health) — Health check endpoint
- [GitHub Docs](https://github.com/ProofHoldings/docs) — Source of truth for all documentation
- [Security & Threat Model](/docs/threat-model) — Guarantees, non-guarantees, and attack vectors
