---
title: Projects & Templates
lastUpdated: "2026-02-06"
status: current
---

# Projects & Templates

> Custom branding and message templates per project

---

## Overview

Projects let you manage multiple brands or applications under a single account. Each project has its own branding (name, logo, colors), callback URLs, and custom message templates that override the default messages sent via WhatsApp, Telegram, SMS, and email.

A default project is automatically created for every user and cannot be deleted.

---

## Project Endpoints

All endpoints require JWT authentication.

Base path: `https://api.proof.holdings/api/v1/me`

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/me/projects` | List all projects |
| `POST` | `/me/projects` | Create new project |
| `GET` | `/me/projects/:id` | Get project details |
| `PUT` | `/me/projects/:id` | Update project |
| `DELETE` | `/me/projects/:id` | Delete project |

---

## Create Project

`POST /api/v1/me/projects`

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Project name (max 100 chars) |
| `description` | string | No | Description (max 500 chars) |
| `branding` | object | No | Branding settings (see below) |

### Branding Object

| Field | Type | Description |
|-------|------|-------------|
| `business_name` | string | Name shown to users in messages |
| `logo_url` | string | Logo URL for email templates |
| `primary_color` | string | Hex color (e.g., `#3B82F6`) |
| `support_email` | string | Support contact shown in messages |

```bash
curl -X POST https://api.proof.holdings/api/v1/me/projects \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Marketplace",
    "description": "Verification flows for the Acme marketplace app",
    "branding": {
      "business_name": "Acme Marketplace",
      "logo_url": "https://acme.com/logo.png",
      "primary_color": "#10B981",
      "support_email": "support@acme.com"
    }
  }'
```

### Response

```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0e1",
  "name": "Acme Marketplace",
  "description": "Verification flows for the Acme marketplace app",
  "is_default": false,
  "branding": {
    "business_name": "Acme Marketplace",
    "logo_url": "https://acme.com/logo.png",
    "primary_color": "#10B981",
    "support_email": "support@acme.com"
  },
  "callbacks": {},
  "templates": [],
  "status": "active",
  "created_at": "2026-02-06T12:00:00Z",
  "updated_at": "2026-02-06T12:00:00Z"
}
```

---

## Update Project

`PUT /api/v1/me/projects/:id`

Accepts any subset of the create fields, plus callback URLs:

| Parameter | Type | Description |
|-----------|------|-------------|
| `callbacks` | object | Redirect URLs for verification flows |

### Callbacks Object

| Field | Type | Description |
|-------|------|-------------|
| `success_url` | string | Redirect after successful verification |
| `failure_url` | string | Redirect after failed verification |
| `cancel_url` | string | Redirect when user cancels |

```bash
curl -X PUT https://api.proof.holdings/api/v1/me/projects/64a1b2c3d4e5f6a7b8c9d0e1 \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "callbacks": {
      "success_url": "https://acme.com/verified",
      "failure_url": "https://acme.com/failed",
      "cancel_url": "https://acme.com/cancelled"
    }
  }'
```

---

## Template Endpoints

