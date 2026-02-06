---
title: Comparison
lastUpdated: "2026-02-05"
status: current
---

# Verification Methods Compared

> How proof.holdings compares to alternatives

---

## Quick Comparison

| Method | Who Initiates | Cost per Verification | Offline Verify | Replay Protection | Asset Types |
|--------|---------------|----------------------|----------------|-------------------|-------------|
| **A2P SMS OTP** | Server | $0.05-0.40 | ❌ | ⚠️ Time-limited | Phone only |
| **TOTP (Authenticator)** | User | Free | ✅ | ✅ | None (2FA only) |
| **WebAuthn/Passkeys** | User | Free | ✅ | ✅ | Device-bound |
| **Email Magic Link** | Server | $0.001-0.01 | ❌ | ✅ | Email only |
| **Wallet Signature** | User | Free | ✅ | ✅ | Wallet only |
| **proof.holdings** | User | €0.0079-0.03 | ✅ | ✅ | Phone, Email, Domain, Social, Wallet |

---

## Detailed Breakdown

### A2P SMS OTP (Twilio, Vonage, etc.)

**How it works:**
```
Server sends SMS → User receives code → User enters code → Verified
```

**Pros:**
- Universal (everyone has SMS)
- Familiar UX

**Cons:**
- Expensive ($0.05-0.40 per message)
- 10-30% delivery failure rates
- Vulnerable to SIM swap, SS7 attacks
- Server-initiated (can be phished)
- No offline verification

**When to use:** Legacy systems requiring outbound SMS

---

### TOTP (Google Authenticator, Authy)

**How it works:**
```
Shared secret → App generates code → User enters code → Verified
```

**Pros:**
- Free
- Offline
- Strong replay protection

**Cons:**
- Requires app installation
- Secret backup complexity
- Not asset verification (just 2FA)
- Device-bound (lost phone = locked out)

**When to use:** Second factor authentication

---

### WebAuthn / Passkeys

**How it works:**
```
Device generates keypair → Challenge-response → Verified
```

**Pros:**
- Strongest security
- Phishing-resistant
- Free
- Great UX (biometrics)

**Cons:**
- Device-bound
- Not asset verification
- Recovery complexity
- Requires compatible device

**When to use:** Primary authentication on modern devices

---

### Email Magic Link

**How it works:**
```
Server sends email → User clicks link → Verified
```

**Pros:**
- Simple UX
- Low cost (~$0.001)
- Universal

**Cons:**
- Email delivery delays
- Phishing vulnerability
- Server-initiated
- Email-only

**When to use:** Low-security email verification

---

### Wallet Signature (Sign-In with Ethereum, etc.)

**How it works:**
```
Challenge message → Wallet signs → Signature verified → Verified
```

**Pros:**
- Free
- Offline-verifiable
- User-initiated
- Cryptographically strong

**Cons:**
- Requires wallet
- Web3 UX complexity
- Wallet addresses only

**When to use:** Web3 applications, crypto-native users

---

### proof.holdings (Reverse OTP)

**How it works (phone):**
```
Server shows code → User sends code via WhatsApp/Telegram/SMS → Verified → Signed proof
```

**How it works (email):**
```
Server creates challenge → User receives OTP + magic link → User clicks or enters code → Signed proof
```

**How it works (domain — 4 methods):**
```
Auto-DNS:  Server detects DNS provider → User authorizes via OAuth → Record created automatically → Signed proof
DNS:       Server creates challenge → User adds TXT record manually → Server verifies → Signed proof
HTTP:      Server creates challenge → User uploads file to /.well-known/ → Server verifies → Signed proof
Email:     Server creates challenge → Sent to admin@domain → User confirms → Signed proof
```
Auto-DNS supports 55 DNS providers for automatic record creation.

**Pros:**
- User-initiated (secure by design)
- Low cost (no per-message fees — you pay per proof, not per SMS)
- Offline-verifiable proofs (RS256)
- Multi-asset (phone, email, domain, social, wallet)
- Portable proofs (reuse across systems)
- No phone number collection upfront
- Domain verification supports Auto-DNS with 53 providers

**When to use:**
- Passwordless authentication
- Asset verification without identity (phone, email, domain)
- Multi-factor with control proof
- Cross-system verification
- Domain ownership verification for SaaS onboarding
- Design your own verification flows — full API freedom, no rigid templates

---

## Cost Comparison

### 10,000 verifications/month

| Provider | Method | Cost |
|----------|--------|------|
| Twilio SMS | A2P SMS | $500-4,000 |
| AWS SNS | A2P SMS | $300-800 |
| Auth0 | SMS OTP | $230+ (plus base) |
| **proof.holdings** | Reverse OTP | **€79** (Business plan) |

### At scale (100,000/month)

| Provider | Method | Cost |
|----------|--------|------|
| Twilio SMS | A2P SMS | $5,000-40,000 |
| **proof.holdings** | Reverse OTP | **Custom pricing** (Enterprise) |

**€79/month for what costs $1,000+ with SMS.**

---

## Security Comparison

