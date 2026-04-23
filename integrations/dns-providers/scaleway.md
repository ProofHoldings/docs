# Scaleway DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Secret Key (X-Auth-Token header) |
| Rate Limit | Standard Scaleway limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to Scaleway Console
2. Navigate to **Identity and Access Management** -> **API Keys**
3. Click **Generate new API key**
4. Set permissions and description
5. Copy the Secret Key (shown only once)

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Scaleway"
4. Enter your Secret Key
5. Click "Verify & Save"

---

## Required Permissions

- DNS Domains: Read/Write
- DNS Records: Read/Write

---

## API Details

**Endpoint:** `https://api.scaleway.com/domain/v2beta1`

**Authentication:**
```
X-Auth-Token: {secretKey}
```

**Create TXT Record:**
```bash
PATCH /dns-zones/{dnsZone}/records
Content-Type: application/json

{
  "changes": [
    {
      "add": {
        "records": [
          {
            "name": "_proof",
            "type": "TXT",
            "ttl": 300,
            "data": "verification-code"
          }
        ]
      }
    }
  ]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid secret key | Regenerate API key in Scaleway Console |
| 403 Forbidden | Insufficient permissions | Check API key has DNS permissions |
| Zone not found | Domain not in Scaleway DNS | Add domain to Scaleway DNS first |
