# Name.com Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Basic Auth (Username + Token) |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Name.com account
2. Navigate to **Account** -> **API Tokens**
3. Click **Generate New Token**
4. Copy the token immediately (shown only once)
5. Note your Name.com username

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Name.com"
4. Enter your Username and API Token
5. Click "Verify & Save"

---

## Required Permissions

- API token with DNS management access

---

## API Details

**Endpoint:** `https://api.name.com/v4`

**Authentication:**
```
Authorization: Basic base64({username}:{token})
```

**Create TXT Record:**
```bash
POST /domains/{domain}/records
Content-Type: application/json

{
  "host": "_proof",
  "type": "TXT",
  "answer": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid credentials | Verify username and API token |
| 403 Forbidden | Insufficient permissions | Check token permissions |
| Domain not found | Domain not in account | Verify domain is registered with Name.com |
