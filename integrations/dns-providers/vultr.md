# Vultr DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Bearer Token (API Key) |
| Rate Limit | Standard Vultr API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Vultr account
2. Navigate to **Account** -> **API**
3. Click **Enable API** if not already enabled
4. Copy your API key
5. Optionally configure IP access control

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Vultr"
4. Enter your API Key
5. Click "Verify & Save"

---

## Required Permissions

- API access enabled
- DNS management permissions (default with API key)

---

## API Details

**Endpoint:** `https://api.vultr.com/v2`

**Authentication:**
```
Authorization: Bearer {apiKey}
```

**Create TXT Record:**
```bash
POST /domains/{domain}/records
Content-Type: application/json

{
  "name": "_proof",
  "type": "TXT",
  "data": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API key | Verify API key in Vultr account |
| 403 Forbidden | IP not allowed | Add your IP to API access list |
| Domain not found | Domain not in Vultr DNS | Add domain to Vultr DNS first |
