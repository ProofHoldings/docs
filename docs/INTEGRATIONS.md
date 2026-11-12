---
title: Integrations
lastUpdated: "2026-10-26"
status: current
---

# Platform Integrations

> Call Proof Holdings from n8n, Zapier, Make or any other automation tool

---

## Today

Call the API from any automation tool through its HTTP step, using the [OpenAPI document](https://api.proof.holdings/api/openapi.json). Import it into the tool's API connector, or point a plain HTTP request at the endpoint you need with `Authorization: Bearer pk_live_...`. The steps are under [Building Custom Integrations](#building-custom-integrations).

---

## Planned

> **⚠️ Not yet available** — The n8n, Zapier, and Make.com integrations below are in development and have not shipped to their marketplaces or package registries. What follows describes the planned coverage, which may change. If you want early access or want to influence the shape of a specific integration, please reach out.

Each will cover the core operations — create verifications, check status, validate proofs — and, on n8n and Zapier, approval requests, without writing code.

### n8n

A community node for n8n workflow automation. Its package, `n8n-nodes-proof-holdings`, is not yet published, so there is nothing to install yet.

**Planned capabilities:**
- Create phone, email, domain, and Telegram bot verifications
- Check verification status
- List verifications
- Create and check verification requests
- Validate and revoke proof tokens, and list revoked proofs
- Test-verify for development workflows
- Trigger verification checks

**Authentication:** API key via n8n credentials

---

### Zapier

A Zapier connector for trigger-and-action workflows.

**Planned capabilities:**
- Create phone, email, domain, and Telegram bot verifications
- Check verification status
- List verifications
- Create and check verification requests
- Validate and revoke proof tokens, and list revoked proofs
- Test-verify for development workflows
- Trigger verification checks

**Authentication:** API key

---

### Make.com

A Make (formerly Integromat) connector for visual automation.

**Planned capabilities:**
- Create phone, email, domain, and Telegram bot verifications
- Check verification status
- List verifications
- Create and check verification requests
- Validate and revoke proof tokens, and list revoked proofs
- Test-verify for development workflows
- Trigger verification checks

**Authentication:** API key

---

### Shared Capabilities

The integrations will share the core set below. Beyond it, coverage differs by platform: approval requests and HITL key management are not planned for every one, and Make cannot build the encrypted message an approval request needs. The core set:

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
| List revoked proofs | Read the revocation list |
| Revoke proof | Invalidate a proof token |
| Get public profile | Retrieve a verified public profile |
| Update public profile | Update profile display settings |
| Trigger verification check | Trigger a domain/DNS re-check |
| Test-verify | Auto-complete a test mode verification |

---

## REST API

All integrations are built on the same REST API available to SDKs and the MCP server. For direct API access:

- [API Reference](/docs/api) — Complete endpoint documentation
- [OpenAPI Spec](https://api.proof.holdings/api/openapi.json) — Machine-readable OpenAPI 3.1 specification
- [SDKs](/docs/sdks) — Official client libraries (JavaScript, Python, Go, PHP)
- [MCP Server](/docs/mcp) — 193 tools for AI agents

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
