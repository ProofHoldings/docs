---
title: How it works
lastUpdated: "2026-10-26"
status: current
---

# How it works

> The complete mental model in one page

---

## One Sentence

**Proof Holdings answers one question: "Does this person control this digital asset right now?"**

Not identity. The control. That question — asked of phones, consents, decisions, and identities — grows into seven verification primitives, each minting the same offline-verifiable proof token.

---

## The Seven Primitives

Four are live today; three are coming soon. Every primitive mints an ES256-signed JWT bearer proof token.

| # | Primitive | Status | What it proves |
|---|-----------|--------|----------------|
| 1 | [Proof of Control](/verify) | live | Control of a phone, email, or domain — social accounts and wallets coming soon |
| 2 | [Proof of Delegation](/delegation) | live | A control-proven domain authorized this artifact — an MCP server, package or endpoint — revocably |
| 3 | [Proof of Action](/primitives/action) | live | A human approved this specific irreversible action |
| 4 | [Proof of Me](/primitives/me) | live | The person reaching out really is you — reverse verification via trusted Circle members ([docs](/docs/proof-me)) |
| 5 | [Proof of Agreement](/primitives/agreement) | coming soon | Group conversations preserved as verifiable records — no API endpoints yet |
| 6 | [Proof of Development](/primitives/development) | coming soon | Verifiable development provenance via team.management — no API endpoints yet |
| 7 | [Proof of Enforcement](/primitives/enforcement) | coming soon | Mandated processes demonstrably ran — no API endpoints yet |

This page covers the live primitives' mechanics. Proof of Me has its own deep-dive at [/docs/proof-me](/docs/proof-me).

---

## How each live proof runs

Each live proof follows the same path: set up, call, wait, check. The sections below this one
give the detail for each step.

### Control

1. **Set up** — create an API key in the dashboard. A `pk_test_` key sends no real messages.
2. **Call** — `POST /api/v1/verifications` with the asset type, channel and identifier. The
   response carries the challenge: a code, our number to send it to, and a deep link.
3. **Wait** — show the person the code. They send it to us from the phone or account being
   verified. Poll `GET /api/v1/verifications/:id` until `status` is `verified`, or receive
   `verification.completed` at your account webhook URL. A verification request sends
   its own events to its own `callback_url`.
