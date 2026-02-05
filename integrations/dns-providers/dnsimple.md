# DNSimple

> Developer-friendly DNS and domain management platform.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | OAuth Access Token or API Token (Bearer) |
| **Rate Limit** | 2400 requests per hour |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Token

1. Log in to [DNSimple](https://dnsimple.com/)
2. Go to **Account** > **Automation**
3. Click **New Access Token**
4. Name it (e.g., "proof-holdings")
5. Copy the token (shown only once)
6. Note your **Account ID** from the URL or Account page

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **DNSimple**
3. Enter:
   - API Token
   - Account ID
4. Click **Save**

---

## Required Permissions

API tokens have full access to:
- Domain management
- DNS record management
- Zone configuration

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.dnsimple.com/v2` |
| **Auth Header** | `Authorization: Bearer {api_token}` |
| **TXT Record Creation** | `POST /{accountId}/zones/{zone}/records` |

**Create Record Request:**
```json
{
  "name": "_proof",
  "type": "TXT",
  "content": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid or expired API token
- Fix: Generate a new token in DNSimple

**404 Zone not found**
- Cause: Domain not in DNSimple account
- Fix: Verify domain is added to your account

**Rate limit exceeded**
- Cause: Too many requests
- Fix: Wait and retry (limit resets hourly)

---

*Last updated: 2026-02-04*
