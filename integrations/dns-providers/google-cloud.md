# Google Cloud DNS

> Scalable, reliable DNS hosting on Google Cloud Platform.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | Service Account (JWT for OAuth2) |
| **Rate Limit** | 10,000 requests per day per project |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Create a Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Select your project
3. Navigate to **IAM & Admin** > **Service Accounts**
4. Click **Create Service Account**
5. Name it (e.g., "proof-holdings-dns")
6. Click **Create and Continue**

### 2. Assign DNS Permissions

1. Add role: **DNS Administrator** (or custom role)
2. Click **Continue** > **Done**
3. Click on the service account > **Keys** tab
4. Click **Add Key** > **Create new key** > **JSON**
5. Download and save the JSON key file

### 3. Extract Credentials from JSON

From the downloaded JSON file, note:
- `project_id`
- `client_email`
- `private_key`

### 4. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Google Cloud DNS**
3. Enter:
   - Project ID
   - Client Email
   - Private Key
4. Click **Save**

---

## Required Permissions

Service account needs:
- `dns.managedZones.list`
- `dns.managedZones.get`
- `dns.resourceRecordSets.list`
- `dns.resourceRecordSets.create`
- `dns.resourceRecordSets.delete`
- `dns.changes.create`

Or use the predefined **DNS Administrator** role.

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://dns.googleapis.com/dns/v1` |
| **Auth Header** | `Authorization: Bearer {access_token}` |
| **Token URL** | `https://oauth2.googleapis.com/token` |
| **TXT Record Creation** | `POST /projects/{project}/managedZones/{zone}/changes` |

**Create TXT Record (via changes):**
```json
{
  "additions": [
    {
      "name": "_proof.example.com.",
      "type": "TXT",
      "ttl": 300,
      "rrdatas": ["\"verification-code\""]
    }
  ]
}
```

**Note:** Google Cloud DNS requires FQDN with trailing dot. TXT values must be quoted.

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid or expired access token
- Fix: Check service account credentials

**403 Forbidden**
- Cause: Service account lacks permissions
- Fix: Assign DNS Administrator role

**Zone not found**
- Cause: Managed zone doesn't exist
- Fix: Create zone in Google Cloud Console

---

*Last updated: 2026-02-04*
