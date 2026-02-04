# AWS Route 53 Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | Access Key ID + Secret Access Key |
| Rate Limit | 5 requests / second |
| Propagation | Instant |
| Signing | AWS Signature V4 |

---

## Setup

### 1. Create IAM User/Role

1. Go to [AWS IAM Console](https://console.aws.amazon.com/iam/)
2. Create a new IAM user or role
3. Attach a policy with these permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:GetHostedZone",
        "route53:ListResourceRecordSets",
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": "*"
    }
  ]
}
```

For more restrictive access, limit to specific hosted zones:
```json
{
  "Resource": "arn:aws:route53:::hostedzone/ZONE_ID_HERE"
}
```

### 2. Create Access Keys

1. In IAM, go to your user → Security credentials
2. Click "Create access key"
3. Copy both:
   - Access Key ID
   - Secret Access Key (shown once)

### 3. Add to proof.holdings

1. Go to Dashboard → Domains
2. Click your domain → "Connect Provider"
3. Select "AWS Route 53"
4. Enter Access Key ID and Secret Access Key
5. Click "Verify & Save"

---

## Required Permissions

```
route53:ListHostedZones
route53:GetHostedZone
route53:ListResourceRecordSets
route53:ChangeResourceRecordSets
```

Minimal scope: single hosted zone.

---

## API Details

**Endpoint:** `https://route53.amazonaws.com`

**Authentication:** AWS Signature V4

**List Hosted Zones:**
```bash
GET /2013-04-01/hostedzone
```

**Create TXT Record:**
```bash
POST /2013-04-01/hostedzone/{zone_id}/rrset

<?xml version="1.0" encoding="UTF-8"?>
<ChangeResourceRecordSetsRequest xmlns="https://route53.amazonaws.com/doc/2013-04-01/">
  <ChangeBatch>
    <Changes>
      <Change>
        <Action>UPSERT</Action>
        <ResourceRecordSet>
          <Name>_proof.example.com.</Name>
          <Type>TXT</Type>
          <TTL>300</TTL>
          <ResourceRecords>
            <ResourceRecord>
              <Value>"proof-verify=ABC123"</Value>
            </ResourceRecord>
          </ResourceRecords>
        </ResourceRecordSet>
      </Change>
    </Changes>
  </ChangeBatch>
</ChangeResourceRecordSetsRequest>
```

---

## Notes

- Route 53 uses UPSERT action (creates or updates)
- Record names require trailing dot (FQDN format)
- TXT values must be quoted
- Route 53 is a global service but uses `us-east-1` for API signing

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `AccessDenied` | Invalid credentials or insufficient permissions | Check IAM policy |
| `NoSuchHostedZone` | Zone ID incorrect or not accessible | Verify hosted zone exists |
| `InvalidChangeBatch` | Malformed request | Check record format |
| `Throttling` | Rate limit exceeded | Wait and retry |
