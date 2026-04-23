---
title: Integrations
lastUpdated: "2026-03-16"
status: current
---

# Platform Integrations

> Connect proof.holdings to your automation stack — n8n, Zapier, Make, and more

---

## Overview

proof.holdings integrates with popular automation platforms so you can add verification and approval workflows without writing code. Each integration exposes the same core capabilities: create verifications, check status, validate proofs, and manage HITL approvals.

---

## Available Integrations

> **⚠️ In active development** — The n8n, Zapier, and Make.com integrations below are under active development and have not yet shipped to their respective marketplaces. The capability surfaces described here reflect the target coverage; some actions may be incomplete or subject to change. If you want early access or want to influence the shape of a specific integration, please reach out.

### n8n

Community node for n8n workflow automation.

**Package:** `n8n-nodes-proof-holdings`

**Install:**
```bash
# In your n8n instance
npm install n8n-nodes-proof-holdings
```

**Capabilities:**
- Create phone, email, domain, and Telegram bot verifications
- Check verification status
- List verifications
- Create and check verification requests
- Validate, list, and revoke proof tokens
- Test-verify for development workflows
- Trigger verification checks

**Authentication:** API key via n8n credentials

---

### Zapier

Zapier connector for trigger-and-action workflows.

**Capabilities:**
- Create phone, email, domain, and Telegram bot verifications
- Check verification status
- List verifications
- Create and check verification requests
- Validate, list, and revoke proof tokens
- Test-verify for development workflows
- Trigger verification checks

**Authentication:** API key

---

### Make.com

Make (formerly Integromat) connector for visual automation.

**Capabilities:**
- Create phone, email, domain, and Telegram bot verifications
- Check verification status
- List verifications
- Create and check verification requests
- Validate, list, and revoke proof tokens
- Test-verify for development workflows
- Trigger verification checks

**Authentication:** API key

---

## Shared Capabilities

All platform integrations support the same core set of 17 capabilities:

| Capability | Description |
|-----------|-------------|
| Create phone verification | Verify phone control via WhatsApp, Telegram, or SMS |
| Create email verification | Verify email control via magic link or OTP |
| Create domain verification | Verify domain control via DNS, HTTP, or email |
| Create Telegram bot verification | Verify Telegram bot ownership |
| Check verification status | Poll a verification for completion |
| List verifications | List all verifications with filters |
| Create verification request | Create a multi-asset B2B request |
| Check verification request | Check request status |
| Validate proof token | Verify a proof token is valid |
| List proofs | List proof tokens |
| Revoke proof | Invalidate a proof token |
| Get public profile | Retrieve a verified public profile |
| Update public profile | Update profile display settings |
| Trigger verification check | Trigger a domain/DNS re-check |
| Test-verify | Auto-complete a test mode verification |

---

## REST API

All integrations are built on the same REST API available to SDKs and the MCP server. For direct API access:

- [API Reference](/docs/api) — Complete endpoint documentation
- [OpenAPI Spec](https://api.proof.holdings/api/openapi.json) — Machine-readable OpenAPI 3.0 specification
- [SDKs](/docs/sdks) — Official client libraries (JavaScript, Python, Go, PHP)
- [MCP Server](/docs/mcp) — 131 tools for AI agents

---

## Building Custom Integrations

The REST API and OpenAPI spec make it straightforward to build integrations for any platform:

1. **Get an API key** — Sign up and generate a `pk_live_*` key from the dashboard
2. **Use the OpenAPI spec** — Import `https://api.proof.holdings/api/openapi.json` into your platform's API connector
3. **Authenticate** — Pass the API key as `Authorization: Bearer pk_live_...`
4. **Test with test mode** — Use a `pk_test_*` key during development — no real messages sent, no quota consumed

---

## Resources

- [API Reference](/docs/api) — Complete API documentation
- [MCP Server](/docs/mcp) — AI agent integration via Model Context Protocol
- [SDKs](/docs/sdks) — JavaScript, Python, Go, PHP client libraries
- [Webhooks](/docs/api#webhooks) — Real-time event notifications
