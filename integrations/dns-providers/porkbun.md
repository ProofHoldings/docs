# Porkbun Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key + Secret Key (Request Body) |
| Rate Limit | Standard Porkbun API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Porkbun account
2. Navigate to **Account** -> **API Access**
3. Enable API access for your account
4. Generate a new API key pair
5. Copy both the API Key and Secret Key
6. Enable API access for the specific domain

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Porkbun"
4. Enter your API Key and Secret Key
5. Click "Verify & Save"

---

## Required Permissions

- API access enabled on account
- API access enabled on domain

---

## API Details

**Endpoint:** `https://porkbun.com/api/json/v3`

**Authentication (in request body):**
```json
{
  "apikey": "{apiKey}",
  "secretapikey": "{secretKey}"
}
```

**Create TXT Record:**
```bash
POST /dns/create/{domain}
Content-Type: application/json

{
  "apikey": "{apiKey}",
  "secretapikey": "{secretKey}",
  "type": "TXT",
  "name": "_proof",
  "content": "verification-code",
  "ttl": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid API key | Wrong credentials | Verify API key and secret in Porkbun account |
| API access disabled | Domain API not enabled | Enable API access for the specific domain |
| Domain not found | Domain not in account | Verify domain is registered with Porkbun |
