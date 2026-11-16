# GoDaddy Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key + Secret |
| Rate Limit | 60 requests / minute |
| Propagation | Instant |
| Bonus | Can verify registrar ownership even if DNS hosted elsewhere |

---

## Setup

### 1. Create API Credentials

1. Go to [GoDaddy Developer Portal](https://developer.godaddy.com/keys)
2. Click "Create New API Key"
3. Set environment: **Production**
4. Copy both:
   - API Key (public)
   - API Secret (private, shown once)

### 2. Add to proof.holdings

1. Go to Dashboard → Domains
2. Click your domain → "Connect Provider"
3. Select "GoDaddy"
4. Enter API Key and Secret
5. Click "Verify & Save"

---

## Required Permissions

GoDaddy API keys have full account access. No granular permissions available.

---

## API Details

**Endpoint:** `https://api.godaddy.com/v1/`

**Authentication:**
```
Authorization: sso-key {api_key}:{api_secret}
```

**Get Domain Info:**
```bash
GET /domains/{domain}
```

**Create TXT Record:**
```bash
PATCH /domains/{domain}/records
[{
  "type": "TXT",
  "name": "_proof",
  "data": "proof-verify=ABC123",
  "ttl": 600
}]
```

---

## Registrar Ownership Verification

If your domain is registered with GoDaddy but DNS is hosted elsewhere (e.g., Cloudflare), GoDaddy credentials can still verify you own the domain registration.

This provides an alternative verification path without needing DNS API access.

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `ACCESS_DENIED` | Invalid credentials | Check API key and secret |
| `NOT_FOUND` | Domain not in account | Verify domain ownership |
| `RATE_LIMIT_EXCEEDED` | Too many requests | Wait 1 minute |
