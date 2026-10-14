# IONOS (1&1) Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key (X-API-Key header) |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to IONOS Developer Portal
2. Navigate to **API Keys** section
3. Click **Create API Key**
4. Copy the API key (format: `publicprefix.secret`)

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "IONOS"
4. Enter your API Key
5. Click "Verify & Save"

---

## Required Permissions

- DNS zone management
- Record read/write access

---

## API Details

**Endpoint:** `https://api.hosting.ionos.com/dns/v1`

**Authentication:**
```
X-API-Key: {publicprefix.secret}
```

**Create TXT Record:**
```bash
POST /zones/{zoneId}/records
Content-Type: application/json

[
  {
    "name": "_proof.example.com",
    "type": "TXT",
    "content": "verification-code",
    "ttl": 300,
    "prio": 0,
    "disabled": false
  }
]
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API key | Regenerate API key in IONOS Developer Portal |
| Zone not found | Domain not in account | Verify domain is registered with IONOS |
| Invalid API key format | Wrong key format | Ensure key includes both prefix and secret |
