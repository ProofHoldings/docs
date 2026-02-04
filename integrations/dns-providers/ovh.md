# OVH Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Application Key + Secret + Consumer Key (Signature) |
| Rate Limit | Standard OVH API limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Go to [OVH API Console](https://eu.api.ovh.com/createToken/)
2. Log in with your OVH account
3. Set application name and description
4. Request the following rights:
   - `GET /domain/zone/*`
   - `POST /domain/zone/*`
   - `DELETE /domain/zone/*`
5. Copy Application Key, Application Secret, and Consumer Key

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "OVH"
4. Enter your Application Key, Application Secret, and Consumer Key
5. Click "Verify & Save"

---

## Required Permissions

- `/domain/zone/*` - GET, POST, DELETE

---

## API Details

**Endpoint:** `https://eu.api.ovh.com/1.0` (or regional endpoint)

**Authentication (Signature):**
```
X-Ovh-Application: {applicationKey}
X-Ovh-Consumer: {consumerKey}
X-Ovh-Timestamp: {timestamp}
X-Ovh-Signature: $1${sha1(applicationSecret+consumerKey+method+url+body+timestamp)}
```

**Create TXT Record:**
```bash
POST /domain/zone/{zoneName}/record
Content-Type: application/json

{
  "fieldType": "TXT",
  "subDomain": "_proof",
  "target": "verification-code",
  "ttl": 300
}
```

**Refresh Zone:**
```bash
POST /domain/zone/{zoneName}/refresh
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid signature | Wrong credentials or timestamp | Verify all three keys and server time sync |
| 403 Forbidden | Insufficient permissions | Regenerate token with correct permissions |
| Zone not found | Domain not in OVH account | Verify domain is managed by OVH |
