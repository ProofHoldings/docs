# Hetzner Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Token |
| Rate Limit | 3600 requests / hour |
| Propagation | Instant |
| Region | German provider |

---

## Setup

### 1. Create API Token

1. Go to [Hetzner DNS Console](https://dns.hetzner.com/)
2. Click your profile → "API tokens"
3. Click "Create API token"
4. Copy the token (shown only once)

### 2. Add to proof.holdings

1. Go to Dashboard → Domains
2. Click your domain → "Connect Provider"
3. Select "Hetzner"
4. Paste your API token
5. Click "Verify & Save"

---

## Required Permissions

Hetzner API tokens have full DNS access. No granular permissions available.

---

## API Details

**Endpoint:** `https://dns.hetzner.com/api/v1`

**Authentication:**
```
Auth-API-Token: {api_token}
```

**List Zones:**
```bash
GET /zones
```

**Create TXT Record:**
```bash
POST /records
{
  "zone_id": "abc123",
  "type": "TXT",
  "name": "_proof",
  "value": "proof-verify=ABC123",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Unauthorized` | Invalid API token | Check token in Hetzner console |
| `Zone not found` | Domain not in account | Add domain to Hetzner DNS |
| `422 Unprocessable Entity` | Invalid record format | Check record parameters |
