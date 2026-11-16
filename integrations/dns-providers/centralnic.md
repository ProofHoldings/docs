# CentralNic Reseller (RRPProxy)

> Domain registrar and DNS services from CentralNic Reseller (formerly RRPProxy).

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | Username + Password |
| **Rate Limit** | Contact CentralNic for limits |
| **Propagation** | Instant (API) |

**Note:** CentralNic acquired RRPProxy. The API is similar to Hexonet/ISPAPI.

---

## Setup

### 1. Get API Credentials

1. Log in to [CentralNic Reseller Control Panel](https://www.rrpproxy.net/)
2. Navigate to **Account** > **API Settings**
3. Note your:
   - Username
   - Password (API password, not account password)
4. For testing, use the OT&E (test) environment

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **CentralNic Reseller**
3. Enter:
   - Username
   - Password
4. Click **Save**

---

## Required Permissions

- DNS Zone management access
- Record creation and deletion permissions

---

## API Details

| Property | Value |
|----------|-------|
| **Production URL** | `https://api.rrpproxy.net/api/call.cgi` |
| **Test URL** | `https://api-ote.rrpproxy.net/api/call.cgi` |
| **Auth Method** | Form POST with s_login and s_pw |
| **TXT Record Creation** | `ModifyDNSZone` command with `addrr0` parameter |

**Request Format:**
```
command=ModifyDNSZone
dnszone=example.com.
addrr0=_proof.example.com. 300 IN TXT "verification-code"
```

**Response Codes:**
- `200`: Success
- `210`: Pending
- `530`: Authentication failed

---

## Troubleshooting

### Common errors

**530 Authentication error**
- Cause: Invalid username or password
- Fix: Verify API credentials

**Zone not found**
- Cause: DNS zone doesn't exist in account
- Fix: Create the zone first or verify zone name

**Invalid RR format**
- Cause: Malformed resource record
- Fix: Check record format follows DNS standards

---

*Last updated: 2026-02-04*
