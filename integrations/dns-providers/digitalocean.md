# DigitalOcean Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Personal Access Token |
| Rate Limit | 250 requests / minute |
| Propagation | Instant |

---

## Setup

### 1. Create Personal Access Token

1. Go to [DigitalOcean API Tokens](https://cloud.digitalocean.com/account/api/tokens)
2. Click "Generate New Token"
3. Name it (e.g., "proof.holdings DNS")
4. Select scope: **Write** (required for creating records)
5. Click "Generate Token"
6. Copy the token (shown only once)

### 2. Add to proof.holdings

1. Go to Dashboard → Domains
2. Click your domain → "Connect Provider"
3. Select "DigitalOcean"
4. Paste your API token
5. Click "Verify & Save"

---

## Required Permissions

Token must have **Write** scope.

---

## API Details

**Endpoint:** `https://api.digitalocean.com/v2`

**Authentication:**
```
Authorization: Bearer {api_token}
```

**List Domains:**
```bash
GET /domains
```

**Create TXT Record:**
```bash
POST /domains/{domain}/records
{
  "type": "TXT",
  "name": "_proof",
  "data": "proof-verify=ABC123",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Unauthorized` | Invalid or expired token | Generate new token |
| `404 Not Found` | Domain not in account | Add domain to DigitalOcean DNS |
| `429 Too Many Requests` | Rate limit exceeded | Wait and retry |
