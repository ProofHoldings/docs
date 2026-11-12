---
title: Message Templates
lastUpdated: "2026-10-29"
status: current
---

# Message Templates

> Custom message templates, one set per sub-account

---

## Overview

Templates customize the messages we send to people during verification, login and 2FA, over
email, Telegram and WhatsApp. A template you have not set falls back to our system default, so you
only override what you want to change. Read the defaults first with
`GET /api/v1/templates/defaults`. A template is only used for a message we actually send, and only
the variables that message fills are replaced — [What is sent today](#what-is-sent-today) lists
both, and `GET /api/v1/templates` returns the same list as `sendable`. A save outside that list
still succeeds, with a warning.

Every message belongs to exactly one sub-account (the public profile it is sent for),
and only that sub-account's branding appears in it. Templates follow the same rule. A message is
rendered from:

1. **The sub-account's template** — the one stored for that sub-account, channel and message type.
2. **System default** — our own copy, when the sub-account has no template of its own.

There is no account-wide template: a template saved for one sub-account is never used by another.
Sub-accounts are managed on their own endpoints and documented in
[Multi-Profile System](/docs/profiles), which also covers each sub-account's name, logo, colour, email
theme and support email — this page covers templates only.

---

## Template Endpoints

Base URL: `https://api.proof.holdings/api/v1/templates`

These endpoints accept **either** an API key (`Authorization: Bearer pk_live_…`) **or** a dashboard
session cookie. API-key callers need the `templates:read` scope to read and `templates:write` to
write. Dashboard sessions additionally require a 2FA step-up before `PUT` and `DELETE`.

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/api/v1/templates` | List a sub-account's templates, and what is sent (`sendable`) |
| `GET` | `/api/v1/templates/defaults` | List our system default of every message we send |
| `GET` | `/api/v1/templates/:channel/:messageType` | Get one resolved template |
| `PUT` | `/api/v1/templates/:channel/:messageType` | Create or update a template |
| `DELETE` | `/api/v1/templates/:channel/:messageType` | Delete a template (revert to the default) |
| `POST` | `/api/v1/templates/preview` | Render a draft against sample data |
| `POST` | `/api/v1/templates/render` | Render a stored template with supplied variables |

---

## Choosing the Sub-account

Every template operation works on one sub-account, named by an optional `?profile_id` query parameter:
`GET /api/v1/templates`, `GET`/`PUT`/`DELETE /api/v1/templates/:channel/:messageType`,
`POST /api/v1/templates/preview` and `POST /api/v1/templates/render`. Preview and render use the
sub-account's own name and support email for `{{business_name}}` and `{{support_email}}`.

`GET /api/v1/templates/defaults` returns the platform defaults, which are the same for every
sub-account.

| Caller | Behaviour |
|--------|-----------|
| **API key bound to a sub-account** | Always that sub-account. Naming a different sub-account returns `403 profile_mismatch` — a key cannot act outside its own binding. |
| **Account-wide API key or dashboard session** | May pass `?profile_id=<id>` to name a sub-account you own; one you do not own returns `403 profile_not_owned`. Omit it to act on your **primary** sub-account. |

A bound key reaches only its own sub-account everywhere else too — see [Sub-accounts and keys](/docs/api#sub-accounts-and-keys).

`GET /api/v1/templates/:channel/:messageType` reports which layer answered in a `scope` field
(`project` or `default`), and names the sub-account in `profile_id`.

---

## Create or Update a Template

`PUT /api/v1/templates/:channel/:messageType`

### URL Parameters

| Parameter | Values |
|-----------|--------|
| `channel` | `telegram`, `whatsapp`, `sms`, `email` |
| `messageType` | A type sent on that channel — see [What is sent today](#what-is-sent-today) |

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `body` | string | Yes | Template text with `{{variables}}` (1–2000 chars) |
| `subject` | string | Email: yes | Email subject line (max 200 chars). Required for `email`, ignored elsewhere |
| `button_text` | string | No | CTA button text (max 50 chars) |
| `button_url_template` | string | No | Button URL template (max 500 chars) |

`is_active` is returned on template responses but is not accepted here — delete the template to stop
using it.

A template is refused with `400 validation_error` when:

- the channel is `email` and there is no `subject`;
- the channel is `sms` and the body may exceed 160 characters (estimated with `{{business_name}}`
  as 30 characters, `{{code}}` as 6 and `{{expires_in}}` as 15);
- the channel is `sms`, the type is `verification_request`, `login_request` or `2fa_request`, and the
  body has no `{{code}}`;
- the body uses a variable not in [Template Variables](#template-variables).

A template that would never be sent as written is still saved, but the `200` response carries a
`warnings` list saying why:

- `template_not_sent` — no message is sent for that channel and type, so the template is never used;
- `template_variable_not_filled` — the `subject`, `body`, `button_text` or `button_url_template`
  uses a variable that message never fills (`{{code}}` on Telegram, say, or `{{user_identifier}}`
  anywhere), so every message for that pair goes out with the platform's default copy instead of
  your template. The names are listed in `variables`.

```json
{
  "success": true,
  "warnings": [
    {
      "code": "template_variable_not_filled",
      "message": "{{code}} is never filled for telegram/verification_request — use {{business_name}}, {{support_email}}, …",
      "variables": ["code"]
    }
  ],
  "template": { "channel": "telegram", "message_type": "verification_request", "...": "..." }
}
```

`warnings` is empty when the template will be sent as written.

### No hard-coded sub-account names

A template may name its brand only through `{{business_name}}`. A `subject`, `body` or `button_text`
that spells out the name of one of your sub-accounts (its business name or display name, three
characters or more, matched case-insensitively as a whole word) is refused with
`400 template_hardcoded_name`. The same word inside a longer word is not a match.

### Example: a custom WhatsApp confirmation

WhatsApp sends only the reply after a verification, login or 2FA completes, and is not given the
code or its expiry, so use only the variables it fills:

```bash
curl -X PUT https://api.proof.holdings/api/v1/templates/whatsapp/verification_success \
  -H "Authorization: Bearer pk_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "Your number is confirmed with {{business_name}}. Questions? Write to {{support_email}}."
  }'
