# Alibaba Cloud DNS (Alidns)

> DNS service from Alibaba Cloud with global anycast network coverage.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | AccessKey ID + Secret (HMAC-SHA1 signature) |
| **Rate Limit** | 100 requests per minute per account |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Get AccessKey Credentials

1. Log in to [Alibaba Cloud Console](https://www.alibabacloud.com/console)
2. Click your profile > **AccessKey Management**
3. Click **Create AccessKey**
4. Save the AccessKey ID and AccessKey Secret
   - **Important:** The secret is shown only once
5. For better security, create a RAM user with limited permissions

### 2. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Alibaba Cloud DNS**
3. Enter:
   - AccessKey ID
   - AccessKey Secret
4. Click **Save**

---

## Required Permissions

For RAM users, attach these policies:
- `AliyunDNSFullAccess` - Full DNS management
- Or create a custom policy with:
  - `alidns:DescribeDomains`
  - `alidns:DescribeDomainRecords`
  - `alidns:AddDomainRecord`
  - `alidns:DeleteDomainRecord`

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://alidns.aliyuncs.com` |
| **Auth Method** | Query parameter signature (HMAC-SHA1) |
| **TXT Record Creation** | `Action=AddDomainRecord&Type=TXT` |

**Request Parameters:**
- `Format`: JSON
- `Version`: 2015-01-09
- `AccessKeyId`: Your access key
- `SignatureMethod`: HMAC-SHA1
- `Signature`: Calculated signature

---

## Troubleshooting

### Common errors

**InvalidAccessKeyId.NotFound**
- Cause: AccessKey ID does not exist
- Fix: Verify the AccessKey ID is correct

**SignatureDoesNotMatch**
- Cause: Incorrect signature calculation
- Fix: Check AccessKey Secret is correct

**InvalidDomainName.NoExist**
- Cause: Domain not in Alibaba Cloud DNS
- Fix: Add domain to Alibaba Cloud DNS first

---

*Last updated: 2026-02-04*
