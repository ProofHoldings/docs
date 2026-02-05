# NameSilo Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key (Query Parameter) |
| Rate Limit | Standard API limits |
| Propagation | 5-15 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your NameSilo account
2. Navigate to **API Manager** in account settings
3. Generate a new API key
4. Copy the API key

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "NameSilo"
4. Enter your API Key
5. Click "Verify & Save"

---

## Required Permissions

- API key with DNS management access

---

## API Details

**Endpoint:** `https://www.namesilo.com/api`

**Authentication (Query Parameter):**
```
?version=1&type=xml&key={apiKey}
```

**Create TXT Record:**
```bash
GET /dnsAddRecord?version=1&type=xml&key={apiKey}&domain=example.com&rrtype=TXT&rrhost=_proof&rrvalue=verification-code&rrttl=3600
```

**Note:** NameSilo uses XML responses and GET requests with query parameters.

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid API key | Wrong or expired key | Regenerate API key in NameSilo account |
| Domain not found | Domain not in account | Verify domain is registered with NameSilo |
| Operation failed | DNS not managed by NameSilo | Ensure domain uses NameSilo nameservers |
