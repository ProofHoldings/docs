# Hexonet (ISPAPI)

> Domain registrar and DNS services via ISPAPI.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | Username + Password |
| **Rate Limit** | Contact Hexonet for limits |
| **Propagation** | Instant (API) |

**Note:** Hexonet uses a command-based API similar to EPP.

---

## Setup

### 1. Get API Credentials

1. Log in to [Hexonet Control Panel](https://www.hexonet.net/)
2. Go to **Account** > **API Access**
3. Note your:
   - Username
   - Password
4. For testing, use entity `1234` (OT&E)
5. For production, use entity `54cd`

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Hexonet**
3. Enter:
   - Username
   - Password
4. Click **Save**

---

## Required Permissions

- DNS zone management
- Record creation and deletion

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.ispapi.net/api/call.cgi` |
| **Auth Method** | Form POST with `s_login` and `s_pw` |
| **TXT Record Creation** | `ModifyDNSZone` command with `addrr0` parameter |

**Request Format:**
```
s_login={username}
s_pw={password}
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
- Fix: Create zone first

**Invalid RR format**
- Cause: Malformed resource record
- Fix: Ensure record follows DNS format standards

---

*Last updated: 2026-02-04*