Templates let you customize the messages sent to users during verification flows.

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/me/projects/:id/templates` | List project templates |
| `PUT` | `/me/projects/:id/templates/:channel/:type` | Create or update template |
| `DELETE` | `/me/projects/:id/templates/:channel/:type` | Delete template (revert to default) |
| `POST` | `/me/projects/:id/templates/preview` | Preview rendered template |

---

## Create/Update Template

`PUT /api/v1/me/projects/:id/templates/:channel/:type`

### URL Parameters

| Parameter | Values |
|-----------|--------|
| `channel` | `telegram`, `whatsapp`, `sms`, `email` |
| `type` | `verification_request`, `verification_success`, `verification_expired`, `login_request`, `login_success`, `2fa_request`, `2fa_success` |

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `body` | string | Yes | Template text with `{{variables}}` (max 2000 chars) |
| `subject` | string | No | Email subject line (max 200 chars, email only) |
| `button_text` | string | No | CTA button text (max 50 chars) |
| `button_url_template` | string | No | Button URL template (max 500 chars) |
| `is_active` | boolean | No | Enable/disable this template |

### Template Variables

Use `{{variable_name}}` syntax in the body text:

| Variable | Description |
|----------|-------------|
| `{{business_name}}` | Project's business name |
| `{{code}}` | Verification code |
| `{{expires_in}}` | Time until expiry |
| `{{user_identifier}}` | The asset being verified |
| `{{channel}}` | Verification channel name |
| `{{callback_url}}` | Callback/redirect URL |
| `{{support_email}}` | Support email address |

### Example: Custom WhatsApp Verification Message

```bash
curl -X PUT https://api.proof.holdings/api/v1/me/projects/64a1b2c3d4e5f6a7b8c9d0e1/templates/whatsapp/verification_request \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "body": "Welcome to {{business_name}}! Your verification code is {{code}}. This code expires in {{expires_in}}.",
    "is_active": true
  }'
```

### Example: Custom Email Template

```bash
curl -X PUT https://api.proof.holdings/api/v1/me/projects/64a1b2c3d4e5f6a7b8c9d0e1/templates/email/verification_request \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "Verify your email for {{business_name}}",
    "body": "Hi there,\n\nPlease verify your email address ({{user_identifier}}) by entering this code:\n\n{{code}}\n\nThis code expires in {{expires_in}}.\n\nIf you did not request this, please ignore this email.\n\n— The {{business_name}} Team",
    "button_text": "Verify Email",
    "is_active": true
  }'
```

---

## Preview Template

`POST /api/v1/me/projects/:id/templates/preview`

Render a template with sample data before saving.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channel` | string | Yes | `telegram`, `whatsapp`, `sms`, `email` |
| `message_type` | string | Yes | Message type (see values above) |
| `body` | string | Yes | Template body to preview |
| `subject` | string | No | Email subject to preview |
| `button_text` | string | No | Button text to preview |
| `button_url_template` | string | No | Button URL to preview |

```bash
curl -X POST https://api.proof.holdings/api/v1/me/projects/64a1b2c3d4e5f6a7b8c9d0e1/templates/preview \
  -H "Cookie: session=eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "email",
    "message_type": "verification_request",
    "subject": "Verify your email for {{business_name}}",
    "body": "Your code is {{code}}. Expires in {{expires_in}}."
  }'
```

---

## Delete Template

`DELETE /api/v1/me/projects/:id/templates/:channel/:type`

Removes a custom template. Future messages for that channel/type will use the system default.

```bash
curl -X DELETE https://api.proof.holdings/api/v1/me/projects/64a1b2c3d4e5f6a7b8c9d0e1/templates/whatsapp/verification_request \
  -H "Cookie: session=eyJ..."
```

---

## Message Types

| Type | When Sent |
|------|-----------|
| `verification_request` | User needs to verify an asset |
| `verification_success` | Asset successfully verified |
| `verification_expired` | Verification challenge expired |
| `login_request` | Login verification initiated |
| `login_success` | Login completed |
| `2fa_request` | 2FA challenge sent |
| `2fa_success` | 2FA completed |

---

## Channel Support

| Channel | Subject | Body | Button | Rate Limit |
|---------|---------|------|--------|------------|
| `email` | Yes | Yes | Yes | 30/min |
| `whatsapp` | No | Yes | No | 30/min |
| `telegram` | No | Yes | No | 30/min |
| `sms` | No | Yes | No | 30/min |

---

## Notes

- Each user gets a **default project** automatically (cannot be deleted)
- Project names must be unique within an account
- When no custom template exists for a channel/type, the system default is used
- Template updates are rate-limited to 30 per minute
- Projects can be archived by setting `status: "archived"` (not yet exposed via API)
