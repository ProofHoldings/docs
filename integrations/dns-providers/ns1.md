# NS1 Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key (X-NSONE-Key header) |
| Rate Limit | Standard NS1 API limits |
| Propagation | Near instant (Anycast) |

---

## Setup

### 1. Create API Credentials

1. Log in to your NS1 portal
2. Navigate to **Account** -> **API Keys**
3. Click **Add Key**
4. Configure permissions (DNS Zone management)
5. Copy the generated API key

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "NS1"
4. Enter your API Key
5. Click "Verify & Save"

---

## Required Permissions

- DNS Zones: Read/Write
- DNS Records: Read/Write

---

## API Details

**Endpoint:** `https://api.nsone.net/v1`

**Authentication:**
```
X-NSONE-Key: {apiKey}
```

**Create TXT Record:**
```bash
PUT /zones/{zone}/_proof.{zone}/TXT
Content-Type: application/json

{
  "zone": "example.com",
  "domain": "_proof.example.com",
  "type": "TXT",
  "ttl": 300,
  "answers": [
    {
      "answer": ["verification-code"]
    }
  ]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API key | Regenerate API key in NS1 portal |
| 403 Forbidden | Insufficient permissions | Ensure key has DNS write permissions |
| Zone not found | Domain not in account | Add zone to NS1 first |
