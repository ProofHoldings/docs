# Akamai Edge DNS

> Enterprise-grade DNS service with global distribution and DDoS protection.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | EdgeGrid (HMAC-SHA256 signature) |
| **Rate Limit** | Contact Akamai for limits |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get EdgeGrid Credentials

1. Log in to [Akamai Control Center](https://control.akamai.com/)
2. Go to **Identity & Access Management** > **Users & API Clients**
3. Click **Create API Client**
4. Select **API service** and assign the **DNS Zone Administration** role
5. Click **Create** and download the credentials file
6. Note the following values:
   - Client Token
   - Client Secret
   - Access Token
   - Host (e.g., `akab-xxxxx.luna.akamaiapis.net`)

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Akamai Edge DNS**
3. Enter:
   - Client Token
   - Client Secret
   - Access Token
   - Host
4. Click **Save**

---

## Required Permissions

- DNS Zone Administration (read/write)
- Ability to manage TXT and CNAME records

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://{host}/config-dns/v2/` |
| **Auth Header** | EdgeGrid signature (HMAC-SHA256) |
| **TXT Record Creation** | `POST /zones/{zone}/names/{fqdn}/types/TXT` |

**Authentication Header Format:**
```
Authorization: EG1-HMAC-SHA256 client_token={token};access_token={token};timestamp={ts};nonce={nonce};signature={sig}
```

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid or expired EdgeGrid credentials
- Fix: Regenerate credentials in Akamai Control Center

**403 Forbidden**
- Cause: API client lacks DNS Zone Administration permission
- Fix: Update API client permissions

**404 Zone not found**
- Cause: Zone doesn't exist or isn't accessible
- Fix: Verify zone name and API client access

---

*Last updated: 2026-02-04*
