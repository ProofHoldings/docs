# ClouDNS

> Global anycast DNS hosting service with DDoS protection.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | Auth ID + Password |
| **Rate Limit** | Varies by plan |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get API Credentials

1. Log in to [ClouDNS Control Panel](https://www.cloudns.net/main/)
2. Go to **API Settings** or **Reseller API**
3. Note your:
   - **Auth ID** (for main account) or **Sub-auth ID** (for sub-users)
   - **Auth Password**
4. Enable API access if not already enabled

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **ClouDNS**
3. Enter:
   - Auth ID
   - Auth Password
   - Check "Sub-user" if using sub-auth credentials
4. Click **Save**

---

## Required Permissions

- DNS zone management
- Record creation and deletion
- API access enabled on account

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://api.cloudns.net` |
| **Auth Method** | Query parameters (auth-id/sub-auth-id + auth-password) |
| **TXT Record Creation** | `POST /dns/add-record.json` |

**Authentication Parameters:**
- `auth-id`: Main account ID
- `sub-auth-id`: Sub-user ID (alternative)
- `sub-auth-user`: Sub-user email (alternative)
- `auth-password`: API password

---

## Troubleshooting

### Common errors

**Failed: Wrong authentication**
- Cause: Invalid auth ID or password
- Fix: Verify credentials in ClouDNS control panel

**Failed: Invalid domain name**
- Cause: Domain not found in account
- Fix: Add domain to ClouDNS first

**Failed: API not enabled**
- Cause: API access disabled for account
- Fix: Enable API in account settings

---

*Last updated: 2026-02-04*
