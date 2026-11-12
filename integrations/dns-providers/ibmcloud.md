# IBM Cloud DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | IAM API Key (Bearer Token) |
| Rate Limit | Standard IBM Cloud limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to IBM Cloud Console
2. Navigate to **Manage** -> **Access (IAM)** -> **API keys**
3. Click **Create an IBM Cloud API key**
4. Name your key and click **Create**
5. Copy the API key immediately (it won't be shown again)
6. Note your DNS Services instance ID from the resource list

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "IBM Cloud DNS"
4. Enter your API Key and Instance ID
5. Click "Verify & Save"

---

## Required Permissions

- DNS Services: Manager or Editor role
- Resource group access for DNS zones

---

## API Details

**IAM Endpoint:** `https://iam.cloud.ibm.com`
**DNS API Endpoint:** `https://api.dns-svcs.cloud.ibm.com/v1`

**Authentication (Token Exchange):**
```bash
POST /identity/token
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ibm:params:oauth:grant-type:apikey&apikey={apiKey}
```

**Create TXT Record:**
```bash
POST /instances/{instanceId}/dnszones/{zoneId}/resource_records
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "name": "_proof.example.com",
  "type": "TXT",
  "ttl": 300,
  "rdata": {
    "text": "verification-code"
  }
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid or expired token | Refresh IAM token or regenerate API key |
| 403 Forbidden | Insufficient permissions | Ensure IAM role includes DNS management |
| Zone not found | Zone not in instance | Verify zone exists in your DNS Services instance |
| Invalid instance ID | Wrong instance ID | Check instance ID in IBM Cloud resource list |
