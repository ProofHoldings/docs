# Gandi LiveDNS

> Domain registrar with integrated LiveDNS hosting service.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | Personal Access Token or API Key (Bearer) |
| **Rate Limit** | Contact Gandi for limits |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Token

1. Log in to [Gandi](https://www.gandi.net/)
2. Go to **Account** > **Security** > **API Key** (deprecated) or **Personal Access Tokens**
3. Click **Create Token**
4. Select permissions:
   - Manage domain DNS records
5. Copy the token

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Gandi**
3. Enter:
   - API Key / Personal Access Token
4. Click **Save**

---

## Required Permissions

Personal Access Token needs:
- See and renew domain names
- Manage domain technical configurations (DNS)

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.gandi.net/v5` |
| **Auth Header** | `Authorization: Bearer {api_key}` |
| **TXT Record Creation** | `PUT /livedns/domains/{domain}/records/{name}/TXT` |

**Create/Update TXT Record:**
```json
{
  "rrset_values": ["verification-code"],
  "rrset_ttl": 300
}
```

**Note:** Gandi uses PUT for both create and update operations.

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid or expired API key
- Fix: Generate new personal access token

**403 Forbidden**
- Cause: Token lacks required permissions
- Fix: Create token with DNS management scope

**404 Domain not found**
- Cause: Domain not using Gandi LiveDNS
- Fix: Enable LiveDNS for the domain

---

*Last updated: 2026-02-04*
