# LuaDNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Basic Auth (Email + API Token) |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your LuaDNS account
2. Navigate to **Account Settings** -> **API Token**
3. Generate or copy your API token
4. Note your account email address

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "LuaDNS"
4. Enter your Email and API Token
5. Click "Verify & Save"

---

## Required Permissions

- DNS zone management
- Record create/delete access

---

## API Details

**Endpoint:** `https://api.luadns.com/v1`

**Authentication:**
```
Authorization: Basic base64({email}:{apiToken})
```

**Create TXT Record:**
```bash
POST /zones/{zoneId}/records
Content-Type: application/json

{
  "name": "_proof.example.com.",
  "type": "TXT",
  "content": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid credentials | Verify email and API token are correct |
| Zone not found | Domain not in account | Add domain to LuaDNS first |
| Invalid token | API token expired or revoked | Generate new API token in account settings |