| Threat | SMS OTP | TOTP | WebAuthn | proof.holdings |
|--------|---------|------|----------|----------------|
| **Phishing** | ⚠️ Vulnerable | ⚠️ Vulnerable | ✅ Resistant | ✅ Resistant |
| **SIM Swap** | ❌ Vulnerable | ✅ Safe | ✅ Safe | ✅ Safe* |
| **SS7 Attack** | ❌ Vulnerable | ✅ Safe | ✅ Safe | ✅ Safe |
| **MITM** | ⚠️ Vulnerable | ⚠️ Vulnerable | ✅ Safe | ✅ Safe |
| **Replay** | ⚠️ Time-window | ✅ Safe | ✅ Safe | ✅ Safe |

*proof.holdings verifies control at verification time. Subsequent SIM swap doesn't invalidate the proof (this is correct behavior - proof is historical).

---

## Feature Comparison

| Feature | SMS OTP | TOTP | WebAuthn | proof.holdings |
|---------|---------|------|----------|----------------|
| **Offline verification** | ❌ | ✅ | ✅ | ✅ |
| **Portable proofs** | ❌ | ❌ | ❌ | ✅ |
| **Multi-asset** | ❌ | ❌ | ❌ | ✅ |
| **No pre-registration** | ❌ | ❌ | ❌ | ✅ |
| **API-first** | ✅ | ⚠️ | ⚠️ | ✅ |
| **Webhooks** | ⚠️ | ❌ | ❌ | ✅ |
| **Audit trail** | ⚠️ | ❌ | ❌ | ✅ |

---

## When to Use What

### Use SMS OTP when:
- You need to send outbound SMS
- Users don't have smartphones
- Legacy system integration

### Use TOTP when:
- Adding 2FA to existing auth
- Users willing to install app
- Highest security needed

### Use WebAuthn when:
- Modern device requirement OK
- Primary authentication
- Best UX priority

### Use proof.holdings when:
- Need proof of asset control (not just auth)
- Cost matters at scale
- Offline verification needed
- Multi-asset verification (phone, email, domain)
- Proofs need to be portable
- Don't want to collect phone numbers upfront
- You want full freedom to design your own verification flows
- Easy, secure setup without rigid templates or vendor lock-in

---

## The proof.holdings Difference

Traditional verification answers: "Did they receive our message?"

proof.holdings answers: "Can they prove control of this asset?"

```
Traditional:                    proof.holdings:

  Server → User                   User → Server
  (push)                          (pull)

  "Trust us, we sent it"          "Here's cryptographic proof"

  Requires API call               Works offline
  to validate                     with public key
```

**The result:** Portable, offline-verifiable proofs of asset control.

---

## Migration Path

### From SMS OTP to proof.holdings

```javascript
// Before (Twilio)
await twilio.messages.create({
  to: userPhone,
  body: `Your code is ${code}`
});

// After (proof.holdings)
const verification = await fetch('https://api.proof.holdings/api/v1/verifications', {
  method: 'POST',
  headers: { 'Authorization': 'Bearer pk_live_...' },
  body: JSON.stringify({
    type: 'phone',
    channel: 'whatsapp', // or 'telegram', 'sms'
    identifier: userPhone
  })
});

// Show user: "Send {code} via WhatsApp"
// Wait for webhook or poll for completion
// Receive signed proof token
```

**Key difference:** User sends TO you, not you sending TO user.

---

### Email Verification

```javascript
const verification = await fetch('https://api.proof.holdings/api/v1/verifications', {
  method: 'POST',
  headers: { 'Authorization': 'Bearer pk_live_...' },
  body: JSON.stringify({
    type: 'email',
    channel: 'email',
    identifier: userEmail
  })
});

// User receives OTP + magic link
// Wait for webhook or poll for completion
// Receive signed proof token (offline-verifiable)
```

**Result:** Portable, signed proof of email control — not just a one-time confirmation.

---

### Domain Verification

```javascript
const verification = await fetch('https://api.proof.holdings/api/v1/verifications', {
  method: 'POST',
  headers: { 'Authorization': 'Bearer pk_live_...' },
  body: JSON.stringify({
    type: 'domain',
    channel: 'auto',       // auto-detect DNS provider (53 supported)
    identifier: 'example.com'
  })
});

// channel options: 'auto' (auto-DNS), 'dns' (manual TXT), 'http' (file upload), 'email' (admin email)
// Auto-DNS: user authorizes via DNS provider OAuth → record created automatically
// Wait for webhook or poll for completion
// Receive signed proof token
```

**Result:** Auto-DNS handles record creation for 53 providers. Four verification methods to fit any workflow.

---

## Summary

| If you need... | Use... |
|----------------|--------|
| Cost-effective phone verification | proof.holdings |
| Email verification with portable proof | proof.holdings |
| Domain ownership verification | proof.holdings |
| Offline-verifiable proofs | proof.holdings |
| Multi-asset verification | proof.holdings |
| Easy and secure setup | proof.holdings |
| Legacy outbound SMS | Twilio/SMS |
| 2FA add-on | TOTP |
| Strongest device auth | WebAuthn |

---

*Questions? hello@proof.holdings*
