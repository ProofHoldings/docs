# INWX Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Username + Password (JSON-RPC) |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your INWX account
2. Navigate to **Account Settings**
3. Enable API access if not already enabled
4. Note your INWX username and password
5. Optionally set up a separate API password

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "INWX"
4. Enter your Username and Password
5. Click "Verify & Save"

---

## Required Permissions

- Nameserver/DNS management access

---

## API Details

**Endpoint:** `https://api.domrobot.com/jsonrpc/`

**Authentication:**
```json
{
  "method": "account.login",
  "params": {
    "user": "{username}",
    "pass": "{password}"
  }
}
```

**Create TXT Record:**
```json
{
  "method": "nameserver.createRecord",
  "params": {
    "user": "{username}",
    "pass": "{password}",
    "domain": "example.com",
    "name": "_proof",
    "type": "TXT",
    "content": "verification-code",
    "ttl": 300
  }
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| Error code 2200 | Authentication failed | Verify username and password |
| Error code 2303 | Domain not found | Ensure domain is managed by INWX nameservers |
| Invalid credentials | Wrong login details | Check credentials in INWX dashboard |
