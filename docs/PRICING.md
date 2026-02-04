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

### Free
**€0/month**

- 1,000 proofs/month
- All verification types (phone, email, domain)
- Unlimited API access
- Test mode included
- Community support

*Perfect for: Testing, side projects, getting started*

---

### Pay As You Go
**€0.03/proof**

- No monthly fee
- No commitment
- All verification types
- Unlimited API access
- Email support

*Perfect for: Variable usage, early-stage startups*

---

### Pro
**€29/month**

- 3,000 proofs included
- €0.025 per additional proof
- **€0.0097/proof effective rate**
- All verification types
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
- Webhooks
- Bring your own phone numbers (BYOC)
- Priority support

*Perfect for: Scale-ups, SaaS platforms*

---

### Enterprise
**Custom pricing**

- 100,000+ proofs/month
- Volume discounts
- Custom SLA
- Dedicated infrastructure

*Contact: hello@proof.holdings*

---

## Cost Comparison

### Per-Verification Cost

| Volume | Free | PAYG | Pro | Business | SMS (Twilio) |
|--------|------|------|-----|----------|--------------|
| 1,000 | €0 | €30 | €29 | €79 | $50-400 |
| 3,000 | - | €90 | €29 | €79 | $150-1,200 |
| 10,000 | - | €300 | €204 | €79 | $500-4,000 |
| 50,000 | - | €1,500 | €1,204 | €879 | $2,500-20,000 |

### Effective Rate at Volume

| Plan | Included | Rate After | Effective at 10K |
|------|----------|------------|------------------|
| Free | 1,000 | - | - |
| PAYG | 0 | €0.03 | €0.03 |
| Pro | 3,000 | €0.025 | €0.0204 |
| Business | 10,000 | €0.02 | **€0.0079** |

---

## What's a Proof?

**1 proof = 1 verified asset**

- ✅ Phone number verified via Telegram → 1 proof
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
- **Unlimited API calls** - No rate limits on reads
- **Unlimited proof validation** - Verify offline with JWKS
- **Test mode** - Full functionality, no charges
- **Webhooks** - Real-time notifications
- **Dashboard** - Manage everything in one place
- **Documentation** - Full API docs, examples

---

## Channels by Plan

| Channel | Free | PAYG | Pro | Business |
|---------|------|------|-----|----------|
| Telegram | ✅ | ✅ | ✅ | ✅ |
| WhatsApp | ✅ | ✅ | ✅ | ✅ |
| SMS | ✅ | ✅ | ✅ | ✅ |
| Email | ✅ | ✅ | ✅ | ✅ |
| DNS | ✅ | ✅ | ✅ | ✅ |
| HTTP | ✅ | ✅ | ✅ | ✅ |
| Own phone numbers (BYOC) | ❌ | ❌ | ❌ | ✅ |

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
- PAYG has no expiration

### Is there a free trial?
- Free plan is forever free (1,000/month)
- No credit card required

---

## Why We're Different

### Traditional SMS Pricing

```
Twilio:
$0.0079/segment + carrier fees + phone number rental
= $0.05-0.40 per verification

At 10,000/month = $500-4,000
```

### proof.holdings Pricing

```
Business plan:
€79/month for 10,000 proofs
= €0.0079/proof

Includes: All channels, offline verification, webhooks, audit trail
```

**Same security. Better features. 10-50x cheaper.**

---

## Getting Started

1. **Sign up free** - No credit card required
2. **Get API key** - Instant access
3. **Make first verification** - Takes 2 minutes
4. **Upgrade when ready** - Keep your data, change plan

```bash
# Your first verification
curl -X POST https://api.proof.holdings/api/v1/verifications \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{"type":"phone","channel":"telegram","identifier":"+15551234567"}'
```

---

*Questions about pricing? Contact hello@proof.holdings*
