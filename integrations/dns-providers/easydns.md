# EasyDNS

> Canadian DNS hosting provider with DDoS protection and advanced features.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Token + API Key (Basic Auth) |
| **Rate Limit** | Contact EasyDNS for limits |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Credentials

1. Log in to [EasyDNS](https://cp.easydns.com/)
2. Go to **Account** > **API Access**
3. Generate API credentials:
   - API Token
   - API Key

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **EasyDNS**
3. Enter:
   - API Token
   - API Key
4. Click **Save**

---

## Required Permissions

- Domain management access
- DNS record creation and deletion

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://rest.easydns.net` |
| **Auth Header** | `Authorization: Basic {base64(token:key)}` |
| **TXT Record Creation** | `PUT /zones/records/add/{domain}/TXT` |

**Create TXT Record:**
```json
{
  "host": "_proof",
  "rdata": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid API token or key
- Fix: Verify credentials in EasyDNS control panel

**403 Forbidden**
- Cause: No access to domain
- Fix: Check domain ownership and API permissions

**404 Domain not found**
- Cause: Domain not in EasyDNS account
- Fix: Add domain to EasyDNS first

---

*Last updated: 2026-02-04*
