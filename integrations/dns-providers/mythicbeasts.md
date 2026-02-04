# Mythic Beasts Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Bearer Token or Basic Auth |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Mythic Beasts control panel
2. Navigate to **API Keys** section
3. Create a new API key with DNS permissions
4. Copy the key ID and secret

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Mythic Beasts"
4. Enter your Key ID and Secret
5. Click "Verify & Save"

---

## Required Permissions

- DNS zone management
- Record read/write access

---

## API Details

**Endpoint:** `https://api.mythic-beasts.com/dns/v2`

**Authentication (Bearer Token):**
```
Authorization: Bearer {accessToken}
```

**Authentication (Basic Auth):**
```
Authorization: Basic base64({keyId}:{secret})
```

**Create TXT Record:**
```bash
POST /zones/{zone}/records/{name}/TXT
Content-Type: application/json

{
  "records": [
    {
      "data": "verification-code",
      "ttl": 300
    }
  ]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API credentials | Regenerate API key in control panel |
| 403 Forbidden | Insufficient permissions | Ensure API key has DNS management permissions |
| Zone not found | Domain not managed | Verify domain is configured with Mythic Beasts DNS |
