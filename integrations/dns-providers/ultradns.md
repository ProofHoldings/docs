# UltraDNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | OAuth2 (Username + Password) |
| Rate Limit | Standard UltraDNS limits |
| Propagation | Near instant |

---

## Setup

### 1. Create API Credentials

1. Log in to UltraDNS Portal
2. Navigate to **Users & Groups**
3. Note your username and password
4. Ensure your user has API access enabled

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "UltraDNS"
4. Enter your Username and Password
5. Click "Verify & Save"

---

## Required Permissions

- Zone read/write access
- Record management permissions

---

## API Details

**Endpoint:** `https://api.ultradns.com`

**Authentication (OAuth2 Token):**
```bash
POST /authorization/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&username={username}&password={password}
```

**Response:**
```json
{
  "access_token": "{accessToken}",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

**Create TXT Record:**
```bash
POST /v3/zones/{zoneName}/rrsets/TXT/_proof
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "ttl": 300,
  "rdata": ["verification-code"]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid credentials | Verify username and password |
| Token expired | Access token timeout | Re-authenticate to get new token |
| Zone not found | Zone not in account | Verify zone exists in UltraDNS |
| 403 Forbidden | Insufficient permissions | Check user has zone management access |
