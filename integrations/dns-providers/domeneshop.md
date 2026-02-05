# Domeneshop

> Norwegian domain registrar and web hosting provider.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Token + Secret (Basic Auth) |
| **Rate Limit** | Contact Domeneshop for limits |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Credentials

1. Log in to [Domeneshop Control Panel](https://www.domeneshop.no/admin)
2. Go to **My Account** > **API Access**
3. Click **Generate API Token**
4. Note the:
   - API Token
   - API Secret

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Domeneshop**
3. Enter:
   - API Token
   - API Secret
4. Click **Save**

---

## Required Permissions

- Domain must have DNS service enabled
- API access enabled on account

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.domeneshop.no/v0` |
| **Auth Header** | `Authorization: Basic {base64(token:secret)}` |
| **TXT Record Creation** | `POST /domains/{domainId}/dns` |

**Create Record Request:**
```json
{
  "host": "_proof",
  "type": "TXT",
  "data": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid API token or secret
- Fix: Regenerate credentials in Domeneshop

**403 Forbidden**
- Cause: API access not enabled
- Fix: Enable API access in account settings

**DNS service not enabled**
- Cause: Domain doesn't have DNS service
- Fix: Enable DNS service for the domain

---

*Last updated: 2026-02-04*
