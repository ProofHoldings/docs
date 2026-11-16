# Yandex Cloud DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | IAM Token or OAuth Token |
| Rate Limit | Standard Yandex Cloud limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to Yandex Cloud Console
2. Navigate to **Service Accounts** or use your user account
3. Create an IAM token or OAuth token:
   - For service accounts: Create authorized key and exchange for IAM token
   - For user accounts: Get OAuth token from Yandex OAuth
4. Note your folder ID (where DNS zone is located)

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Yandex Cloud DNS"
4. Enter your Token and Folder ID
5. Click "Verify & Save"

---

## Required Permissions

- dns.editor role or higher on the folder

---

## API Details

**Endpoint:** `https://dns.api.cloud.yandex.net/dns/v1`

**Authentication:**
```
Authorization: Bearer {iamToken}
```

**Get IAM Token (for service accounts):**
```bash
POST https://iam.api.cloud.yandex.net/iam/v1/tokens
Content-Type: application/json

{
  "jwt": "{signedJwt}"
}
```

**Create TXT Record:**
```bash
POST /dns/v1/zones/{dnsZoneId}:upsertRecordSets
Authorization: Bearer {iamToken}
Content-Type: application/json

{
  "additions": [
    {
      "name": "_proof.example.com.",
      "type": "TXT",
      "ttl": 300,
      "data": ["verification-code"]
    }
  ]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid or expired token | Refresh IAM token |
| 403 Forbidden | Insufficient permissions | Grant dns.editor role to account |
| Zone not found | Wrong folder or zone ID | Verify folder ID and zone exists |
| Token expired | IAM token timeout (12 hours) | Generate new IAM token |
