---
title: Pricing
lastUpdated: "2026-08-25"
status: current
---

# proof.holdings Pricing

> Simple, transparent pricing. Pay for proofs, not messages.

<!-- GENERATED:billing-status:START -->
> **Usage is currently free.** Billing is not switched on yet: proofs beyond your included
> amount are recorded, and nothing is charged. The prices below are what they will cost once
> billing goes live — nothing on this page is being billed today.
<!-- GENERATED:billing-status:END -->

---

## The Value Proposition

**€79/month for what costs $1,000+ with SMS.**

Traditional SMS verification costs $0.05-0.40 per message. At 10,000 verifications:
- Twilio SMS: $500-4,000/month
- proof.holdings: **€79/month**

And you get more: offline-verifiable proofs, multi-channel support, webhooks, audit trails.

---

## Plans

<!-- GENERATED:plans:START -->
### Pay as you go

**€0.03/proof** (300 free proofs/month)

- No monthly fee
- No commitment
- All proof types
- Unlimited API calls
- Test mode included

*Perfect for: Testing, side projects, variable usage, early-stage startups*

---

### €9/month

- 1,000 proofs/month included
- €0.03 per additional proof
- **€0.009/proof effective rate**
- All proof types
- Webhooks
- Custom branding
- Test mode included
- Email support

*Perfect for: Live products with steady, modest volume*

---

### €29/month

- 3,000 proofs/month included
- €0.025 per additional proof
- **€0.0097/proof effective rate**
- All proof types
- Webhooks
- Custom branding
- Test mode included
- Priority support

*Perfect for: Growing apps and small businesses*

---

### €79/month

- 10,000 proofs/month included
- €0.02 per additional proof
- **€0.0079/proof effective rate**
- All proof types
- Webhooks
- Custom branding
- Test mode included
- Priority support

*Perfect for: Scale-ups and SaaS platforms*

---

### Custom

**Custom pricing**

- 100,000+ proofs/month
- Volume discounts
- Dedicated infrastructure
- Bring your own channels
- Custom terms

*Perfect for: High volume, regulated, or bring-your-own-channel deployments*
<!-- GENERATED:plans:END -->

---

## Custom Branding (All Plans)

**All plans** include custom branding on shared infrastructure:
- `@proof_holdings_bot` on Telegram
- Shared WhatsApp and  numbers
- Emails from proof.holdings

**Custom branding** lets users see YOUR context in messages:

| Element | Shared | Custom Branding |
|---------|--------|-----------------|
| Business name | "Proof Holdings" | "Your Company Name" |
| Email logo | proof.holdings logo | Your logo |
| Email colors | Default blue | Your brand colors |
| Support email | hello@proof.holdings | support@yourcompany.com |
| Message templates | Default text | Custom text |

---

## BYOC - Bring Your Own Channels (Custom plan)

Full channel ownership - users interact with YOUR endpoints:

| Channel | Shared | BYOC |
|---------|--------|------|
| Telegram | `@proof_holdings_bot` | `@your_company_bot` |
| WhatsApp | Shared number | Your WhatsApp Business number |
| Shared number | Your  Business number |
| Email | verify@verify.proof.holdings | noreply@yourcompany.com |
| SMS | Shared number | Your phone number |

BYOC = complete channel ownership. Users never see proof.holdings.

---

## Cost Comparison

### Per-Verification Cost

<!-- GENERATED:cost-table:START -->
| Volume | Pay as you go | €9/mo | €29/mo | €79/mo | SMS (Twilio) |
|---|---|---|---|---|---|
| 1,000 | €21 | €9 | €29 | €79 | $50-400 |
| 3,000 | €81 | €69 | €29 | €79 | $150-1,200 |
| 10,000 | €291 | €279 | €204 | €79 | $500-4,000 |
| 50,000 | €1,491 | €1,479 | €1,204 | €879 | $2,500-20,000 |
<!-- GENERATED:cost-table:END -->

### Effective Rate at Volume

<!-- GENERATED:effective-rate:START -->
| Plan | Base price | Included | Rate after | Effective at 10K |
|---|---|---|---|---|
| Pay as you go | €0 | 300 | €0.03 | €0.0291 |
| €9/mo | €9 | 1,000 | €0.03 | €0.0279 |
| €29/mo | €29 | 3,000 | €0.025 | €0.0204 |
| €79/mo | €79 | 10,000 | €0.02 | €0.0079 |
| Custom | Custom | 100,000+ | Volume discount | Contact us |
<!-- GENERATED:effective-rate:END -->

---

## What's a Proof?

**1 proof = 1 verified asset**

- ✅ Phone number verified via Telegram → 1 proof
- ✅ Phone number verified via WhatsApp → 1 proof
- ✅ Phone number verified via  → 1 proof
- ✅ Phone number verified via SMS → 1 proof
- ✅ Email verified via magic link → 1 proof
- ✅ Domain verified via DNS → 1 proof
- ✅ Social account verified via OAuth → 1 proof (coming soon)
- ✅ Wallet verified via signature → 1 proof (coming soon)

**What's NOT counted:**
- ❌ Failed verification attempts
- ❌ Expired challenges (not completed)
- ❌ Test mode verifications (`pk_test_` keys — unlimited, free)
- ❌ API calls (unlimited)
- ❌ Webhook deliveries (unlimited)
- ❌ Proof validations (unlimited, offline)

---

## All Plans Include

- **All verification types** - Phone, email, domain, social, wallet
- **All channels** - Telegram, WhatsApp, SMS, Email, DNS, HTTP
- **Unlimited API calls** - No rate limits on reads
- **Unlimited proof validation** - Verify offline with JWKS
- **Test mode** - Full functionality, no charges
- **Webhooks** - Real-time notifications
- **Dashboard** - Manage everything in one place
- **Documentation** - Full API docs, examples

---

## Features by Plan

<!-- GENERATED:features:START -->
| Feature | Pay as you go | €9 | €29 | €79 | Custom |
|---|---|---|---|---|---|
| All channels (Telegram, WhatsApp, SMS, Email, DNS, HTTP) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Custom branding (business name, logo, colors) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Custom message templates | ✅ | ✅ | ✅ | ✅ | ✅ |
| Webhooks | ✅ | ✅ | ✅ | ✅ | ✅ |
| BYOC (your own Telegram, WhatsApp, Email, SMS) | ❌ | ❌ | ❌ | ❌ | ✅ |
| Dedicated support | ❌ | ❌ | ❌ | ✅ | ✅ |
<!-- GENERATED:features:END -->

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
- PAYG includes 300 free proofs/month
- No credit card required to start
- Test mode (`pk_test_` keys) is always free with unlimited verifications

### What exactly is test mode?
- Use `pk_test_` API keys instead of `pk_live_` — all API calls work identically
- No real messages are sent (SMS, email, WhatsApp, Telegram are all simulated)
- Use the `test-verify` endpoint to auto-complete verifications instantly
- Test verifications don't count toward your monthly quota or billing
- Webhooks still fire (with `"test": true` flag) so you can test your integration end-to-end

### What's the difference between the €79/month and Custom plans?
- **€79/month**: Custom branding in messages, shared infrastructure
- **Custom**: BYOC - your own Telegram bot, WhatsApp number, dedicated infrastructure

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
€79/month plan:
€79 for 10,000 proofs
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
