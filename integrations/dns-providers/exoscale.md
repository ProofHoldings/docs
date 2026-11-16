# Exoscale DNS

> European cloud platform with DNS hosting services.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Key + Secret (Basic Auth) |
| **Rate Limit** | Contact Exoscale for limits |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Credentials

1. Log in to [Exoscale Portal](https://portal.exoscale.com/)
2. Go to **IAM** > **API Keys**
3. Click **Create API Key**
4. Note the:
   - API Key
   - API Secret (shown only once)

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Exoscale**
3. Enter:
   - API Key
   - API Secret
4. Click **Save**

---

## Required Permissions

API keys can be scoped to:
- DNS operations only
- Full account access

For DNS only, select **DNS** scope when creating the key.

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.exoscale.com/dns/v1` |
| **Auth Header** | `Authorization: Basic {base64(key:secret)}` |
| **TXT Record Creation** | `POST /domains/{domain}/records` |

**Create TXT Record:**
```json
{
  "record": {
    "name": "_proof",
    "record_type": "TXT",
    "content": "verification-code",
    "ttl": 300
  }
}
```

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid API key or secret
- Fix: Verify credentials or regenerate

**403 Forbidden**
- Cause: API key lacks DNS permissions
- Fix: Create key with DNS scope

**404 Domain not found**
- Cause: Domain not in Exoscale DNS
- Fix: Add domain to Exoscale first

---

*Last updated: 2026-02-04*
