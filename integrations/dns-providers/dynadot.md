# Dynadot

> Domain registrar with integrated DNS hosting services.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Key |
| **Rate Limit** | 1 request at a time (no concurrent requests) |
| **Propagation** | Instant (API) |

**Important:** Dynadot's `set_dns2` command overwrites ALL DNS records by default. Use `add_dns_to_current_setting=1` to append records.

---

## Setup

### 1. Get API Key

1. Log in to [Dynadot](https://www.dynadot.com/)
2. Go to **My Account** > **API**
3. Enable API access if not already enabled
4. Copy your API Key

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Dynadot**
3. Enter:
   - API Key
4. Click **Save**

---

## Required Permissions

- API access enabled on account
- Domain management access

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.dynadot.com/api3.json` |
| **Auth Method** | Query parameter `key={api_key}` |
| **TXT Record Creation** | `command=set_dns2` with `add_dns_to_current_setting=1` |

**Add TXT Record:**
```
?key={api_key}
&command=set_dns2
&domain=example.com
&add_dns_to_current_setting=1
&subdomain0=_proof
&sub_record_type0=txt
&sub_record0=verification-code
```

**Limits:**
- 20 main domain TXT records
- 100 subdomain records

---

## Troubleshooting

### Common errors

**Invalid API key**
- Cause: API key is incorrect or disabled
- Fix: Verify API key in Dynadot account

**Domain not found**
- Cause: Domain not in Dynadot account
- Fix: Verify domain registration

**Concurrent request error**
- Cause: Multiple API calls at same time
- Fix: Dynadot only allows 1 request at a time

---

*Last updated: 2026-02-04*
