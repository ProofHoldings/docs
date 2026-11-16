---
title: Multi-Profile System
lastUpdated: "2026-10-26"
status: current
---

# Multi-Profile System

> Create multiple public profiles with independent settings and verified assets

---

## Overview

A public profile is the public face of a sub-account: each sub-account has one. An account can have several sub-accounts, each with its own display name, bio, theme and choice of which verified phones, emails and domains to show — for example one per brand, or one personal and one for a business.

Each profile gets a unique shareable URL (`/p/{profileId}`) and can optionally claim a username, shown at `/u/{username}`.

---

## Endpoints

Profile endpoints accept a dashboard session cookie or an API key with the `profiles:read` /
`profiles:write` scope, and act on the caller's own account. With a dashboard session, `PATCH`,
`DELETE` and set-primary first need a 2FA step-up (`403 2fa_required` otherwise); an API key does not.

Base path: `https://api.proof.holdings/api/v1`

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/me/profiles` | List all profiles |
| `POST` | `/me/profiles` | Create new profile |
| `GET` | `/me/profiles/:profileId` | Get specific profile |
| `PATCH` | `/me/profiles/:profileId` | Update profile |
| `DELETE` | `/me/profiles/:profileId` | Delete profile |
| `POST` | `/me/profiles/:profileId/primary` | Set as primary profile |

Three further endpoints are under `/profiles`, not `/me`:

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/profiles/me/assets` | List your verified assets and their `id`s (the `asset_id` below) |
| `PUT` | `/profiles/profiles/:profileId/proofs` | Update profile's public proofs |
| `POST` | `/profiles/me/username` | Claim a username (dashboard session only) |

---

## Create Profile

`POST /api/v1/me/profiles`

Each plan allows a set number of profiles; creating one more answers `403 plan_limit_reached`, with
the plan's `limit` and your `current` count.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `display_name` | string | No | Display name (max 100 chars) |
| `bio` | string | No | Profile bio (max 500 chars) |
| `avatar_url` | string | No | HTTPS URL or base64 data URI |
| `business_name` | string | No | Business name (max 100 chars) |
| `is_business` | boolean | No | Whether this is a business profile |
| `is_public` | boolean | No | Whether the profile is publicly visible |
| `is_primary` | boolean | No | Set as the primary profile |
| `theme` | object | No | Visual theme (see Theme Object) |
| `email_theme` | string | No | `dark` or `light` — how this sub-account's emails look (unset renders as `dark`) |
| `support_email` | string | No | Where recipients of this sub-account's messages can reach it; fills `{{support_email}}` |

```bash
curl -X POST https://api.proof.holdings/api/v1/me/profiles \
  -H "Cookie: token=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "display_name": "team.management",
    "bio": "Verified contacts for team.management",
    "is_business": true,
    "business_name": "team.management",
    "is_public": true,
    "theme": {
      "primary_color": "#3B82F6",
      "background_type": "gradient",
      "gradient_start": "#1E3A5F",
      "gradient_end": "#0F172A",
      "font_style": "modern"
    }
  }'
```

### Response

```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0e1",
  "profile_id": "k7x2m9p4",
  "display_name": "team.management",
  "bio": "Verified contacts for team.management",
  "is_business": true,
  "business_name": "team.management",
  "is_public": true,
  "is_primary": false,
  "email_theme": null,
  "support_email": null,
  "verification_level": "basic",
  "public_proofs": [],
  "custom_links": [],
  "theme": {
    "primary_color": "#3B82F6",
    "background_type": "gradient",
    "gradient_start": "#1E3A5F",
    "gradient_end": "#0F172A",
    "font_style": "modern"
  },
  "view_count": 0,
  "created_at": "2026-02-06T12:00:00Z",
  "updated_at": "2026-02-06T12:00:00Z"
}
```

---

## Update Profile

`PATCH /api/v1/me/profiles/:profileId`

Accepts any subset of the create fields, plus additional settings:

| Parameter | Type | Description |
|-----------|------|-------------|
| `show_verification_dates` | boolean | Show when each asset was verified |
| `show_proof_channels` | boolean | Show which channel was used |
| `custom_links` | array | Up to 20 custom links (see below) |

`support_email` also accepts `null`, which clears it.

### Custom Links

Each link in the `custom_links` array:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Link title (max 100 chars) |
| `url` | string | Yes | Link URL (max 500 chars) |
| `icon` | string | No | Icon identifier (max 50 chars) |
| `is_visible` | boolean | No | Whether link is shown publicly |
| `display_order` | number | No | Sort order (0-based) |

```bash
curl -X PATCH https://api.proof.holdings/api/v1/me/profiles/64a1b2c3d4e5f6a7b8c9d0e1 \
  -H "Authorization: Bearer pk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "bio": "Updated bio for team.management",
    "custom_links": [
      { "title": "Website", "url": "https://team.management", "display_order": 0 },
      { "title": "Support", "url": "https://team.management/support", "display_order": 1 }
    ]
  }'
```

---

## List Your Assets

`GET /api/v1/profiles/me/assets`

