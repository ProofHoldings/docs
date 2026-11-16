# eNom

> Domain registrar with reseller services and DNS hosting.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | UID + Password (query parameters) |
| **Rate Limit** | Contact eNom for limits |
| **Propagation** | Instant (API) |

**Note:** eNom uses a query-string based API. Record management requires setting ALL records at once.

---

## Setup

### 1. Get API Credentials

1. Log in to [eNom Control Panel](https://www.enom.com/login.aspx)
2. Go to **Account** > **API Settings**
3. Enable API access
4. Note your:
   - UID (account ID)
   - API Password (not account password)

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **eNom**
3. Enter:
   - UID
   - Password
4. Click **Save**

---

## Required Permissions

- Domain management access
- DNS host record management

---

## API Details

| Property | Value |
|----------|-------|
| **Production URL** | `https://reseller.enom.com/interface.asp` |
| **Test URL** | `https://resellertest.enom.com/interface.asp` |
| **Auth Method** | Query parameters `uid` and `pw` |
| **TXT Record Creation** | `command=SetHosts` (sets ALL records) |

**Important:** The `SetHosts` command replaces ALL host records. You must:
1. Get existing records with `GetHosts`
2. Add your new record to the list
3. Set all records with `SetHosts`

**Request Format:**
```
?uid={uid}&pw={password}&command=SetHosts
&sld=example&tld=com
&HostName1=_proof&RecordType1=TXT&Address1=verification-code
```

---

## Troubleshooting

### Common errors

**Authentication failed**
- Cause: Invalid UID or API password
- Fix: Verify credentials in eNom control panel

**Domain not found**
- Cause: Domain not in eNom account
- Fix: Verify domain registration and ownership

**Invalid TLD**
- Cause: Compound TLD parsing issue
- Fix: Ensure correct SLD/TLD split (e.g., example/co.uk)

---

*Last updated: 2026-02-04*
