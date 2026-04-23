# Hostinger Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Bearer Token |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Hostinger account
2. Navigate to **Account Settings** or **API** section
3. Generate a new API token
4. Copy the API token securely

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Hostinger"
4. Enter your API Token
5. Click "Verify & Save"

---

## Required Permissions

- Domain management access
- DNS record read/write permissions

---

## API Details

**Endpoint:** `https://api.hostinger.com/v1`

**Authentication:**
```
Authorization: Bearer {apiToken}
```

**Create TXT Record:**
```bash
POST /domains/{domainId}/dns
Content-Type: application/json

{
  "type": "TXT",
  "name": "_proof",
  "content": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API token | Regenerate API token in Hostinger dashboard |
| 403 Forbidden | Insufficient permissions | Ensure token has DNS management access |
| Domain not found | Domain not in account | Verify domain is registered with Hostinger |
