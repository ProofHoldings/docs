# Azure DNS

> Microsoft Azure's DNS hosting service with global anycast network.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | Service Principal (OAuth2) |
| **Rate Limit** | 500 requests per 5 minutes |
| **Propagation** | Instant (API) |

---

## Setup

### 1. Create a Service Principal

1. Log in to [Azure Portal](https://portal.azure.com/)
2. Open **Azure Active Directory** > **App registrations**
3. Click **New registration**
4. Enter a name (e.g., "proof-holdings-dns")
5. Click **Register**
6. Note the **Application (client) ID** and **Directory (tenant) ID**
7. Go to **Certificates & secrets** > **New client secret**
8. Create a secret and copy the value immediately

### 2. Assign DNS Permissions

1. Go to your DNS Zone resource
2. Click **Access control (IAM)** > **Add role assignment**
3. Select **DNS Zone Contributor** role
4. Assign to your service principal

### 3. Note Your Resource Details

- **Subscription ID**: Found in Subscription settings
- **Resource Group**: The group containing your DNS zone

### 4. Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Azure DNS**
3. Enter:
   - Tenant ID
   - Client ID
   - Client Secret
   - Subscription ID
   - Resource Group
4. Click **Save**

---

## Required Permissions

The service principal needs:
- **DNS Zone Contributor** role on the DNS zone(s)

Alternatively, create a custom role with:
- `Microsoft.Network/dnsZones/read`
- `Microsoft.Network/dnsZones/TXT/*`
- `Microsoft.Network/dnsZones/CNAME/*`

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Network/dnsZones/` |
| **Auth Header** | `Authorization: Bearer {access_token}` |
| **API Version** | 2018-05-01 |
| **TXT Record Creation** | `PUT /zones/{zone}/TXT/{name}` |

---

## Troubleshooting

### Common errors

**401 Unauthorized**
- Cause: Invalid or expired credentials
- Fix: Check client ID, client secret, and tenant ID

**403 Forbidden**
- Cause: Service principal lacks permissions
- Fix: Assign DNS Zone Contributor role

**404 Not Found**
- Cause: Zone doesn't exist in specified resource group
- Fix: Verify zone name and resource group

---

*Last updated: 2026-02-04*
