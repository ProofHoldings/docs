# Zilore DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key (X-Auth-Key header) |
| Rate Limit | Standard Zilore limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Zilore account
2. Navigate to **Account Settings** -> **API**
3. Generate a new API key
4. Copy the API key

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Zilore"
4. Enter your API Key
5. Click "Verify & Save"

---

## Required Permissions

- DNS zone management
- Record read/write access

---

## API Details

**Endpoint:** `https://api.zilore.com/dns/v1`

**Authentication:**
```
X-Auth-Key: {apiKey}
```

**Create TXT Record:**
```bash
POST /zones/{zoneName}/records
Content-Type: application/json
X-Auth-Key: {apiKey}

{
  "name": "_proof",
  "type": "TXT",
  "content": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API key | Regenerate API key in Zilore account |
| Zone not found | Domain not in account | Add domain to Zilore DNS first |
| Invalid record | Wrong record format | Check record parameters |
