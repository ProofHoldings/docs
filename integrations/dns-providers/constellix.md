# Constellix DNS

> Enterprise DNS platform with GeoDNS, failover, and real-time analytics.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Key + Secret Key (HMAC-SHA1) |
| **Rate Limit** | Varies by plan |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Credentials

1. Log in to [Constellix Dashboard](https://dns.constellix.com/)
2. Go to **Account** > **API Keys**
3. Click **Create API Key**
4. Note the:
   - API Key
   - Secret Key (shown only once)

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Constellix**
3. Enter:
   - API Key
   - Secret Key
4. Click **Save**

---

## Required Permissions

API keys have full access to:
- Domain management
- DNS record management
- Zone configuration

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.dns.constellix.com/v4` |
| **Auth Header** | `x-cns-security-token: {apiKey}:{signature}:{timestamp}` |
| **TXT Record Creation** | `POST /domains/{domainId}/records/txt` |

**Authentication Header Format:**
```
x-cns-security-token: {apiKey}:{hmac_sha1_signature}:{unix_timestamp}
```

The signature is HMAC-SHA1 of the timestamp using the secret key.

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid API key or signature
- Fix: Verify API key and secret key

**403 Forbidden**
- Cause: API key lacks permissions
- Fix: Check API key permissions in dashboard

**Domain not found**
- Cause: Domain ID doesn't exist
- Fix: Verify domain is added to Constellix

---

*Last updated: 2026-02-04*
