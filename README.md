# proof.holdings - Multi-Surface Proof of Control

**Cryptographic proofs of digital asset control.**

proof.holdings issues signed JWT proofs that a user controls a digital asset (phone, email, domain, social, wallet) by completing a challenge from that asset. Proofs are offline-verifiable using RS256 public keys.

[![API Status](https://img.shields.io/badge/API-Live-brightgreen)](https://api.proof.holdings/health)
[![JWKS](https://img.shields.io/badge/JWKS-RS256-blue)](https://api.proof.holdings/.well-known/jwks.json)
[![OpenAPI](https://img.shields.io/badge/OpenAPI-3.0-6BA539)](https://api.proof.holdings/api/docs)

---
## The Abstraction

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ASSET ──────► CHALLENGE ──────► USER ACTION ──────► PROOF               │
│                                                                             │
│    "phone"       "Send X7K2M9     User sends         Signed JWT            │
│    "+370..."      via WhatsApp"   the message        (offline-verifiable)  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Three primitives. Four steps. That's it.**

---

## Quick Start

```bash
# 1. Create verification challenge
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{"type":"phone","channel":"whatsapp","identifier":"+37069199199"}'

# Response: { "id": "...", "challenge": { "code": "X7K2M9", "deep_link": "wa.me/..." } }

# 2. User sends challenge code via WhatsApp (or clicks deep_link)

# 3. Poll for completion (or use webhook)
curl https://api.proof.holdings/api/v1/verifications/{id} \
  -H "Authorization: Bearer pk_live_..."

# Response includes: { "status": "verified", "proof": { "token": "eyJ..." } }

# 4. Verify proof offline
curl https://api.proof.holdings/.well-known/jwks.json
# Then: jwt.verify(proof_token, publicKey, { algorithms: ['RS256'] })
```

> **Sandbox:** Use `pk_test_` keys to test without sending real messages or consuming quota. Auto-complete verifications with `POST /api/v1/verifications/{id}/test-verify`. See [Test Mode](docs/API.md#test-mode-sandbox).

---

## Three Primitives

| Primitive | Endpoint | Purpose |
|-----------|----------|---------|
| `createChallenge` | `POST /api/v1/verifications` | Generate challenge for user |
| `verifyProof` | `POST /api/v1/proofs/validate` or offline via JWKS | Validate proof token |
| `revokeProof` | `POST /api/v1/proofs/:id/revoke` | Invalidate a proof |

---

## Live Channels

| Type | Channels | Status |
|------|----------|--------|
| **Phone** | WhatsApp, Telegram, Viber, SMS | ✅ Live |
| **Email** | OTP + Magic Link | ✅ Live |
| **Domain** | DNS, HTTP, Email, Auto-DNS (55 providers) | ✅ Live |
| **Social** | GitHub, Google, X, etc. | Coming Soon |
| **Wallet** | Ethereum, Solana, Bitcoin | Coming Soon |

---

## Design Principles

| Principle | What it means |
|-----------|---------------|
| **Determinism** | Same input = same output. No hidden state. |
| **Primitives** | Three functions. Compose them as needed. |
| **Testability** | Test vectors provided. Golden cases for SDKs. |
| **Composability** | Proofs are portable. Use across systems. |
| **Clarity** | No magic. Explicit behavior. Read the code. |

---

## Machine-Readable Endpoints

| Endpoint | Purpose | Cache |
|----------|---------|-------|
| `GET /.well-known/jwks.json` | RS256 public keys for offline verification | 24h |
| `GET /api/v1/proofs/revoked` | Signed revocation list | 5min |
| `GET /health` | Service status | No cache |
| `GET /api/openapi.json` | OpenAPI 3.0 specification | 1h |

---

## Documentation

| Document | Description |
|----------|-------------|
| [**PRIMITIVES.md**](docs/PRIMITIVES.md) | Core mental model in one page |
| [**API.md**](docs/API.md) | Complete API reference |
| [**SMART_REUSE.md**](docs/SMART_REUSE.md) | Skip re-verification with existing proofs |
| [**PROFILES.md**](docs/PROFILES.md) | Multiple public profiles per account |
| [**PROJECTS.md**](docs/PROJECTS.md) | Custom branding and message templates |
| [**THREAT_MODEL.md**](docs/THREAT_MODEL.md) | Security guarantees and non-goals |
| [**COMPARISON.md**](docs/COMPARISON.md) | vs SMS OTP, TOTP, WebAuthn |
| [**PRICING.md**](docs/PRICING.md) | Pricing tiers |
| [**openapi.json**](docs/openapi.json) | OpenAPI 3.0 spec for SDK generation |
| [**test-vectors.json**](docs/test-vectors.json) | Golden test cases for SDK authors |

---

## DNS Provider Integrations

Auto-DNS verification supports **55 providers**. [Full list →](integrations/dns-providers/README.md)

**Global Leaders:** Cloudflare, GoDaddy, Route53, Google Cloud, Azure, DigitalOcean

**Major Registrars:** Namecheap, NameSilo, Porkbun, Name.com, IONOS, OVH, Gandi

**EU Regional:** Hetzner, TransIP, INWX, Netcup, Infomaniak, Domeneshop, Scaleway

[See all 55 providers →](integrations/dns-providers/README.md)

---

## Why proof.holdings?

**Traditional SMS verification:**
```
Server sends SMS → User receives code → User enters code → Verified
     ($0.05-0.40)     (can be phished)     (replay window)
```

**proof.holdings (Reverse OTP):**
```
Server shows code → User sends TO server → Verified → Signed proof
     (free)           (user-initiated)      (offline-verifiable)
```

**Result:** €79/month for what costs $1,000+ with SMS.

---

## Links

- **Website:** https://proof.holdings
- **API:** https://api.proof.holdings
- **Docs:** https://proof.holdings/docs
- **Status:** https://api.proof.holdings/health
- **API Explorer:** https://api.proof.holdings/api/docs

---

## License

Proprietary. Contact hello@proof.holdings for licensing.
