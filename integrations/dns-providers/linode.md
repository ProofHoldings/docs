# Linode Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Personal Access Token (Bearer) |
| Rate Limit | Standard Linode API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to Linode Cloud Manager
2. Click your profile icon -> **API Tokens**
3. Click **Create a Personal Access Token**
4. Set label and expiry
5. Select **Domains** scope with Read/Write access
6. Click **Create Token** and copy it immediately

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Linode"
4. Enter your API Token
5. Click "Verify & Save"

---

## Required Permissions

- Domains: Read/Write

---

## API Details

**Endpoint:** `https://api.linode.com/v4`

**Authentication:**
```
Authorization: Bearer {apiToken}
```

**Create TXT Record:**
```bash
POST /domains/{domainId}/records
Content-Type: application/json

{
  "type": "TXT",
  "name": "_proof",
  "target": "verification-code",
  "ttl_sec": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API token | Regenerate token in Linode Cloud Manager |
| Insufficient permissions | Wrong token scope | Ensure Domains Read/Write scope is enabled |
| Domain not found | Domain not in account | Add domain to Linode DNS Manager first |
