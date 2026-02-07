---
title: Smart Reuse
lastUpdated: "2026-02-06"
status: current
---

# Smart Reuse

> Skip re-verification when a user already has a valid proof

---

## Overview

When a logged-in user opens a verification request, Smart Reuse checks whether they have already verified the requested assets. If a valid proof exists, the user can verify with a single click instead of repeating the full challenge flow.

This is useful for returning users, multi-step onboarding, and any scenario where the same user verifies the same asset across different requests.

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. User opens verification request                                         │
│  2. Client calls GET /proofs → finds existing proof for +37069199199        │
│  3. can_reuse: true → show "Use existing proof" button                      │
│  4. Client calls POST /auto-verify → asset verified instantly               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Two API calls.** No challenge, no user action beyond a click.

---

## Endpoints

### Check Existing Proofs

`GET /api/v1/verify/request/:id/proofs`

Returns which assets the logged-in user has already verified. Requires a JWT session cookie (optional auth -- unauthenticated users get an empty response).

#### Response (Authenticated)

```json
{
  "request_id": "507f1f77bcf86cd799439011",
  "user_logged_in": true,
  "assets": [
    {
      "asset_index": 0,
      "type": "phone",
      "identifier": "+37069199199",
      "verified_at": "2026-01-15T10:00:00Z",
      "channel": "whatsapp",
      "can_reuse": true
    },
    {
      "asset_index": 1,
      "type": "email",
      "identifier": "user@example.com",
      "verified_at": "2025-11-01T08:00:00Z",
      "channel": "email",
      "can_reuse": false,
      "reason": "Proof is older than validity requirement"
    }
  ],
  "reusable_count": 1,
  "total_required": 2
}
```

#### Response (Unauthenticated)

```json
{
  "request_id": "507f1f77bcf86cd799439011",
  "user_logged_in": false,
  "assets": [],
  "reusable_count": 0,
  "total_required": 2
}
```

#### Asset Fields

| Field | Type | Description |
|-------|------|-------------|
| `asset_index` | number | Position in the request's `assets` array |
| `type` | string | `phone`, `email`, `domain` |
| `identifier` | string | The verified identifier |
| `verified_at` | string | ISO 8601 timestamp of original verification |
| `channel` | string | Channel used for original verification |
| `can_reuse` | boolean | Whether the proof meets validity requirements |
| `reason` | string | Present when `can_reuse` is false |

---

### Auto-Verify (Reuse Proof)

`POST /api/v1/verify/request/:id/assets/:assetIndex/auto-verify`

Verify an asset using an existing proof. Requires authentication.

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reuse_proof` | boolean | Yes | Must be `true` for proof reuse |
| `identifier` | string | No | Which identifier to reuse (required when asset has no pre-set identifier) |

```bash
curl -X POST https://api.proof.holdings/api/v1/verify/request/507f1f77bcf86cd799439011/assets/0/auto-verify \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "reuse_proof": true,
    "identifier": "+37069199199"
  }'
```

#### Response (Success)

```json
{
  "status": "verified",
  "verified_at": "2026-02-06T12:00:00Z",
  "method": "reused_proof",
  "original_verified_at": "2026-01-15T10:00:00Z",
  "request_status": "completed",
  "progress": { "verified": 2, "total": 2, "required": 2 }
}
```

#### Error Responses

| Code | HTTP | Description |
|------|------|-------------|
| `unauthorized` | 401 | User not logged in |
| `no_existing_proof` | 400 | User has no proof for this asset |
| `proof_too_old` | 400 | Proof does not meet validity requirement |
| `already_verified` | 400 | Asset is already verified |
| `request_expired` | 400 | Verification request has expired |

---

## Validity Requirements

Verification requests can specify a maximum proof age via `validity_requirement`. This controls whether an existing proof is fresh enough to reuse.

| Format | Meaning | Example |
|--------|---------|---------|
| `24h` | Hours | Proof must be < 24 hours old |
| `7d` | Days | Proof must be < 7 days old |
| `1w` | Weeks | Proof must be < 1 week old |
| `1m` | Months | Proof must be < 1 month old |

When creating a verification request, set `validity_requirement` to enforce freshness:

```bash
curl -X POST https://api.proof.holdings/api/v1/verification-requests \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "assets": [
      { "type": "phone", "identifier": "+37069199199" },
      { "type": "email", "identifier": "user@example.com" }
    ],
    "validity_requirement": "30d"
  }'
```

If no `validity_requirement` is set, any non-expired proof can be reused.

---

## Integration Example

```typescript
import { Proof } from '@proof/sdk';

const proof = new Proof('pk_live_your_key');

// 1. Create a verification request with validity requirement
const request = await proof.verificationRequests.create({
  assets: [
    { type: 'phone', identifier: '+37069199199' },
    { type: 'email', identifier: 'user@example.com' },
  ],
  validity_requirement: '30d',
});

// 2. On the verification page, check for existing proofs
const existing = await fetch(
  `https://api.proof.holdings/api/v1/verify/request/${request.id}/proofs`,
  { credentials: 'include' }
).then(r => r.json());

// 3. For each reusable proof, auto-verify
for (const asset of existing.assets.filter(a => a.can_reuse)) {
  await fetch(
    `https://api.proof.holdings/api/v1/verify/request/${request.id}/assets/${asset.asset_index}/auto-verify`,
    {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        reuse_proof: true,
        identifier: asset.identifier,
      }),
    }
  );
}
```

---

## Supported Asset Types

Smart Reuse works with all asset types that store proofs on the user account:

| Type | Source | Match Logic |
|------|--------|-------------|
| `phone` | User's verified phones | Exact phone number match (normalized) |
| `email` | User's verified emails | Exact email match (normalized) |
| `domain` | User's verified domains | Exact domain match (normalized) |

---

## Notes

- Smart Reuse only works for **logged-in users** with a JWT session
- A new proof token is generated for each reuse (linked to the new verification request)
- The `method: "reused_proof"` field in the response distinguishes reused proofs from fresh verifications
- Webhooks fire normally when auto-verify completes a request