Lists every active verified asset on your account. Its `id` is the `asset_id` that
[Update Public Proofs](#update-public-proofs) takes. `is_public` says whether the asset is already
in the profile's public proofs. Accepts a dashboard session or an API key with `profiles:read`.

| Query | Required | Description |
|-------|----------|-------------|
| `profile_id` | No | Which profile `is_public` is computed for; defaults to your primary profile. A key bound to a sub-account always means its own |

```bash
curl https://api.proof.holdings/api/v1/profiles/me/assets?profile_id=64a1b2c3d4e5f6a7b8c9d0e1 \
  -H "Authorization: Bearer pk_live_..."
```

```json
{
  "assets": [
    {
      "id": "507f1f77bcf86cd799439011",
      "type": "phone",
      "identifier": "+37069199199",
      "channel": "whatsapp",
      "verified_at": "2026-01-15T10:00:00Z",
      "is_public": true
    },
    {
      "id": "507f1f77bcf86cd799439012",
      "type": "domain",
      "identifier": "team.management",
      "channel": "dns",
      "verified_at": "2026-01-20T09:30:00Z",
      "is_public": false
    }
  ]
}
```

---

## Update Public Proofs

`PUT /api/v1/profiles/profiles/:profileId/proofs`

Control which verified assets appear on a profile and how they are displayed.

### Request Body

| Parameter | Type | Description |
|-----------|------|-------------|
| `proofs` | array | Up to 50 proof display settings |

Each proof in the array:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `asset_id` | string | Yes | ID of the verified asset |
| `is_visible` | boolean | No | Whether to show on profile |
| `mask_level` | string | No | `full`, `partial` (default), or `hidden` |
| `display_order` | number | No | Sort order (0-based) |
| `label` | string | No | Custom label, max 50 chars (e.g., "Support Phone") |

```bash
curl -X PUT https://api.proof.holdings/api/v1/profiles/profiles/64a1b2c3d4e5f6a7b8c9d0e1/proofs \
  -H "Cookie: token=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "proofs": [
      {
        "asset_id": "507f1f77bcf86cd799439011",
        "is_visible": true,
        "mask_level": "partial",
        "display_order": 0,
        "label": "Main Phone"
      },
      {
        "asset_id": "507f1f77bcf86cd799439012",
        "is_visible": true,
        "mask_level": "full",
        "display_order": 1,
        "label": "Business Domain"
      }
    ]
  }'
```

### Privacy Masking

| Level | Phone | Email | Domain |
|-------|-------|-------|--------|
| `full` | +37069199199 | hello@team.management | team.management |
| `partial` | +3••••••••99 | h•••@team.management | team.management |
| `hidden` | Phone verified | h•••@•••.management | Domain verified |

A partial phone keeps its first two and last two characters. A hidden email still shows its first
letter and top-level domain; a domain is never masked at `partial`.

---

## Claim a Username

`POST /api/v1/profiles/me/username`

Claims the username shown at `/u/{username}`. Dashboard session only (an API key is refused), and
rate-limited to 5 attempts per minute per IP.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | string | Yes | 3–30 characters: letters, numbers and underscores. Stored lowercase |

```bash
curl -X POST https://api.proof.holdings/api/v1/profiles/me/username \
  -H "Cookie: token=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{ "username": "teammanagement" }'
```

```json
{
  "username": "teammanagement",
  "username_claimed_at": "2026-02-06T12:00:00Z",
  "profile_url": "/u/teammanagement"
}
```

| Code | HTTP | Description |
|------|------|-------------|
| `already_claimed` | 409 | Your account already has a username |
| `username_taken` | 409 | Someone else has it |
| `username_reserved` | 409 | Reserved for the system |
| `username_retired` | 409 | It belonged to a deleted sub-account or account; it is never released |

---

## Theme Object

| Field | Type | Values |
|-------|------|--------|
| `primary_color` | string | Hex color (e.g., `#3B82F6`) |
| `background_type` | string | `solid` or `gradient` |
| `background_color` | string | Hex color (solid backgrounds) |
| `gradient_start` | string | Hex color (gradient start) |
| `gradient_end` | string | Hex color (gradient end) |
| `font_style` | string | `default`, `modern`, or `classic` |

---

## Public Profile URLs

Profiles are publicly accessible without authentication:

| URL Pattern | Example |
|-------------|---------|
| `/p/:profileId` | `https://proof.holdings/p/k7x2m9p4` |
| `/u/:username` | `https://proof.holdings/u/teammanagement` |

---

## Verification Levels

Every profile reports `verification_level: "basic"` today. The values `verified` and `business`
exist in the schema but nothing awards them yet, so a profile does not earn a higher level by
adding assets.

---

## Notes

- Each user can have multiple profiles, but only one can be **primary**
- Every outbound message carries exactly one sub-account's branding — name, logo, colour, email theme, support email and [message templates](/docs/templates). A key bound to a sub-account always uses that sub-account; an account-wide key or a dashboard session uses the sub-account it names, else the primary profile. There is no account-wide branding
- A key bound to a sub-account reads every profile but changes only its own; see [Sub-accounts and keys](/docs/api#sub-accounts-and-keys)
- Deleting a profile does not revoke the underlying verified assets
- Profile IDs are cryptographically random 8-character strings
