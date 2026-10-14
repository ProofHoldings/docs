# Cloudflare Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Token |
| Rate Limit | 1200 requests / 5 minutes |
| Propagation | Instant |

---

## Setup

### 1. Create API Token

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
2. Click "Create Token"
3. Use template: **Edit zone DNS**
4. Set permissions:
   - Zone > DNS > Edit
5. Set zone resources:
   - Include > Specific zone > your-domain.com
   - Or: Include > All zones (if managing multiple domains)
6. Click "Continue to summary" → "Create Token"
7. Copy the token (shown only once)

### 2. Add to proof.holdings

1. Go to Dashboard → Domains
2. Click your domain → "Connect Provider"
3. Select "Cloudflare"
4. Paste your API token
5. Click "Verify & Save"

---

## Required Permissions

```
Zone:DNS:Edit
```

Minimal scope: single zone only.

---

## API Details

**Endpoint:** `https://api.cloudflare.com/client/v4/`

**Authentication:**
```
Authorization: Bearer {api_token}
```

**Get Zone ID:**
```bash
GET /zones?name=example.com
```

**Create TXT Record:**
```bash
POST /zones/{zone_id}/dns_records
{
  "type": "TXT",
  "name": "_proof.example.com",
  "content": "proof-verify=ABC123",
  "ttl": 60
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `Authentication error` | Invalid or expired token | Create new token |
| `Zone not found` | Token doesn't have access to zone | Check zone permissions |
| `Rate limited` | Too many requests | Wait 5 minutes |
