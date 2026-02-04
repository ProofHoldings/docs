# Dyn Managed DNS

> Enterprise DNS service, now part of Oracle Cloud Infrastructure.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | Customer Name + Username + Password (Session-based) |
| **Rate Limit** | Contact Oracle for limits |
| **Propagation** | Instant (API) |

**Note:** Dyn was acquired by Oracle, but the legacy API is still available.

---

## Setup

### 1. Get API Credentials

1. Log in to [Dyn Dashboard](https://manage.dynect.net/)
2. Go to **Users & Groups**
3. Create or select a user with API access
4. Note your:
   - Customer Name
   - Username
   - Password

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Dyn**
3. Enter:
   - Customer Name
   - Username
   - Password
4. Click **Save**

---

## Required Permissions

User needs:
- Zone Read permission
- Zone Write permission
- Record management permissions

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.dynect.net/REST` |
| **Auth Method** | Session token via `POST /Session/` |
| **Auth Header** | `Auth-Token: {session_token}` |
| **TXT Record Creation** | `POST /TXTRecord/{zone}/{fqdn}/` |

**Session Creation:**
```json
{
  "customer_name": "your_customer",
  "user_name": "your_username",
  "password": "your_password"
}
```

**Important:** Changes must be published with `PUT /Zone/{zone}/` with `{"publish": true}`.

---

## Troubleshooting

### Common errors

**Login failed**
- Cause: Invalid customer name, username, or password
- Fix: Verify credentials in Dyn dashboard

**Zone not found**
- Cause: Zone doesn't exist or no access
- Fix: Check zone name and user permissions

**Changes not visible**
- Cause: Zone not published after changes
- Fix: Changes require a publish operation

---

*Last updated: 2026-02-04*
