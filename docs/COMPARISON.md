---
title: Comparison
lastUpdated: "2026-10-26"
status: current
---

# Verification Methods Compared

> How Proof Holdings compares to alternatives

---

## Quick Comparison

| Method | Who Initiates | Cost per Verification | Offline Verify | Replay Protection | Asset Types |
|--------|---------------|----------------------|----------------|-------------------|-------------|
| **A2P SMS OTP** | Server | $0.0457–$0.3868 per SMS abroad (Twilio, October 2026) | ❌ | ⚠️ Time-limited | Phone only |
| **TOTP (Authenticator)** | User | Free | ✅ | ✅ | None (2FA only) |
| **WebAuthn/Passkeys** | User | Free | ✅ | ✅ | Device-bound |
| **Email Magic Link** | Server | — | ❌ | ✅ | Email only |
| **Wallet Signature** | User | Free | ✅ | ✅ | Wallet only |
| **Proof Holdings** | User | from €0.0079 | ✅ | ✅ | Phone, Email, Domain (social and wallet coming soon) |

---

## Detailed Breakdown

### A2P SMS OTP (Twilio, Vonage, etc.)

**How it works:**
```
Server sends SMS → User receives code → User enters code → Verified
```

**Pros:**
- Works on any phone that receives SMS

**Cons:**
- Costly abroad: sending one SMS to the ten countries we compared costs $0.0457–$0.3868 ([Twilio's published rates](https://assets.cdn.prod.twilio.com/pricing-csv/SMSPricing.csv), October 2026)
- Codes can fail to arrive: [Microsoft has reported SMS delivery as low as 50% in some regions](https://techcommunity.microsoft.com/blog/microsoft-entra-blog/its-time-to-hang-up-on-phone-transports-for-authentication/1751752), and when India changed its SMS filtering rules in March 2021, [OTP failure at leading banks went above 25%](https://www.business-standard.com/article/economy-policy/sms-scrubbing-faulty-implementation-wipes-out-otps-40-traffic-disrupted-121030900202_1.html)
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
- Phishing-resistant
- Free

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
- Works with any email address

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
- Wallet addresses only

**When to use:** Web3 applications, crypto-native users

---

### Proof Holdings (Reverse OTP)

Reverse OTP: the person sends the code to us from the phone or account being verified, instead of typing a code we sent.

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
Auto-DNS supports 54 DNS providers for automatic record creation.

**Pros:**
- You pay per proof, not per message sent
- Offline-verifiable proofs (ES256, checked with our public key)
- Multi-asset (phone, email, domain; social and wallet coming soon)

**When to use:**
- Asset verification without identity (phone, email, domain)
- Domain ownership verification

---

## Cost Comparison

### 10,000 verifications/month

| Provider | Method | Cost |
|----------|--------|------|
| Twilio SMS | A2P SMS, sent abroad | $457–$3,868 |
| **Proof Holdings** | Reverse OTP | **€79** (the €79 plan includes 10,000 proofs) |

### At scale (100,000/month)

| Provider | Method | Cost |
|----------|--------|------|
| Twilio SMS | A2P SMS, sent abroad | $4,570–$38,680 |
| **Proof Holdings** | Reverse OTP | **Custom pricing** (Enterprise) |

SMS figures: $0.0457–$0.3868 per message, the range of [Twilio's published rates](https://assets.cdn.prod.twilio.com/pricing-csv/SMSPricing.csv), October 2026, to send one SMS to Germany, France, the Netherlands, Lithuania, Poland, the United Kingdom, the UAE, India, Nigeria and Brazil, multiplied by the volume.

**€79 a month for 10,000 proofs; sending 10,000 SMS abroad through Twilio costs $457–$3,868.**

---

## Security Comparison

| Threat | SMS OTP | TOTP | WebAuthn | Proof Holdings |
|--------|---------|------|----------|----------------|
| **Resists fake code-entry pages** | ❌ No | ❌ No | ✅ Yes | ✅ Phone; ⚠️ email sends a code |
| **SIM Swap** | ⚠️ Depends on the carrier† | ✅ Safe | ✅ Safe | ⚠️ Partial* |
| **SS7 Attack** | ⚠️ Depends on the carrier† | ✅ Safe | ✅ Safe | ✅ Telegram, WhatsApp; ⚠️ SMS‡ |
| **MITM** | ⚠️ Vulnerable | ⚠️ Vulnerable | ✅ Safe | ⚠️ A phisher can relay the code§ |
| **Replay** | ⚠️ Time-window | ✅ Safe | ✅ Safe | ✅ Safe |

*On Telegram and WhatsApp a SIM swap alone is not enough: the attacker must also take over the messaging account, which two-step verification blocks when the user has turned it on. On plain SMS, whoever holds the SIM can send the message. A proof attests control at verification time, so a later SIM swap doesn't invalidate it (correct behavior: a proof is historical).

†Whether a SIM swap or an SS7 interception gets the code depends on the carrier's own controls: how it checks a request to move a number to a new SIM, and whether it filters SS7 signalling.

‡Telegram and WhatsApp messages do not travel over SS7. A text message's sender number can be spoofed: we check each inbound SMS against carrier data and record the result in the proof's `confidence` claim — see the [threat model](/docs/threat-model).

§A phisher can ask someone to send their code to our number for a verification the phisher started; no channel that sends a code can rule this out. WebAuthn can, because the browser binds the credential to the site.

For phone and messaging verification, no code travels to the person, so there is no code to steal from a fake 'enter your code' page. A phisher can still ask someone to send a code to the phisher's own number — see the [threat model](/docs/threat-model#2-phishing).

---

## Feature Comparison

| Feature | SMS OTP | TOTP | WebAuthn | Proof Holdings |
|---------|---------|------|----------|----------------|
| **Offline verification** | ❌ | ✅ | ✅ | ✅ |
| **Portable proofs** | ❌ | ❌ | ❌ | ✅ |
| **Multi-asset** | ❌ | ❌ | ❌ | ✅ |
| **No pre-registration** | ✅ | ❌ | ❌ | ✅ |
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

### Use WebAuthn when:
- Modern device requirement OK
- Primary authentication

### Use Proof Holdings when:
- Need proof of asset control (not just auth)
- Cost matters at scale
- Offline verification needed
- Multi-asset verification (phone, email, domain)
- Proofs need to be portable

---

## The Proof Holdings Difference

Traditional verification answers: "Did they receive our message?"

Proof Holdings answers: "Can they prove control of this asset?"

```
Traditional:                    Proof Holdings:

  Server → User                   User → Server
  (push)                          (pull)

  "Trust us, we sent it"          "Here's cryptographic proof"

  Requires API call               Works offline
  to validate                     with public key
```

**The result:** Portable, offline-verifiable proofs of asset control.

---

## Migration Path

### From SMS OTP to Proof Holdings

```javascript
// Before (Twilio)
await twilio.messages.create({
  to: userPhone,
  body: `Your code is ${code}`
});

// After (Proof Holdings)
const verification = await fetch('https://api.proof.holdings/api/v1/verifications', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer pk_live_...',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    type: 'phone',
    channel: 'whatsapp', // or 'telegram', 'sms'
    identifier: userPhone
  })
});

// Show user: "Send {code} via WhatsApp"
// Receive verification.completed at your account webhook URL, or poll GET /api/v1/verifications/:id
// Receive signed proof token
```

**Key difference:** User sends TO you, not you sending TO user.

---

### Email Verification

```javascript
const verification = await fetch('https://api.proof.holdings/api/v1/verifications', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer pk_live_...',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    type: 'email',
    channel: 'email',
    identifier: userEmail
  })
});

// User receives OTP + magic link
// Receive verification.completed at your account webhook URL, or poll GET /api/v1/verifications/:id
// Receive signed proof token (offline-verifiable)
```

**Result:** Portable, signed proof of email control — not just a one-time confirmation.

---

### Domain Verification

```javascript
const verification = await fetch('https://api.proof.holdings/api/v1/verifications', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer pk_live_...',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    type: 'domain',
    channel: 'dns',        // the user adds a TXT record
    identifier: 'example.com'
  })
});

// channel options: 'dns' (manual TXT), 'http' (file upload), 'email' (admin email),
// 'auto' (record created through the DNS provider's API — send `dns_provider` credentials; 54 providers supported)
// Once the record or file is in place, POST /api/v1/verifications/:id/verify to run the check
// ('email' instead takes the mailed code at POST /api/v1/verifications/:id/submit);
// then receive verification.completed at your account webhook URL, or read GET /api/v1/verifications/:id
// Receive signed proof token
```

**Result:** Auto-DNS handles record creation for 54 providers. Four verification methods to fit any workflow.

---

## Summary

| If you need... | Use... |
|----------------|--------|
| Cost-effective phone verification | Proof Holdings |
| Email verification with portable proof | Proof Holdings |
| Domain ownership verification | Proof Holdings |
| Offline-verifiable proofs | Proof Holdings |
| Multi-asset verification | Proof Holdings |
| Legacy outbound SMS | Twilio/SMS |
| 2FA add-on | TOTP |
| Strongest device auth | WebAuthn |

---

*Questions? hello@proof.holdings*
