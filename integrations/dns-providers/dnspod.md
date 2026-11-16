# DNSPod

> DNS service by Tencent Cloud, popular in China with global coverage.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Token (ID + Token format) |
| **Rate Limit** | Varies by account level |
| **Propagation** | Instant (API) |

**Note:** DNSPod has two versions:
- **China:** `https://dnsapi.cn` (default)
- **International:** `https://api.dnspod.com`

---

## Setup

### 1. Get API Token

1. Log in to [DNSPod Console](https://console.dnspod.cn/) or [International Console](https://www.dnspod.com/)
2. Go to **Account** > **API Token**
3. Click **Create Token**
4. Note the Token ID and Token value
5. Format: `{token_id},{token_value}`

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **DNSPod**
3. Enter:
   - Login Token (format: `id,token`)
   - Check "International" if using international version
4. Click **Save**

---

## Required Permissions

API tokens can be scoped to:
- Specific domains
- Full account access

---

## API Details

| Property | Value |
|----------|-------|
| **China URL** | `https://dnsapi.cn` |
| **International URL** | `https://api.dnspod.com` |
| **Auth Method** | POST body with `login_token` |
| **TXT Record Creation** | `POST /Record.Create` |

**Request Body:**
```
login_token=123456,abcdef
format=json
domain=example.com
sub_domain=_proof
record_type=TXT
record_line=default
value=verification-code
ttl=300
```

---

## Troubleshooting

### Common errors

**Code -1: Login failed**
- Cause: Invalid login token
- Fix: Verify token format is `id,token`

**Domain not exist**
- Cause: Domain not in DNSPod account
- Fix: Add domain to DNSPod first

**Record already exists**
- Cause: Duplicate record
- Fix: Delete existing record or use different subdomain

---

*Last updated: 2026-02-04*
