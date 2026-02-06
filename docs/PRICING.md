---
title: Pricing
lastUpdated: "2026-02-05"
status: current
---

# proof.holdings Pricing

> Simple, transparent pricing. Pay for proofs, not messages.

---

## The Value Proposition

**€79/month for what costs $1,000+ with SMS.**

Traditional SMS verification costs $0.05-0.40 per message. At 10,000 verifications:
- Twilio SMS: $500-4,000/month
- proof.holdings: **€79/month**

And you get more: offline-verifiable proofs, multi-channel support, webhooks, audit trails.

---

## Plans

### Pay As You Go
**€0.03/proof** (1,000 proofs free every month)

- No monthly fee
- No commitment
- All verification types (phone, email, domain)
- Unlimited API calls
- Custom branding
- Test mode included
- Email support

*Perfect for: Testing, side projects, variable usage, early-stage startups*

---

### Pro
**€29/month**

- 3,000 proofs included
- €0.025 per additional proof
- **€0.0097/proof effective rate**
- All verification types
- Custom branding
- Webhooks
- Priority support

*Perfect for: Growing apps, small businesses*

---

### Business
**€79/month**

- 10,000 proofs included
- €0.02 per additional proof
- **€0.0079/proof effective rate**
- All verification types
- Custom branding
- Webhooks
- Priority support

*Perfect for: Scale-ups, SaaS platforms*

---

### White-Label
**€190/month**

- 10,000 proofs included
- €0.015 per additional proof
- **€0.019/proof effective rate**
- All verification types
- Webhooks
- Custom branding
- **BYOC (your own Telegram bot, WhatsApp, Viber, Email, SMS)**
- Priority support

*Perfect for: Platforms requiring complete white-label experience*

---

### Enterprise
**Custom pricing**

- 100,000+ proofs/month
- Volume discounts
- Custom SLA
- Dedicated infrastructure

*Contact: hello@proof.holdings*

---

## Custom Branding (All Plans)

**All plans** include custom branding on shared infrastructure:
- `@proof_holdings_bot` on Telegram
- Shared WhatsApp and Viber numbers
- Emails from proof.holdings

**Custom branding** lets users see YOUR context in messages:

| Element | Shared | Custom Branding |
|---------|--------|-----------------|
| Business name | "Proof of Holdings" | "Your Company Name" |
| Email logo | proof.holdings logo | Your logo |
| Email colors | Default blue | Your brand colors |
| Support email | hello@proof.holdings | support@yourcompany.com |
| Message templates | Default text | Custom text |

---

## BYOC - Bring Your Own Channels (White-Label)

Full channel ownership - users interact with YOUR endpoints:

| Channel | Shared | BYOC |
|---------|--------|------|
| Telegram | `@proof_holdings_bot` | `@your_company_bot` |
| WhatsApp | Shared number | Your WhatsApp Business number |
| Viber | Shared number | Your Viber Business number |
| Email | verify@verify.proof.holdings | noreply@yourcompany.com |
| SMS | Shared number | Your phone number |

BYOC = complete white-label experience. Users never see proof.holdings.

---

## Cost Comparison

### Per-Verification Cost

| Volume | PAYG | Pro | Business | White-Label | SMS (Twilio) |
|--------|------|-----|----------|-------------|--------------|
| 1,000 | €0 (free) | €29 | €79 | €190 | $50-400 |
| 3,000 | €60 | €29 | €79 | €190 | $150-1,200 |
| 10,000 | €270 | €204 | €79 | €190 | $500-4,000 |
| 50,000 | €1,470* | €1,204* | €879* | €790* | $2,500-20,000 |

*50K calculation: Base price + (volume - included) × overage rate

### Effective Rate at Volume

| Plan | Base Price | Included | Rate After | Effective at 10K |
|------|------------|----------|------------|------------------|
| PAYG | €0 | 1,000 | €0.03 | €0.027 |
| Pro | €29 | 3,000 | €0.025 | €0.0204 |
| Business | €79 | 10,000 | €0.02 | **€0.0079** |
| White-Label | €190 | 10,000 | €0.015 | €0.019 |
| Enterprise | Custom | 100,000+ | Volume discount | Contact us |

---

## What's a Proof?

**1 proof = 1 verified asset**

- ✅ Phone number verified via Telegram → 1 proof
- ✅ Phone number verified via WhatsApp → 1 proof
- ✅ Phone number verified via Viber → 1 proof
- ✅ Phone number verified via SMS → 1 proof
- ✅ Email verified via magic link → 1 proof
- ✅ Domain verified via DNS → 1 proof
- ✅ Social account verified via OAuth → 1 proof
- ✅ Wallet verified via signature → 1 proof

**What's NOT counted:**
- ❌ Failed verification attempts
- ❌ Expired challenges (not completed)
- ❌ API calls (unlimited)
- ❌ Webhook deliveries (unlimited)
- ❌ Proof validations (unlimited, offline)

---

## All Plans Include

- **All verification types** - Phone, email, domain, social, wallet
- **All channels** - Telegram, WhatsApp, Viber, SMS, Email, DNS, HTTP
- **Unlimited API calls** - No rate limits on reads
- **Unlimited proof validation** - Verify offline with JWKS
- **Test mode** - Full functionality, no charges
- **Webhooks** - Real-time notifications
- **Dashboard** - Manage everything in one place
- **Documentation** - Full API docs, examples

---

## Features by Plan

| Feature | PAYG | Pro | Business | White-Label |
|---------|------|-----|----------|-------------|
| All channels (Telegram, WhatsApp, Viber, SMS, Email, DNS, HTTP) | ✅ | ✅ | ✅ | ✅ |
| Custom branding (business name, logo, colors) | ✅ | ✅ | ✅ | ✅ |
| Custom message templates | ✅ | ✅ | ✅ | ✅ |
| Webhooks | ✅ | ✅ | ✅ | ✅ |
| BYOC (your own Telegram, WhatsApp, Viber, Email, SMS) | ❌ | ❌ | ❌ | ✅ |
| Priority support | ❌ | ❌ | ✅ | ✅ |

---

## FAQ

### How does billing work?
- Monthly billing, charged at start of cycle
- Overage charged at end of cycle
- No long-term contracts

### Can I change plans?
- Upgrade anytime, prorated
- Downgrade at end of billing cycle
- No penalties

### Do unused proofs roll over?
- No, included proofs reset monthly
- PAYG overage has no expiration

### Is there a free trial?
- PAYG includes 1,000 free proofs/month
- No credit card required to start

### What's the difference between Business and White-Label?
- **Business (€79)**: Custom branding in messages, shared infrastructure
- **White-Label (€190)**: BYOC - your own Telegram bot, WhatsApp number, etc.

---

## Why We're Different

### Traditional SMS Pricing

```
SMS verification costs $0.05-0.50+ per message
depending on country and carrier.

At 10,000/month = $500-5,000+
```

### proof.holdings Pricing

```
Business plan:
€79/month for 10,000 proofs
= €0.0079/proof

Includes: All channels, offline verification, webhooks, audit trail
```

**Same security. Better features. 10-50x lower cost.**

---

## Getting Started

1. **Sign up** - No credit card required
2. **Get API key** - Instant access
3. **Make first verification** - Takes 2 minutes
4. **Upgrade when ready** - Keep your data, change plan

```bash
# Your first verification
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{"type":"phone","channel":"whatsapp","identifier":"+37069199199"}'
```

---

*Questions about pricing? Contact hello@proof.holdings*
