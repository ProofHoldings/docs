# Rackspace Cloud DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | OpenStack Auth (Username + API Key) |
| Rate Limit | Standard Rackspace limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to Rackspace Cloud Control Panel
2. Navigate to **Account Settings** -> **API Keys**
3. View or regenerate your API key
4. Note your Rackspace username and API key

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Rackspace"
4. Enter your Username and API Key
5. Click "Verify & Save"

---

## Required Permissions

- Cloud DNS management access

---

## API Details

**Auth Endpoint:** `https://identity.api.rackspacecloud.com/v2.0`
**DNS Endpoint:** `https://dns.api.rackspacecloud.com/v1.0/{accountId}`

**Authentication (Get Token):**
```bash
POST /tokens
Content-Type: application/json

{
  "auth": {
    "RAX-KSKEY:apiKeyCredentials": {
      "username": "{username}",
      "apiKey": "{apiKey}"
    }
  }
}
```

**Create TXT Record:**
```bash
POST /domains/{domainId}/records
X-Auth-Token: {authToken}
Content-Type: application/json

{
  "records": [
    {
      "name": "_proof.example.com",
      "type": "TXT",
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
| 401 Unauthorized | Invalid credentials | Verify username and API key |
| Token expired | Auth token timeout | Re-authenticate to get new token |
| Domain not found | Domain not in account | Add domain to Rackspace Cloud DNS |
