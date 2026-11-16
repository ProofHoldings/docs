# Infomaniak Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Bearer Token |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Infomaniak Manager
2. Navigate to **Developer** -> **API Tokens**
3. Click **Create a token**
4. Select necessary scopes (domain management)
5. Copy the generated API token

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Infomaniak"
4. Enter your API Token
5. Click "Verify & Save"

---

## Required Permissions

- Domain service access
- DNS record management

---

## API Details

**Endpoint:** `https://api.infomaniak.com`

**Authentication:**
```
Authorization: Bearer {apiToken}
```

**Create TXT Record:**
```bash
POST /1/domain/{domainId}/dns/record
Content-Type: application/json

{
  "type": "TXT",
  "source": "_proof",
  "target": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API token | Regenerate token in Infomaniak Manager |
| 403 Forbidden | Insufficient permissions | Check token scopes include domain management |
| Domain not found | Domain not in account | Verify domain ownership in Infomaniak |
