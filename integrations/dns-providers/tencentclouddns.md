# Tencent Cloud DNS (DNSPod) Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | TC3-HMAC-SHA256 Signature (Secret ID + Key) |
| Rate Limit | Standard Tencent Cloud limits |
| Propagation | 1-5 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to Tencent Cloud Console
2. Navigate to **Access Management** -> **API Keys**
3. Click **Create Key**
4. Copy the SecretId and SecretKey

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Tencent Cloud DNS"
4. Enter your Secret ID and Secret Key
5. Click "Verify & Save"

---

## Required Permissions

- DNSPod full access or QcloudDNSPodFullAccess policy

---

## API Details

**Endpoint:** `https://dnspod.tencentcloudapi.com`

**Authentication (TC3-HMAC-SHA256):**
```
Authorization: TC3-HMAC-SHA256 Credential={secretId}/{date}/dnspod/tc3_request, SignedHeaders=content-type;host;x-tc-action, Signature={signature}
X-TC-Action: CreateRecord
X-TC-Version: 2021-03-23
X-TC-Region: ap-guangzhou
X-TC-Timestamp: {timestamp}
```

**Create TXT Record:**
```bash
POST /
Content-Type: application/json

{
  "Domain": "example.com",
  "SubDomain": "_proof",
  "RecordType": "TXT",
  "RecordLine": "default",
  "Value": "verification-code",
  "TTL": 300
}
```

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| AuthFailure | Invalid credentials | Verify SecretId and SecretKey |
| SignatureExpired | Timestamp too old | Ensure server time is synchronized |
| InvalidParameter.DomainNotExist | Domain not found | Add domain to DNSPod first |
| UnauthorizedOperation | Insufficient permissions | Attach DNSPod access policy to account |
