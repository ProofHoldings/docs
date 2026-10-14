# PowerDNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key (X-API-Key header) |
| Rate Limit | Self-hosted (configurable) |
| Propagation | Near instant |

---

## Setup

### 1. Create API Credentials

1. Access your PowerDNS server configuration
2. Enable the API in `pdns.conf`:
   ```
   api=yes
   api-key=your-secret-api-key
   webserver=yes
   webserver-address=0.0.0.0
   webserver-port=8081
   ```
3. Restart PowerDNS service
4. Note your API key and server URL

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "PowerDNS"
4. Enter your Server URL and API Key
5. Click "Verify & Save"

---

## Required Permissions

- API access enabled on PowerDNS server
- Zone management permissions

---

## API Details

**Endpoint:** `{serverUrl}/api/v1` (self-hosted)

**Authentication:**
```
X-API-Key: {apiKey}
```

**Create TXT Record (PATCH zone):**
```bash
PATCH /servers/localhost/zones/{zone}
Content-Type: application/json

{
  "rrsets": [
    {
      "name": "_proof.example.com.",
      "type": "TXT",
      "ttl": 300,
      "changetype": "REPLACE",
      "records": [
        {
          "content": "\"verification-code\"",
          "disabled": false
        }
      ]
    }
  ]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API key | Check api-key in pdns.conf |
| Connection refused | API not enabled | Enable api=yes in configuration |
| Zone not found | Zone not configured | Create zone in PowerDNS first |
| Webserver not running | Wrong port/address | Verify webserver settings in pdns.conf |
