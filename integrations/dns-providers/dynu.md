# Dynu

> Dynamic DNS and domain hosting service with free DNS options.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Key |
| **Rate Limit** | Contact Dynu for limits |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Key

1. Log in to [Dynu Control Panel](https://www.dynu.com/en-US/ControlPanel)
2. Go to **API Credentials**
3. Generate or view your API Key

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Dynu**
3. Enter:
   - API Key
4. Click **Save**

---

## Required Permissions

API key provides full access to:
- Domain management
- DNS record management

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.dynu.com/v2` |
| **Auth Header** | `API-Key: {your_api_key}` |
| **TXT Record Creation** | `POST /dns/{domainId}/record` |

**Create TXT Record:**
```json
{
  "nodeName": "_proof",
  "recordType": "TXT",
  "textData": "verification-code",
  "ttl": 300,
  "state": true
}
```

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid API key
- Fix: Regenerate API key in Dynu control panel

**403 Forbidden**
- Cause: API access restricted
- Fix: Check account status and API permissions

**Domain not found**
- Cause: Domain not in Dynu account
- Fix: Add domain to Dynu first

---

*Last updated: 2026-02-04*
