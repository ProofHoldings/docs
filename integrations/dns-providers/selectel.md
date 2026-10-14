# Selectel DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Token (X-Token header) |
| Rate Limit | Standard Selectel limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to Selectel Control Panel
2. Navigate to **Profile** -> **API Keys**
3. Click **Generate Key**
4. Copy the generated API token

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Selectel"
4. Enter your API Token
5. Click "Verify & Save"

---

## Required Permissions

- DNS service access
- Zone management permissions

---

## API Details

**Endpoint:** `https://api.selectel.ru/domains/v2`

**Authentication:**
```
X-Token: {apiToken}
```

**Create TXT Record:**
```bash
POST /zones/{zoneId}/rrset
Content-Type: application/json

{
  "name": "_proof.example.com.",
  "type": "TXT",
  "ttl": 300,
  "records": [
    {
      "content": "verification-code"
    }
  ]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API token | Regenerate token in Selectel Control Panel |
| 403 Forbidden | Insufficient permissions | Ensure token has DNS access |
| Zone not found | Domain not in account | Add domain to Selectel DNS first |
