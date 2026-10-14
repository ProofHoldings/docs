---
title: Message Templates
lastUpdated: "2026-08-26"
status: current
---

# Message Templates

> Custom message templates for your account, with optional per-profile overrides

---

## Overview

Templates customize the messages sent to people during verification, login, 2FA, confirmation and
account-deletion flows over WhatsApp, Telegram, SMS and email. A template you have not set falls
back to the Proof system default, so you only override what you want to change.

Templates resolve in three steps, most specific first:

1. **Profile override** — a template stored against one public profile.
2. **Account template** — your account-wide template.
3. **System default** — Proof's own copy.

A public profile is the "brand" a template can be scoped to. Profiles are managed on their own
endpoints and documented in [Multi-Profile System](/docs/profiles) — this page covers templates
only.

---

## Template Endpoints

Base URL: `https://api.proof.holdings/api/v1/templates`

These endpoints accept **either** an API key (`Authorization: Bearer pk_live_…`) **or** a dashboard
session cookie. API-key callers need the `templates:read` scope to read and `templates:write` to
write. Dashboard sessions additionally require a 2FA step-up before `PUT` and `DELETE`.

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/api/v1/templates` | List your templates and the available variables |
| `GET` | `/api/v1/templates/defaults` | List Proof's system default templates |
| `GET` | `/api/v1/templates/:channel/:messageType` | Get one resolved template |
| `PUT` | `/api/v1/templates/:channel/:messageType` | Create or update a template |
| `DELETE` | `/api/v1/templates/:channel/:messageType` | Delete a template (revert to the default) |
| `POST` | `/api/v1/templates/preview` | Render a draft against sample data |
| `POST` | `/api/v1/templates/render` | Render a stored template with supplied variables |

---

## Scoping a Template to One Profile

The scoped endpoints accept an optional `?profile_id` query parameter that selects which scope you
are reading or writing: `GET /api/v1/templates`, `GET`/`PUT`/`DELETE /api/v1/templates/:channel/:messageType`,
and `POST /api/v1/templates/render`.

Two endpoints deliberately ignore it, because neither reads or writes a stored template:
`GET /api/v1/templates/defaults` returns the platform defaults, which are the same for every scope,
and `POST /api/v1/templates/preview` renders the draft you send it. Passing `?profile_id` to either
is accepted and has no effect rather than being rejected — do not read a successful response from
them as confirmation that a scope was applied.

| Caller | Behaviour |
|--------|-----------|
| **API key** | The scope is the profile bound to that key, if it has one. A `profile_id` in the query or body is **ignored** — a key cannot act outside its own binding. |
| **Dashboard session** | May pass `?profile_id=<id>`. Ownership is checked; a profile you do not own returns `403 profile_not_owned`. Omit it to act on the account scope. |

`GET /api/v1/templates/:channel/:messageType` reports which of the three layers answered, in a
`scope` field on the response.

---

## Create or Update a Template

`PUT /api/v1/templates/:channel/:messageType`

### URL Parameters

| Parameter | Values |
|-----------|--------|
| `channel` | `telegram`, `whatsapp`, `sms`, `email` |
| `messageType` | Any value from [Message Types](#message-types) below |

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `body` | string | Yes | Template text with `{{variables}}` (1–2000 chars) |
| `subject` | string | No | Email subject line (max 200 chars, email only) |
| `button_text` | string | No | CTA button text (max 50 chars) |
| `button_url_template` | string | No | Button URL template (max 500 chars) |

`is_active` is returned on template responses but is not accepted here — delete the template to stop
using it.

### Example: a custom WhatsApp verification message

```bash
curl -X PUT https://api.proof.holdings/api/v1/templates/whatsapp/verification_request \
  -H "Authorization: Bearer pk_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "Welcome to {{business_name}}! Your verification code is {{code}}. This code expires in {{expires_in}}."
  }'
```

### Example: a custom email template, scoped to one profile

```bash
curl -X PUT "https://api.proof.holdings/api/v1/templates/email/verification_request?profile_id=64a1b2c3d4e5f6a7b8c9d0e1" \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "Verify your email for {{business_name}}",
    "body": "Hi there,\n\nPlease verify your email address ({{user_identifier}}) by entering this code:\n\n{{code}}\n\nThis code expires in {{expires_in}}.\n\nIf you did not request this, please ignore this email.\n\n— The {{business_name}} Team",
    "button_text": "Verify Email"
  }'
```

---

## Preview a Template

`POST /api/v1/templates/preview`

Render a draft against sample data before saving it. Nothing is stored.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channel` | string | Yes | `telegram`, `whatsapp`, `sms`, `email` |
| `message_type` | string | Yes | See [Message Types](#message-types) |
| `body` | string | Yes | Template body to preview |
| `subject` | string | No | Email subject to preview |
| `button_text` | string | No | Button text to preview |
| `button_url_template` | string | No | Button URL to preview |

```bash
curl -X POST https://api.proof.holdings/api/v1/templates/preview \
  -H "Authorization: Bearer pk_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "email",
    "message_type": "verification_request",
    "subject": "Verify your email for {{business_name}}",
    "body": "Your code is {{code}}. Expires in {{expires_in}}."
  }'
```

---

## Delete a Template

`DELETE /api/v1/templates/:channel/:messageType`

Removes your custom template. Later messages for that channel and message type fall back to the next
layer — the account template if you deleted a profile override, otherwise the Proof system default.

```bash
curl -X DELETE https://api.proof.holdings/api/v1/templates/whatsapp/verification_request \
  -H "Authorization: Bearer pk_live_your_api_key_here"
```

---

## Message Types

| Type | When Sent |
|------|-----------|
| `verification_request` | Someone needs to verify an asset |
| `verification_success` | Asset successfully verified |
| `verification_expired` | Verification challenge expired |
| `login_request` | Login verification initiated |
| `login_success` | Login completed |
| `2fa_request` | 2FA challenge sent |
| `2fa_success` | 2FA completed |
| `account_deletion_request` | Account deletion requested |
| `account_deletion_success` | Account deletion completed |
| `confirmation_request` | An approval was requested |
| `confirmation_approved` | An approval was granted |
| `confirmation_denied` | An approval was refused |

---

## Template Variables

Use `{{variable_name}}` in the body (and in an email `subject`).

| Variable | Description |
|----------|-------------|
| `{{business_name}}` | Your business display name |
| `{{code}}` | Verification code (when applicable) |
| `{{expires_in}}` | Time until expiry (e.g. "10 minutes") |
| `{{user_identifier}}` | Masked phone or email |
| `{{channel}}` | Current channel name |
| `{{callback_url}}` | Success callback URL |
| `{{support_email}}` | Your support email |
| `{{action_label}}` | The action requested (e.g. "Two-Factor Authentication") |
| `{{action_title}}` | Short title of what was requested |
| `{{action_description}}` | What the recipient is being asked to do |
| `{{amount}}` | Amount, for approval-type actions |
| `{{currency}}` | Currency, for approval-type actions |
| `{{entity}}` | Entity or counterparty, for approval-type actions |

The six action variables are populated for confirmation and 2FA flows; they render empty elsewhere.

---

## See Also

| Topic | Documentation |
|-------|---------------|
| Managing public profiles | [Multi-Profile System](/docs/profiles) |
| Full endpoint reference | [API Reference](/docs/api) |
