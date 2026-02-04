# DNS Made Easy

> Enterprise DNS hosting with 100% uptime SLA and global anycast network.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Key + Secret Key (HMAC-SHA1) |
| **Rate Limit** | 150 requests per 5 minutes |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Credentials

1. Log in to [DNS Made Easy](https://dnsmadeeasy.com/)
2. Go to **Config** > **Account Information**
3. Navigate to **API Credentials**
4. Generate or view your:
   - API Key
   - Secret Key

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **DNS Made Easy**
3. Enter:
   - API Key
   - Secret Key
4. Click **Save**

---

## Required Permissions

API credentials have full access to:
- Managed DNS domains
- DNS record management

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.dnsmadeeasy.com/V2.0` |
| **Auth Headers** | `x-dnsme-apiKey`, `x-dnsme-requestDate`, `x-dnsme-hmac` |
| **TXT Record Creation** | `POST /dns/managed/{domainId}/records` |

**Authentication Headers:**
```
x-dnsme-apiKey: {your_api_key}
x-dnsme-requestDate: {UTC_date_string}
x-dnsme-hmac: {hmac_sha1_hex}
```

The HMAC is calculated from the request date using your secret key.

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid API key or HMAC signature
- Fix: Verify API key and secret key

**403 Forbidden**
- Cause: Account restrictions or IP not whitelisted
- Fix: Check account status and IP whitelist settings

**404 Domain not found**
- Cause: Domain ID doesn't exist
- Fix: Verify domain is in DNS Made Easy

---

*Last updated: 2026-02-04*
