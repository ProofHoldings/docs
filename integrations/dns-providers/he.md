# Hurricane Electric DNS

> Free DNS hosting service with global anycast network.

---

## Overview

| Property | Value |
|----------|-------|
| **Auth Type** | DDNS Key (per-record) or Username + Password (session-based) |
| **Rate Limit** | Contact HE for limits |
| **Propagation** | Instant (API) |

**Note:** HE DNS uses a form-based/HTML interface. Full API access requires session authentication.

---

## Setup

### Option 1: DDNS Key (Per-Record)

1. Log in to [Hurricane Electric DNS](https://dns.he.net/)
2. Select your zone
3. For each record, click the DDNS icon
4. Enable DDNS and generate a key
5. Note the DDNS key for each record

### Option 2: Account Credentials

1. Log in to [Hurricane Electric DNS](https://dns.he.net/)
2. Use your account:
   - Username (email)
   - Password

### Add to proof.holdings

1. Go to **Settings** > **DNS Providers**
2. Click **Add Provider** > **Hurricane Electric**
3. Enter either:
   - DDNS Key (for updating existing records)
   - OR Username + Password (for full management)
4. Click **Save**

---

## Required Permissions

- DDNS Key: Can only update the specific record
- Account credentials: Full zone management

---

## API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://dns.he.net` |
| **DDNS Update** | `GET /nic/update?hostname={fqdn}&password={key}&txt={value}` |
| **Session Auth** | `POST /index.cgi` with form credentials |
| **TXT Record Creation** | Form POST to `/index.cgi` |

**DDNS Update Response Codes:**
- `good`: Update successful
- `nochg`: No change needed
- `badauth`: Invalid DDNS key
- `nohost`: Record not found

---

## Limitations

- DDNS keys only update existing records
- No native REST API (HTML form-based)
- For verification, use DNS lookup instead of API query

---

## Troubleshooting

### Common errors

**badauth**
- Cause: Invalid DDNS key
- Fix: Regenerate DDNS key for the record

**nohost**
- Cause: Record doesn't exist
- Fix: Create the record manually first, then enable DDNS

**Login failed**
- Cause: Invalid username or password
- Fix: Verify account credentials

---

*Last updated: 2026-02-04*
