# Rage4 Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Basic Auth (Email + API Key) |
| Rate Limit | Standard API limits |
| Propagation | Near instant (GeoDNS) |

---

## Setup

### 1. Create API Credentials

1. Log in to your Rage4 account
2. Navigate to **Account Settings**
3. Find your API key or generate a new one
4. Note your account email and API key

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Rage4"
4. Enter your Email and API Key
5. Click "Verify & Save"

---

## Required Permissions

- DNS zone management
- Record create/delete access

---

## API Details

**Endpoint:** `https://rage4.com/rapi`

**Authentication:**
```
Authorization: Basic base64({email}:{apiKey})
```

**Create TXT Record:**
```bash
POST /createrecord
Content-Type: application/json

{
  "id": {domainId},
  "name": "_proof.example.com",
  "content": "verification-code",
  "type": "TXT",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid credentials | Verify email and API key |
| Domain not found | Domain not in account | Add domain to Rage4 first |
| Invalid record | Wrong record format | Check record name and content format |