4. **Check** — validate the proof token, online with `POST /api/v1/proofs/validate` or offline
   against our public keys ([Offline Verification](#offline-verification)).

### Delegation

1. **Set up** — prove control of the domain with a `domain` verification and keep its `ph_ctl_*`
   handle.
2. **Call** — `POST /api/v1/delegations` with that handle, the delegate (a `url` or a `purl`) and
   the scope. The response carries the delegation token.
3. **Wait** — nobody has to answer: the token is issued at once. Publish it alongside the
   artifact.
4. **Check** — a verifier checks the published token offline against our public keys and the
   status list, and pins the domain they expected. See [Proof of Delegation](#proof-of-delegation)
   for what a verified result does and does not tell them.

### Action (approvals)

1. **Set up** — create an approval config (`POST /api/v1/hitl`), upload its encryption keys and
   finalize it. Finalizing sends each approver a request to allow your approval requests; each
   approver taps "Allow" on their own channel. Until one has, an approval request is refused
   with `403 no_authorized_channels`.
2. **Call** — `POST /api/v1/confirmations` with the config id and the encrypted message. The
   answer is always `pending`.
3. **Wait** — the approver reads the request in their browser and approves or denies it. Learn
   the decision from the `confirmation.approved` / `confirmation.denied` webhook or by polling
   `GET /api/v1/confirmations/:id`.
4. **Check** — an approver's decision carries a proof token (a timeout does not); validate it
   like any other.

### Me

1. **Set up** — create a Circle, add the people who know you as members, and send each one an
   invite. A member joins by tapping the Telegram or WhatsApp link.
2. **Call** — when someone contacts a member claiming to be you, the member taps a button, or
   you call `POST /api/v1/identity-challenges` with the Circle and the member.
3. **Wait** — you get a CONFIRM on a different channel from the one the impersonator used, and
   approve or deny it. Poll `GET /api/v1/identity-challenges/:id` for the result.
4. **Check** — the member is told the result; a confirmation carries a `proof_id` you can check
   or revoke. Details in the [Proof of Me docs](/docs/proof-me).

---

## Core API Operations

Your entire Proof of Control integration uses three functions:

```
createChallenge(asset)  →  challenge
verifyProof(proof)      →  { valid, claims }
revokeProof(proofId)    →  { success }
```

> **Test mode:** All three operations take `pk_test_` keys. No real messages are sent, you can auto-complete verifications with `POST /api/v1/verifications/:id/test-verify`, and `verifyProof` answers a test-mode token `{ valid: false, reason: "test_environment" }`: test-verify checks nothing. See [API Reference — Test Mode](/docs/api#test-mode-sandbox).

### 1. createChallenge

Ask us to create a verification challenge.

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
  "id": "6720a1b2c3d4e5f6a7b8c9d0",
  "type": "phone",
  "channel": "whatsapp",
  "status": "pending",
  "identifier": "+37069199199",
  "challenge": {
    "code": "X7K2M9",
    "expires_at": "2026-10-26T10:10:00Z",
    "send_to": "+15557766633",
    "send_to_formatted": "+1 (555) 776-6633",
    "instruction": "Send 'X7K2M9' to +1 (555) 776-6633 on whatsapp",
    "deep_link": "https://wa.me/15557766633?text=Verify%20X7K2M9"
  },
  "created_at": "2026-10-26T10:00:00Z"
}
```

The person sends the code from the phone being verified **to our verification number** (`challenge.send_to`).
`challenge.deep_link` opens WhatsApp with that message already typed.

### 2. verifyProof

Verify a proof token. The online check needs no API key; the offline check needs only our public keys (JWKS).

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
  "proof_id": "ph_ctl_9f2c4e7a1b3d5f6e8a0c2b4d6f8e1a3c",
  "verification": {
    "id": "6720a1b2c3d4e5f6a7b8c9d0",
    "type": "phone",
    "channel": "whatsapp",
    "identifier_hash": "1a84d7ea5c1f3b9e2d0a6c8f4b7e9d1c3a5f7b9d2e4c6a8f0b1d3e5c7a9f2b4d",
    "verified_at": "2026-10-26T10:00:00Z",
    "expires_at": "2027-10-26T10:00:00Z"
  }
}
```

The response never carries the raw phone number or email — only its SHA-256 hash. To check a
proof belongs to a specific identifier, send it as `identifier` beside `proof_token`; a proof
bound to another identifier answers `valid: false` with `reason: "identifier_mismatch"`.

### 3. revokeProof

Invalidate a proof (GDPR deletion, fraud, etc).

```bash
POST /api/v1/proofs/{id}/revoke
{ "reason": "user_request" }
```

---

## Proof of Control

The first live primitive ([overview page](/verify)): verify that a person or business controls an asset they claim. The challenge/verify/revoke operations above, the asset types and Reverse OTP below, are its mechanics. Reverse OTP applies to phones — the user sends the message to us; email uses magic link/OTP, domains use DNS/HTTP/admin email; social (OAuth) and wallets (signing) are coming soon.

### Asset Types

| Type | Channels | Example |
|------|----------|---------|
| `phone` | whatsapp, telegram, sms | +37069199199 |
| `email` | email | hello@team.management |
| `domain` | dns, http, email, auto | team.management |
| `social` (coming soon) | github, google, facebook, x, linkedin, instagram, youtube, tiktok | @username |
| `wallet` (coming soon) | ethereum, solana, bitcoin | 0xAb5801... |
| `account` (coming soon) | coinbase, kraken | user@exchange |
| `telegram_bot` | telegram_bot_token | @mybusinessbot |
---

## The Innovation: Reverse OTP

Traditional OTP:
```
Server sends SMS to user  →  User enters code  →  Verified
     (paid per message)        (can be phished)
```

Reverse OTP (Proof Holdings):
```
Server shows code  →  User sends message TO server  →  Verified
     (low cost)          (no code is sent to the user)
```

**Why it matters:**
- Low cost (no per-message fees — you pay per proof, not per SMS)
- No code travels to the person, so there is no code to steal from a fake "enter your code" page. A phisher can still ask someone to send a code to the phisher's own number — see the [threat model](/docs/threat-model).
- Reliable (no carrier delivery issues for messaging apps)
- The message can be pre-typed via a deep link to our verification number (`challenge.deep_link`, e.g. `https://wa.me/15557766633?text=Verify%20X7K2M9`)

---

## Proof Token Structure

```json
{
  "iss": "proof.holdings",
  "sub": "6720a1b2c3d4e5f6a7b8c9d0",
  "iat": 1793008800,
  "exp": 1824544800,

  "proof_schema_version": "1.3",
  "user_id": "6720a0f1e2d3c4b5a6978877",
  "type": "phone",
  "channel": "whatsapp",
  "identifier_hash": "1a84d7ea5c1f3b9e2d0a6c8f4b7e9d1c3a5f7b9d2e4c6a8f0b1d3e5c7a9f2b4d",
  "proof_id": "ph_ctl_9f2c4e7a1b3d5f6e8a0c2b4d6f8e1a3c",
  "jti": "ph_jti_4b6d8f0a2c4e6a8c0e2a4c6e8b0d2f4a",
  "status": {
    "status_list": { "idx": 4211, "uri": "https://api.proof.holdings/api/v1/proofs/status-list" }
  },
  "verified_at": "2026-10-26T10:00:00Z"
}
```

**Key properties:**
- `identifier_hash` — Privacy: the SHA-256 of the normalized identifier, as bare hex; the raw identifier is not in the token
- `proof_id` — The proof's stable public handle; `jti` names its current generation (verifying again gives a new `jti`; a refresh or a re-sign keeps it)
- `status.status_list` — Where a verifier reads the proof's current status (valid, revoked or suspended) without asking us about this proof
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

The revocation list does not show suspended proofs; check the status list (`status.status_list`
in the token), or use `proofs.verifyOffline` in any SDK (`verify_offline` in Python,
`VerifyOffline` in Go), which checks signature, expiry and status.

---

## What Proof Holdings is

- ✅ Proof-of-control primitive
- ✅ Verification infrastructure
- ✅ Passwordless authentication building block
- ✅ Multi-asset verification API

## What Proof Holdings is not

- ❌ Identity provider (we don't know WHO)
- ❌ KYC solution (we don't verify documents)
- ❌ Fraud prevention (we provide signals)
- ❌ Permanent ownership (proofs expire)

---

## Complete Flow Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  YOUR APP                        PROOF HOLDINGS                    USER      │
│  ────────                        ──────────────                    ────      │
│                                                                              │
│     │                                  │                             │       │
│     │  POST /verifications             │                             │       │
│     │  { type: "phone",                │                             │       │
│     │    channel: "whatsapp",          │                             │       │
│     │    identifier: "+370..." }       │                             │       │
│     │ ─────────────────────────────►   │                             │       │
│     │                                  │                             │       │
│     │  { challenge: { code: "X7K2M9",  │                             │       │
│     │      deep_link: "wa.me/..." } }  │                             │       │
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
│     │  GET /verifications/:id (poll)   │                             │       │
│     │ ─────────────────────────────►   │                             │       │
│     │  { status: "verified",           │                             │       │
│     │    proof: { token: "eyJ..." } }  │                             │       │
│     │ ◄─────────────────────────────   │                             │       │
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
| verifyProof | `POST /api/v1/proofs/validate` | None (rate-limited per IP) |
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

# 3. Poll for completion, or receive verification.completed at your account webhook URL
curl https://api.proof.holdings/api/v1/verifications/{id} \
  -H "Authorization: Bearer pk_live_..."

# 4. Verify proof offline
curl https://api.proof.holdings/.well-known/jwks.json
# Then: jwt.verify(proof_token, publicKey, {algorithms: ['ES256', 'RS256']})
```

---

---

## Proof of Delegation

Proof of Delegation is a signed statement that whoever controls a domain authorized a named artifact — nothing more. It says nothing about the artifact, its safety, or the business behind it.

The **principal** is a domain proven through Proof of Control, addressed by its `ph_ctl_*` handle. The **delegate** is a typed identifier — an `https` URL or a package URL (`purl`) — so the token points at exactly one package, container or endpoint. The delegation mints an ES256 token you publish alongside the artifact, and anyone can verify that token offline.

What a verified result establishes is bounded and worth stating plainly: **the domain you expected authorized this artifact** — not that the artifact is safe, audited or endorsed, and not that another domain has not published its own delegation over the same artifact. Any domain controller can mint a valid delegation naming someone else's package, so a verifier that does not pin the expected domain itself learns only that *some* domain made a claim.

Revoking is immediate at the API and reaches every verifier reading the published status list within a minute. Revoking one delegation leaves the domain proof and every sibling delegation intact; revoking the domain's control proof cascades to every delegation issued from it.

### Delegation Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  YOU                    PROOF HOLDINGS           ANY VERIFIER    │
│                                                                  │
│     │  prove the domain        │                      │          │
│     │ ──────────────────────►  │  ph_ctl_… handle     │          │
│     │ ◄──────────────────────  │                      │          │
│     │                          │                      │          │
│     │  POST /delegations       │                      │          │
│     │  { control_proof,        │                      │          │
│     │    delegate }            │                      │          │
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
- **Expiring** — A delegation can never outlive the control proof beneath it
- **Revocable** — Per delegation, without touching the domain proof or its siblings
- **Publishable** — The token is meant to be published on the artifact and verified offline

---

## Proof of Action (HITL)

Proof of Action is the human-in-the-loop (HITL) primitive. AI agents and automations request human approval before taking action. The approver gets a notification on Telegram or WhatsApp carrying a link made for them alone, reads the end-to-end encrypted request in their browser, and decides there; the decision is signed with the key they enrolled, and the record names whose key signed.

A confirmation is created `pending` and resolved later: `POST /api/v1/confirmations` never answers with the decision. Learn it from the `confirmation.approved` / `confirmation.denied` webhook or by polling `GET /api/v1/confirmations/:id`.

### HITL Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  AI AGENT              PROOF HOLDINGS              APPROVER      │
│                                                                  │
│     │  POST /confirmations     │                      │          │
│     │  { hitl_id, message }    │                      │          │
│     │ ──────────────────────►  │                      │          │
│     │  201 { status: "pending" }                      │          │
│     │ ◄──────────────────────  │                      │          │
│     │                          │  notification with   │          │
│     │                          │  their own link      │          │
│     │                          │ ──────────────────►  │          │
│     │                          │                      │          │
│     │                          │  decides in browser  │          │
│     │                          │  signed with own key │          │
│     │                          │ ◄──────────────────  │          │
│     │                          │                      │          │
│     │  confirmation.approved   │                      │          │
│     │  webhook, or poll        │                      │          │
│     │  GET /confirmations/:id: │                      │          │
│     │  { status: "approved",   │                      │          │
│     │    proof_token: "eyJ...",│                      │          │
│     │    response: {           │                      │          │
│     │      action: "approve",  │                      │          │
│     │      text: "Ship it!" } }│                      │          │
│     │ ◄──────────────────────  │                      │          │
│     │                          │                      │          │
│     │  ✓ Agent proceeds        │                      │          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Setup: four steps before the first request

A config only accepts approval requests once it is active and at least one approver has allowed it.
Skip a step and `POST /api/v1/confirmations` is refused: `404 hitl_not_found` (config not finalized),
`400 keypair_required` (no keys), or `403 no_authorized_channels` (nobody has allowed it yet).

**1. Create the config** — channels and timeout (`timeout_seconds`, 60–86400, default 3600). It starts `pending`:

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

**2. Upload the encryption keys.** Requests are end-to-end encrypted: generate an RSA-4096 key
pair and encrypt the private key with a password that never reaches us (the SDK helper
`generateEncryptionKeyPair` returns all three fields), then upload them:

```bash
PUT /api/v1/hitl/YOUR_HITL_CONFIG_ID/keys
{
  "public_key": "-----BEGIN PUBLIC KEY-----\n...",
  "encrypted_private_key": "...",
  "kdf_salt": "..."
}
```

**3. Finalize.** `POST /api/v1/hitl/YOUR_HITL_CONFIG_ID/finalize` makes the config `active` and
sends each approver channel a consent request.

**4. The approver allows it.** Each approver answers "Allow" on their own channel; that creates the
[authorization](#agent-contact-authorizations-consent) the request needs. Only channels with an active
authorization receive approval requests. `POST /api/v1/hitl/:id/authorize` sends the consent request
again.

### Agent Contact Authorizations (consent)

Before an agent may send approval requests to a person, that person has to allow it. That consent is an **authorization** — a scoped, revocable grant the user approves on their own channel. It lives here, with the approvals it gates, and not under Proof of Delegation: its scope union is `confirmations | identity_challenge`, so it serves this primitive and Proof of Me. (This documentation described it as the delegation primitive until 2026-08 — the two are separate objects on separate routes.)

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  YOUR APP               PROOF HOLDINGS              USER        │
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
| Upload encryption keys | `PUT /api/v1/hitl/:id/keys` | API Key |
| Activate (finalize) config | `POST /api/v1/hitl/:id/finalize` | API Key |
| Re-send consent requests | `POST /api/v1/hitl/:id/authorize` | API Key |
| Request approval | `POST /api/v1/confirmations` | API Key |
| Get confirmation | `GET /api/v1/confirmations/:id` | API Key |
| List confirmations | `GET /api/v1/confirmations` | API Key |

### Key Properties

- **Async, decided in the browser** — Telegram and WhatsApp deliver a content-free notification with the approver's own link; approving with buttons in the chat is not supported until a messenger client can hold the approver's key
- **Signed by whoever the link was sent to** — the approver's enrolled key, or the config key on the owner's link; a forwarded link alone cannot decide, and the decision record keeps the signer, key, signed bytes and signature
- **Configurable timeout** — Default 1 hour, max 24 hours; confirmation expires if no response
- **Comments** — Humans can add a comment with their approve/deny decision
- **Cryptographic proof** — Every decision produces an ES256-signed JWT proof token
- **First-response-wins** — Atomic resolution prevents race conditions
- **Webhooks** — `confirmation.approved` and `confirmation.denied` events fire on resolution; the body names its event in `event` (the same value as the `X-Proof-Event` header)

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

## Summary

Seven primitives — four live today, three coming soon. Offline-verifiable proofs for everything.

```
Proof of Control:    createChallenge → user action → verifyProof
Proof of Delegation: proven domain → createDelegation over url/purl → published token
Proof of Action:     approver allows → createConfirmation → human decides → proof token
Proof of Me:         enroll Circle members → challenge → cross-channel CONFIRM → proof token
Coming soon:         Agreement · Development · Enforcement
```

---

## Resources

- [API Reference](/docs/api) — Complete endpoint documentation
- [OpenAPI Spec](https://api.proof.holdings/api/openapi.json) — Machine-readable OpenAPI 3.1 specification
- [JWKS Public Keys](https://api.proof.holdings/.well-known/jwks.json) — For offline proof verification
- [Status](https://status.proof.holdings) — Live status of the API, the MCP server, the public keys for checking proofs and the website ([health check JSON](https://api.proof.holdings/health))
- [GitHub Docs](https://github.com/ProofHoldings/docs) — Source of truth for all documentation
- [Security & Threat Model](/docs/threat-model) — Guarantees, non-guarantees, and attack vectors
