# Oracle Cloud DNS Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | OCI API Signature (RSA Key) |
| Rate Limit | Standard OCI limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to Oracle Cloud Console
2. Navigate to **Identity & Security** -> **Users**
3. Select your user and go to **API Keys**
4. Click **Add API Key**
5. Generate or upload an RSA key pair
6. Download the private key and note the fingerprint
7. Note your tenancy OCID, user OCID, and compartment OCID

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Oracle Cloud DNS"
4. Enter your Tenancy OCID, User OCID, Fingerprint, Private Key, and Region
5. Click "Verify & Save"

---

## Required Permissions

- DNS Zone Management
- DNS Record Management

---

## API Details

**Endpoint:** `https://dns.{region}.oraclecloud.com/20180115`

**Authentication (OCI Signature):**
```
Authorization: Signature version="1",keyId="{tenancyOcid}/{userOcid}/{fingerprint}",algorithm="rsa-sha256",headers="(request-target) date host",signature="{signature}"
```

**Create TXT Record:**
```bash
POST /zones/{zoneName}/records/_proof.{zoneName}/TXT
Content-Type: application/json

{
  "items": [
    {
      "domain": "_proof.example.com",
      "rtype": "TXT",
      "ttl": 300,
      "rdata": "verification-code"
    }
  ]
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid signature | Verify private key and fingerprint match |
| 404 Zone not found | Zone not in compartment | Check zone exists in the specified compartment |
| 403 Forbidden | Insufficient IAM permissions | Add DNS management policies to user |
| Invalid key | Wrong key format | Ensure private key is in PEM format |
