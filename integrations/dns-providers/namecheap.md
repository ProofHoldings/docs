# Namecheap Integration

## Overview

| Property | Value |
|----------|-------|
| Auth Type | API Key + Username (Query Parameters) |
| Rate Limit | Standard API limits |
| Propagation | 5-30 minutes |

---

## Setup

### 1. Create API Credentials

1. Log in to your Namecheap account
2. Navigate to **Profile** -> **Tools** -> **API Access**
3. Enable API access (requires account balance or purchases)
4. Copy your API Key
5. Whitelist your IP address
6. Note your Namecheap username

### 2. Add to proof.holdings

1. Go to Dashboard -> Domains
2. Click your domain -> "Connect Provider"
3. Select "Namecheap"
4. Enter your Username, API Key, and Client IP
5. Click "Verify & Save"

---

## Required Permissions

- API access enabled
- IP address whitelisted
- Domain management permissions

---

## API Details

**Endpoint:** `https://api.namecheap.com/xml.response`

**Authentication (Query Parameters):**
```
?ApiUser={username}&ApiKey={apiKey}&UserName={username}&ClientIp={clientIp}
```

**Set DNS Hosts (including TXT record):**
```bash
GET /xml.response?ApiUser={username}&ApiKey={apiKey}&UserName={username}&ClientIp={clientIp}&Command=namecheap.domains.dns.setHosts&SLD={secondLevelDomain}&TLD={topLevelDomain}&HostName1=_proof&RecordType1=TXT&Address1=verification-code&TTL1=300
```

**Note:** Namecheap uses XML API and requires setting all hosts at once.

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| API access denied | IP not whitelisted | Add your IP to API whitelist in Namecheap |
| Invalid API key | Wrong credentials | Verify API key in Profile -> Tools |
| Command failed | API not enabled | Enable API access in account settings |
| Domain not found | Domain not in account | Verify domain ownership in Namecheap |
