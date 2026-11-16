# Netlify Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Personal Access Token (Bearer) |
| Rate Limit | Standard Netlify API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Netlify account
2. Navigate to **User Settings** -> **Applications**
3. Under **Personal access tokens**, click **New access token**
4. Give it a description and click **Generate token**
5. Copy the token immediately (shown only once)

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Netlify"
4. Enter your Personal Access Token
5. Click "Verify & Save"

---

## Required Permissions

- Personal access token with DNS management access

---

## API Details

**Endpoint:** `https://api.netlify.com/api/v1`

**Authentication:**
```
Authorization: Bearer {accessToken}
```

**Create TXT Record:**
```bash
POST /dns_zones/{zoneId}/dns_records
Content-Type: application/json

{
  "type": "TXT",
  "hostname": "_proof",
  "value": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid access token | Regenerate token in Netlify settings |
| 404 Zone not found | Domain not in Netlify DNS | Add domain to Netlify DNS first |
| 403 Forbidden | Insufficient permissions | Ensure token has DNS access |
