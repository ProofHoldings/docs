# Netcup Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Customer Number + API Key + API Password (Session-based) |
| Rate Limit | Standard API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Netcup Customer Control Panel (CCP)
2. Navigate to **Master Data** -> **API**
3. Enable API access and generate API key
4. Set an API password
5. Note your customer number

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Netcup"
4. Enter your Customer Number, API Key, and API Password
5. Click "Verify & Save"

---

## Required Permissions

- API access enabled
- DNS management permissions

---

## API Details

**Endpoint:** `https://ccp.netcup.net/run/webservice/servers/endpoint.php?JSON`

**Authentication (Login Request):**
```json
{
  "action": "login",
  "param": {
    "customernumber": "{customerNumber}",
    "apikey": "{apiKey}",
    "apipassword": "{apiPassword}"
  }
}
```

**Create TXT Record:**
```json
{
  "action": "updateDnsRecords",
  "param": {
    "customernumber": "{customerNumber}",
    "apikey": "{apiKey}",
    "apisessionid": "{sessionId}",
    "domainname": "example.com",
    "dnsrecordset": {
      "dnsrecords": [
        {
          "hostname": "_proof",
          "type": "TXT",
          "destination": "verification-code",
          "priority": 0
        }
      ]
    }
  }
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| Login failed | Invalid credentials | Verify customer number, API key, and password |
| Session expired | Session timeout | Re-authenticate to get new session ID |
| Domain not found | Domain not in account | Verify domain is managed by Netcup |