```

### Example: a custom email template for one sub-account

```bash
curl -X PUT "https://api.proof.holdings/api/v1/templates/email/verification_request?profile_id=64a1b2c3d4e5f6a7b8c9d0e1" \
  -H "Authorization: Bearer pk_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "Verify your email for {{business_name}}",
    "body": "Hi there,\n\nPlease verify your email address by entering this code:\n\n{{code}}\n\nThis code expires in {{expires_in}}.\n\nIf you did not request this, please ignore this email.\n\n— The {{business_name}} Team",
    "button_text": "Verify Email"
  }'
```

---

## Preview a Template

`POST /api/v1/templates/preview`

Render a draft against sample data before saving it. Nothing is stored. Pass `?profile_id` to
preview it with that sub-account's name and support email. `validation_errors` and `is_valid` say
whether a save would be refused; `warnings` carries the same warnings a save would return.

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

Removes the sub-account's custom template. Later messages from that sub-account on that channel and message
type use our system default.

```bash
curl -X DELETE https://api.proof.holdings/api/v1/templates/whatsapp/verification_success \
  -H "Authorization: Bearer pk_live_your_api_key_here"
```

---

## Message Types

Every type below is a valid `messageType` in a path, but a template is used only for the ones
sent on that channel ([What is sent today](#what-is-sent-today)).

| Type | When Sent |
|------|-----------|
| `verification_request` | Someone needs to verify an asset |
| `verification_success` | Asset successfully verified |
| `verification_expired` | Not sent today |
| `verification_refused` | Your app refused a verified number, on a request created with `outcome: "relying_party"` ([Your Decision After the Proof](/docs/api#your-decision-after-the-proof)) |
| `login_request` | A login verification is requested |
| `login_success` | Login completed |
| `2fa_request` | A 2FA challenge is requested |
| `2fa_success` | 2FA completed |
| `account_deletion_request` | Not sent today |
| `account_deletion_success` | Not sent today |
| `confirmation_request` | Not sent today |
| `confirmation_approved` | Not sent today |
| `confirmation_denied` | Not sent today |

### What is sent today

| Type | Email | Telegram | WhatsApp |
|------|-------|----------|----------|
| `verification_request`, `login_request`, `2fa_request` | Yes | Yes | No |
| `verification_success`, `login_success`, `2fa_success` | No | Yes | Yes |
| `verification_refused` | No | Yes | Yes |

Only the pairs marked Yes are ever sent. Nothing is sent over **SMS** — the person sends the code to
us — so an SMS template is never used, nor is any `verification_expired`, `account_deletion_*` or
`confirmation_*` template (approval requests use their own fixed notice). Such a template is still
saved, with a `template_not_sent` warning, and is listed, read and deletable like any other.

---

## Template Variables

Use `{{variable_name}}` in the body (and in an email `subject`), written exactly that way: a
template whose subject, body, button text or button URL holds a misspelt token such as
`{{ code }}` or `{{code-name}}`, or an unknown variable, is refused when you save it.

A message never goes out with a raw `{{variable}}` in it. If your template uses a variable that
is not filled for that message, the message is sent with the platform's default copy instead of
your template — so use only the variables the channel fills. A variable the message never fills
draws a `template_variable_not_filled` warning on save; `sendable` on `GET /api/v1/templates`
lists the ones each channel and type fills.

| Variable | Description | Filled |
|----------|-------------|--------|
| `{{business_name}}` | The sub-account's name | Every channel |
| `{{support_email}}` | The sub-account's support email | Every channel |
| `{{code}}` | Verification code | Email only |
| `{{expires_in}}` | Time until expiry (e.g. "10 minutes") | Email only |
| `{{action_label}}` | The action requested (e.g. "Two-Factor Authentication") | When the verification carries an action; otherwise empty |
| `{{action_title}}` | Short title of what was requested | When the verification carries an action; otherwise empty |
| `{{action_description}}` | What the recipient is being asked to do | When the action has a description; otherwise empty |
| `{{currency}}` | Currency, for approval-type actions | When the action has one; otherwise empty |
| `{{entity}}` | Entity or counterparty, for approval-type actions | When the action has one; otherwise empty |
| `{{reason}}` | Your app's reason for refusing a verified number | In `verification_refused` only; empty when you gave none |
| `{{amount}}` | Amount, for approval-type actions | Only when the action has a numeric amount; otherwise the default copy is sent |
| `{{user_identifier}}` | Masked phone or email | Never filled today — the default copy is sent |
| `{{channel}}` | Current channel name | Never filled today — the default copy is sent |
| `{{callback_url}}` | Success callback URL | Never filled today — the default copy is sent |

`POST /api/v1/templates/preview` renders every variable with sample data. A variable the message
never fills is listed in `warnings`; one it fills only sometimes (an action's `{{amount}}`,
say) still renders in the preview.

---

## See Also

| Topic | Documentation |
|-------|---------------|
| Managing sub-accounts and their branding | [Multi-Profile System](/docs/profiles) |
| Full endpoint reference | [API Reference](/docs/api) |
