# TransIP Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Private Key JWT Authentication |
| Rate Limit | Standard TransIP limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to TransIP Control Panel
2. Navigate to **My Account** -> **API**
3. Enable API access
4. Generate a new key pair
5. Download the private key file
6. Note your TransIP login name

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "TransIP"
4. Enter your Login Name and Private Key
5. Click "Verify & Save"

---

## Required Permissions

- API access enabled
- Domain management permissions

---

## API Details

**Endpoint:** `https://api.transip.nl/v6`

**Authentication (JWT with Private Key):**
```
1. Create JWT token signed with private key
2. Exchange for access token:
   POST /auth
   {"login": "{loginName}", "nonce": "{uuid}", "read_only": false, "expiration_time": "30 minutes", "label": "proof.holdings", "global_key": true}
   Signature: base64(rsa_sign(body, privateKey))

3. Use access token:
   Authorization: Bearer {accessToken}
```

**Create TXT Record:**
```bash
POST /domains/{domainName}/dns
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "dnsEntry": {
    "name": "_proof",
    "expire": 300,
    "type": "TXT",
    "content": "verification-code"
  }
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid token | Re-authenticate with private key |
| Invalid signature | Wrong private key | Ensure private key matches API configuration |
| Domain not found | Domain not in account | Verify domain is registered with TransIP |
| API disabled | API access not enabled | Enable API in TransIP Control Panel |
