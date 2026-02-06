---
title: Multi-Profile System
lastUpdated: "2026-02-06"
status: current
---

# Multi-Profile System

> Create multiple public profiles with independent settings and verified assets

---

## Overview

Users can create multiple public profiles, each with its own display name, bio, theme, and set of verified assets. This supports use cases like separating personal and business identities, or managing multiple brands under one account.

Each profile gets a unique shareable URL (`/p/{profileId}`) and can optionally claim a vanity username (`/@{username}`).

---

## Endpoints

All profile endpoints require JWT authentication and are scoped to the authenticated user.

Base path: `https://api.proof.holdings/api/v1/me`

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/me/profiles` | List all profiles |
| `POST` | `/me/profiles` | Create new profile |
| `GET` | `/me/profiles/:profileId` | Get specific profile |
| `PATCH` | `/me/profiles/:profileId` | Update profile |
| `DELETE` | `/me/profiles/:profileId` | Delete profile |
| `POST` | `/me/profiles/:profileId/primary` | Set as primary profile |
| `PUT` | `/me/profiles/:profileId/proofs` | Update profile's public proofs |

---

## Create Profile

`POST /api/v1/me/profiles`

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

```bash
curl -X POST https://api.proof.holdings/api/v1/me/profiles \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "display_name": "Acme Corp",
    "bio": "Verified digital presence for Acme Corporation",
    "is_business": true,
    "business_name": "Acme Corp",
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
  "display_name": "Acme Corp",
  "bio": "Verified digital presence for Acme Corporation",
  "is_business": true,
  "business_name": "Acme Corp",
  "is_public": true,
  "is_primary": false,
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
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "bio": "Updated bio for Acme",
    "custom_links": [
      { "title": "Website", "url": "https://acme.com", "display_order": 0 },
      { "title": "Support", "url": "https://acme.com/support", "display_order": 1 }
    ]
  }'
```

---

## Update Public Proofs

`PUT /api/v1/me/profiles/:profileId/proofs`

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
| `mask_level` | string | No | `full`, `partial`, or `hidden` |
| `display_order` | number | No | Sort order (0-based) |
| `label` | string | No | Custom label (e.g., "Support Phone") |

```bash
curl -X PUT https://api.proof.holdings/api/v1/me/profiles/64a1b2c3d4e5f6a7b8c9d0e1/proofs \
  -H "Cookie: session=eyJ..." \
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
| `full` | +37069199199 | user@example.com | example.com |
| `partial` | +3••••9199 | us•••@example.com | example.com |
| `hidden` | Verified Phone | Verified Email | Verified Domain |

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
| `/@:username` | `https://proof.holdings/@acmecorp` |

---

## Verification Levels

Profiles automatically earn verification badges based on verified asset count:

| Level | Requirement |
|-------|-------------|
| `basic` | Default for new profiles |
| `verified` | Multiple verified asset types |
| `business` | Business profile with verified domain |

---

## Notes

- Each user can have multiple profiles, but only one can be **primary**
- The primary profile is used as the default for branding in verification flows
- Deleting a profile does not revoke the underlying verified assets
- Profile IDs are cryptographically random 8-character strings
- Username claiming is rate-limited to 5 attempts per minute
