---
title: Security
lastUpdated: "2026-02-05"
status: current
---

# proof.holdings Threat Model

> Security analysis for developers integrating proof.holdings

## TL;DR

proof.holdings verifies **control of digital assets** (phone, email, domain, wallet) without revealing identity. We issue signed proof tokens (RS256 JWT) that can be verified offline via JWKS.

**Key security properties:**
- Reverse OTP model (user sends to us, not vice versa)
- Constant-time challenge comparison (timing attack resistant)
- Asymmetric signatures (private key never leaves server)
- 13 threat categories actively mitigated

**What we don't do:** Identity verification, KYC, fraud prevention, permanent records.

---

## Table of Contents

- [What is proof.holdings?](#what-is-proofholdings)
- [Security Properties](#security-properties)
- [Threat Analysis](#threat-analysis)
  - [Threats We Protect Against](#threats-we-protect-against)
  - [Threats We Do NOT Protect Against](#threats-we-do-not-protect-against)
- [Explicit Non-Goals](#explicit-non-goals)
- [Security Recommendations for Integrators](#security-recommendations-for-integrators)
- [Cryptographic Details](#cryptographic-details)
- [Incident Response](#incident-response)
- [Version History](#version-history)

---

## What is proof.holdings?

proof.holdings is a **proof-of-control primitive** — it answers: "Does this person control this digital asset right now?"

It is NOT:
- An identity provider (we don't know WHO they are)
- A KYC solution (we don't verify identity documents)
- A fraud prevention system (we provide signals, not decisions)
- A permanent ownership record (proofs expire)

---

## Security Properties

### What proof.holdings Guarantees

| Property | How We Achieve It |
|----------|-------------------|
| **Authenticity** | RS256 asymmetric signatures — proofs can't be forged without private key |
| **Integrity** | JWT structure — any tampering invalidates signature |
| **Non-repudiation** | Signed by proof.holdings — we attest to the verification |
| **Time-boundedness** | `exp` claim — proofs expire, forcing re-verification |
| **Privacy** | Identifier hashed with SHA-256 — raw identifier not in token |
| **Offline verification** | JWKS public key — verify without API call |

### What proof.holdings Does NOT Guarantee

| Non-Guarantee | Why |
|---------------|-----|
| **Current control** | Proof attests to control at verification time, not now |
| **Identity** | Control ≠ identity. We verify control of an asset, not who you are |
| **Exclusive control** | Multiple people may control same asset |
| **Future control** | Assets can be transferred, numbers ported, domains sold |
| **Intent** | We verify control, not what user intends to do |

---

## Threat Analysis

### Threats We Protect Against

#### 1. SMS Interception (SS7 Attacks)
**Traditional OTP vulnerability:** Attacker intercepts SMS containing OTP code.

**Our mitigation:** Reverse OTP — user sends code TO us, not from us. Attacker would need to:
- Know the challenge code (displayed to user in your app UI)
- Have control of user's device to send message
- Complete within TTL window

**Residual risk:** If attacker has physical device access, they can complete verification.

#### 2. Phishing
**Traditional vulnerability:** User tricked into entering code on fake site.

**Our mitigation:** User-initiated flow — user sends message to our verified number/bot. There's no "enter code here" form to spoof.

**Residual risk:** Sophisticated phishing could instruct user to send code to attacker's number (social engineering).

#### 3. Credential Stuffing
**Traditional vulnerability:** Attacker uses leaked passwords.

**Our mitigation:** Passwordless — no credentials to stuff. Each verification requires current asset control.

**Residual risk:** None for this attack vector.

#### 4. Replay Attacks
**Traditional vulnerability:** Attacker reuses captured authentication.

**Our mitigation:**
- Challenge codes expire (phone: 10 min, email: 30 min, domain: 24 hours)
- Proofs have `exp` claim (default 30 days)
- Each verification generates unique proof token
- `iat` (issued at) prevents backdating

**Residual risk:** Valid proof can be replayed within its validity window — this is by design (proofs are meant to be portable).

#### 5. Timing Attacks
**Traditional vulnerability:** Attacker infers information from response timing.

**Our mitigation:**
- `crypto.timingSafeEqual()` for all challenge comparisons
- Constant-time operations regardless of match/mismatch
- No early-exit on partial matches

**Residual risk:** None for timing-based inference.

#### 6. Brute Force
**Traditional vulnerability:** Attacker guesses challenge codes.

**Our mitigation:**
- 6-character alphanumeric (36^6 = 2.1 billion combinations)
- Maximum 3 attempts per challenge
- Challenge expires after attempts exhausted
- Rate limiting per tenant and per IP
- Redis-backed tracking
- Exponential backoff between attempts (1s → 2s → 4s → 8s → 16s)

**Residual risk:** 3/2.1B = 0.00000014% chance per challenge window.

#### 7. Token Forgery
**Traditional vulnerability:** Attacker creates fake proofs.

**Our mitigation:**
- RS256 asymmetric signing
- Private key stored securely on server
- Public key only verifies, cannot sign
- Key ID (`kid`) enables rotation

**Residual risk:** Private key compromise would allow forgery — mitigated by server access controls and audit logging.

#### 8. Man-in-the-Middle
**Traditional vulnerability:** Attacker intercepts API communication.

**Our mitigation:**
- HTTPS only (HSTS with preload directive)
- Cloudflare proxy with origin IP lockdown
- Webhook signatures (HMAC-SHA256 with timestamp)

**Residual risk:** Cloudflare compromise would be catastrophic (affects most of internet).

#### 9. Denial of Service (Request Flooding)
**Traditional vulnerability:** Attacker sends oversized payloads to exhaust memory.

**Our mitigation:**
- Request body size limit (10KB max)
- Cloudflare DDoS protection
- Rate limiting per tenant and per IP

**Residual risk:** Distributed attacks at Cloudflare scale.

#### 10. Server-Side Request Forgery (SSRF)
**Traditional vulnerability:** Attacker tricks server into making requests to internal resources.

**Our mitigation:**
- URL validation blocks localhost, 127.0.0.1, private IP ranges
- Blocks .local, .internal, .localhost domains
- Webhook URLs validated before delivery

**Residual risk:** DNS rebinding attacks (mitigated by short TTL).

#### 11. Path Traversal & Probing
**Traditional vulnerability:** Attacker probes for sensitive files (.git, .env, admin panels).

**Our mitigation:**
- Nginx WAF blocklist (40+ attack patterns)
- Blocks: .git, .env, wp-admin, phpmyadmin, .sql, shell paths
- Auto-blocking of repeat offenders (IP escalation to firewall)

**Residual risk:** Zero-day paths not in blocklist.

#### 12. Session Hijacking
**Traditional vulnerability:** Attacker reuses stolen JWT tokens.

**Our mitigation:**
- JWT blacklist checked on every request (Redis-backed)
- Logout invalidates token immediately
- Short token expiry with refresh rotation

**Residual risk:** Token valid until next Redis check (milliseconds).

#### 13. Clickjacking & XSS
**Traditional vulnerability:** Attacker embeds site in iframe or injects scripts.

**Our mitigation:**
- Helmet.js security headers (X-Frame-Options: DENY, X-Content-Type-Options: nosniff)
- Strict CORS (production locked to https://proof.holdings)
- Content-Security-Policy headers

**Residual risk:** Browser without header support (rare, legacy).

---

### Threats We Do NOT Protect Against

#### 1. SIM Swap After Verification
**Scenario:** User verifies phone, then attacker SIM-swaps the number.

**Why we don't protect:** Proof attests to control at verification time. Subsequent SIM swap doesn't invalidate the proof — this is correct behavior (proof is a historical attestation).

**Mitigation for integrators:** Set short proof expiry, require re-verification for sensitive actions.

#### 2. Compromised User Device
**Scenario:** Attacker has malware on user's phone/computer.

**Why we don't protect:** If attacker controls the device, they can complete verification as if they were the user. No remote verification system can distinguish legitimate user from attacker with device access.

**Mitigation for integrators:** Defense in depth — combine with device fingerprinting, behavioral analysis.

#### 3. Social Engineering
**Scenario:** Attacker convinces user to send verification code.

**Why we don't protect:** We verify asset control, not user intent. If user voluntarily completes verification for attacker, we have no way to know.

**Mitigation for integrators:** Clear UI messaging about what verification means and its consequences.

#### 4. Insider Threat
**Scenario:** proof.holdings employee acts maliciously.

**Current mitigation:**
- Principle of least privilege
- Audit logging
- No access to raw credentials (bcrypt hashed)

**Future consideration:** Hardware security modules (HSM) could provide additional key protection.

#### 5. Collusion
**Scenario:** User and attacker work together to create "verified" accounts.

**Why we don't protect:** If user legitimately controls asset and chooses to help attacker, verification is technically valid.

**Mitigation for integrators:** This is a business logic problem, not a verification problem.

---

## Explicit Non-Goals

### We Are NOT Building:

1. **Identity Verification**
   - We prove control, not identity
   - Same phone can be controlled by multiple people
   - Control of asset says nothing about who you are

2. **KYC/AML Solution**
   - We don't collect identity documents
   - We don't verify legal identity
   - We're a primitive, not a compliance solution

3. **Fraud Prevention**
   - We provide signals ("this person controls this phone")
   - Fraud decisions are your responsibility
   - A verified phone number can still be used for fraud

4. **Permanent Ownership Record**
   - Proofs expire by design
   - Assets change hands
   - Re-verification is expected and encouraged

5. **Real-time Control Monitoring**
   - We verify control at a point in time
   - We don't monitor ongoing control
   - Asset transfer after verification is not detected

---

## Security Recommendations for Integrators

### Proof Validation Checklist

```
1. ✓ Verify signature using JWKS public key
2. ✓ Check `exp` claim (not expired)
3. ✓ Check `iss` claim equals "proof.holdings"
4. ✓ Check revocation list (GET /api/v1/proofs/revoked)
5. ✓ Verify `type` and `channel` match your requirements
6. ✓ For sensitive actions: require fresh proof (check `iat`)
```

### Expiry Recommendations

| Use Case | Recommended Expiry |
|----------|-------------------|
| One-time verification | 1 hour |
| Session authentication | 7 days |
| Account linking | 30 days |
| Compliance records | 90 days |
| Low-risk preferences | 365 days |

### Defense in Depth

Don't rely solely on proof.holdings. Combine with:
- Device fingerprinting
- Behavioral analysis
- IP reputation
- Rate limiting on your end
- Step-up authentication for sensitive actions

---

## Cryptographic Details

### Challenge Generation
```
Algorithm: crypto.randomBytes(12) with modulo reduction
Alphabet: A-Z, 0-9 (36 characters)
Length: 6 characters
Entropy: log2(36^6) = 31.0 bits
Comparison: crypto.timingSafeEqual()
```

### Proof Token Signing
```
Algorithm: RS256 (RSA with SHA-256)
Key size: 2048 bits
Key distribution: JWKS (/.well-known/jwks.json)
Token format: JWT (RFC 7519)
Note: Production uses RS256. HS256 fallback exists for development only.
```

### Identifier Hashing
```
Algorithm: SHA-256
Input: Normalized identifier (E.164 for phone, lowercase for email)
Output: 64-character hex string
Purpose: Privacy — raw identifier not in token
```

### BYOC Credential Encryption
```
Algorithm: AES-256-GCM
Key derivation: PBKDF2 with 100,000 iterations
Purpose: Encrypt bring-your-own-credentials (Twilio, SendGrid, etc.) at rest
Note: Each tenant's credentials encrypted with unique derived key
```

### Wallet Address Validation
```
Ethereum: 0x + 40 hex characters (checksum validated)
Solana: Base58, 32-44 characters
Bitcoin: Legacy (1...), P2SH (3...), Bech32 (bc1...)
Purpose: Reject malformed addresses before verification attempt
```

### Webhook Signatures
```
Algorithm: HMAC-SHA256
Payload: timestamp.body
Header: X-Proof-Signature
Validation: Compare within 5-minute window
Note: Production warns if webhook secret not configured — receivers cannot verify authenticity without it.
```

---

## Incident Response

### If You Suspect Compromise

1. **Revoke affected proofs** — `POST /api/v1/proofs/:id/revoke`
2. **Check revocation list** — `GET /api/v1/proofs/revoked`
3. **Contact us** — hello@proof.holdings
4. **Require re-verification** — Don't trust existing proofs

### Our Commitment

- Security issues disclosed via hello@proof.holdings
- Critical vulnerabilities: Immediate key rotation capability
- Transparency: Public disclosure after fix (coordinated)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.2 | 2026-02-04 | Added: SSRF protection, WAF blocklist, session hijacking, clickjacking/XSS, BYOC encryption, wallet validation, TL;DR, TOC, related docs |
| 1.1 | 2026-02-04 | Added: exponential backoff, request size limits, DoS protection, HSTS preload |
| 1.0 | 2026-02-04 | Initial threat model |

---

*This document is intended for developers integrating proof.holdings. For security researchers, please report vulnerabilities to hello@proof.holdings.*

---

## Related Documentation

- [API Reference](./API.md) — Endpoints, authentication, webhooks
- [Primitives](./PRIMITIVES.md) — Core mental model
- [Pricing](./PRICING.md) — Plans and verification limits
- [Test Vectors](./test-vectors.json) — Golden test cases for SDK authors
