# Vercel DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Bearer Token |
| Rate Limit | Standard Vercel API limits |
| Propagation | Near instant |

---

## Setup

### 1. Create API Credentials

1. Log in to your Vercel account
2. Navigate to **Settings** -> **Tokens**
3. Click **Create Token**
4. Set token name, scope, and expiration
5. Copy the token immediately (shown only once)

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Vercel"
4. Enter your API Token
5. Click "Verify & Save"

---

## Required Permissions

- Full Account access or specific project access with DNS permissions

---

## API Details

**Endpoint:** `https://api.vercel.com`

**Authentication:**
```
Authorization: Bearer {token}
```

**Create TXT Record:**
```bash
POST /v4/domains/{domain}/records
Content-Type: application/json

{
  "name": "_proof",
  "type": "TXT",
  "value": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid token | Regenerate token in Vercel settings |
| 403 Forbidden | Insufficient scope | Create token with full account access |
| Domain not found | Domain not in Vercel | Add domain to Vercel project first |
| Token expired | Token past expiration | Create new token with longer expiration |
