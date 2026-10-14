# Aruba.it DNS

> DNS service from Aruba S.p.A., Italy's leading hosting and domain registrar.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | API Token + Username + Password |
| **Rate Limit** | Contact Aruba for limits |
| **Propagation** | Instant (API) |

**Note:** This integration is for Aruba S.p.A. (Italian hosting/registrar), NOT HPE Aruba Networks.

---

## Setup

### 1. Get API Credentials

1. Log in to [Aruba Business Dashboard](https://admin.arubabusiness.it/)
2. Navigate to **API Settings** or **Developer Tools**
3. Generate an API Token
4. Note your:
   - API Token (ARUBA_TK)
   - Username/Account Key (ARUBA_AK)
   - Password/Account Secret (ARUBA_AS)

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Aruba.it**
3. Enter:
   - API Token
   - Username
   - Password
4. Click **Save**

---

## Required Permissions

- Domain management access
- DNS record creation and deletion

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://admin.arubabusiness.it/ws/LevelDomainBase.asmx` |
| **Auth Method** | SOAP Header with credentials |
| **API Type** | SOAP/XML |
| **TXT Record Creation** | `AddDnsRecord` SOAP action |

**SOAP Header Format:**
```xml
<dom:Credentials>
  <dom:UserName>{username}</dom:UserName>
  <dom:Password>{password}</dom:Password>
  <dom:Token>{api_token}</dom:Token>
</dom:Credentials>
```

---

## Troubleshooting

### Common errors

**Authentication failed**
- Cause: Invalid username, password, or API token
- Fix: Verify all credentials in Aruba Business dashboard

**Domain not found**
- Cause: Domain not registered or not accessible
- Fix: Verify domain exists in your Aruba account

**SOAP fault**
- Cause: Malformed request or server error
- Fix: Check request format and try again

---

*Last updated: 2026-02-04*
