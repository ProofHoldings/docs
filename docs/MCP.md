---
title: MCP Server
lastUpdated: "2026-03-16"
status: current
---

# MCP Server

> 131 tools for AI agents — verify assets, manage approvals, and automate trust workflows

---

## What is MCP?

The [Model Context Protocol](https://modelcontextprotocol.io) (MCP) lets AI assistants call tools directly. The proof.holdings MCP server wraps our entire REST API into 131 structured tools that any MCP-compatible agent can use — Claude, LangChain, CrewAI, OpenAI Agents SDK, Cursor, Windsurf, and more.

---

## Installation

```bash
npm install -g @proof-holdings/mcp-server
```

Or run directly with npx:

```bash
npx -y @proof-holdings/mcp-server
```

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PROOF_API_KEY` | Yes | — | Your API key (`pk_live_*` or `pk_test_*`) |
| `PROOF_BASE_URL` | No | `https://api.proof.holdings` | API base URL |

---

## Client Setup

### Claude Desktop

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "proof-holdings": {
      "command": "npx",
      "args": ["-y", "@proof-holdings/mcp-server"],
      "env": {
        "PROOF_API_KEY": "pk_live_YOUR_API_KEY"
      }
    }
  }
}
```

### Claude Code CLI

```bash
claude mcp add proof-holdings -- npx -y @proof-holdings/mcp-server \
  --env PROOF_API_KEY=pk_live_YOUR_API_KEY
```

### Cursor

Add to `.cursor/mcp.json` in your project:

```json
{
  "mcpServers": {
    "proof-holdings": {
      "command": "npx",
      "args": ["-y", "@proof-holdings/mcp-server"],
      "env": {
        "PROOF_API_KEY": "pk_live_YOUR_API_KEY"
      }
    }
  }
}
```

### Windsurf

Add to your Windsurf MCP configuration:

```json
{
  "mcpServers": {
    "proof-holdings": {
      "command": "npx",
      "args": ["-y", "@proof-holdings/mcp-server"],
      "env": {
        "PROOF_API_KEY": "pk_live_YOUR_API_KEY"
      }
    }
  }
}
```

---

## Tool Overview (131 tools)

### Verifications (12 tools)

Create and manage asset control verification challenges.

| Tool | Description |
|------|-------------|
| `create_verification` | Create a new verification challenge |
| `get_verification` | Get verification status |
| `list_verifications` | List all verifications |
| `trigger_verification` | Trigger a verification check |
| `submit_verification_code` | Submit a verification code |
| `resend_verification` | Resend verification challenge |
| `test_verify` | Auto-complete a test verification |
| `list_verified_users` | List verified users |
| `get_verified_user` | Get a specific verified user |
| `start_domain_verification` | Start domain verification |
| `check_domain_verification` | Check domain verification status |
| `wait_for_verification` | Poll until verification completes |

### Verification Requests (6 tools)

Multi-asset B2B verification request flows.

| Tool | Description |
|------|-------------|
| `create_verification_request` | Create a multi-asset request |
| `get_verification_request` | Get request status |
| `list_verification_requests` | List all requests |
| `get_request_by_reference` | Get request by reference ID |
| `cancel_verification_request` | Cancel a request |
| `wait_for_request` | Poll until request completes |

### Proofs (4 tools)

Validate, check, and revoke cryptographic proof tokens.

| Tool | Description |
|------|-------------|
| `validate_proof` | Validate a proof token |
| `get_proof_status` | Check proof status |
| `revoke_proof` | Revoke a proof |
| `list_revoked_proofs` | List revoked proofs |

### HITL Approvals (8 tools)

Human-in-the-loop approval workflows — create configs and request human approval.

| Tool | Description |
|------|-------------|
| `create_hitl` | Create a HITL approval config |
| `get_hitl` | Get a HITL config |
| `list_hitls` | List all HITL configs |
| `update_hitl` | Update a HITL config |
| `delete_hitl` | Archive a HITL config |
| `create_confirmation` | Request human approval |
| `get_confirmation` | Get confirmation status |
| `list_confirmations` | List confirmations (filter by status, config, search) |

### Authorizations (5 tools)

Manage delegated consent — users authorize agents to contact them.

| Tool | Description |
|------|-------------|
| `create_authorization` | Request user authorization |
| `get_authorization` | Get authorization status |
| `list_authorizations` | List all authorizations |
| `revoke_authorization` | Revoke an authorization |
| `export_authorizations` | Export authorizations |

### Sessions (3 tools)

Phone-first authentication sessions.

| Tool | Description |
|------|-------------|
| `create_session` | Create an auth session |
| `get_session` | Get session status |
| `wait_for_session` | Poll until session completes |

### Templates (7 tools)

Custom message branding and templates.

| Tool | Description |
|------|-------------|
| `list_templates` | List all templates |
| `get_default_templates` | Get default templates |
| `get_template` | Get a specific template |
| `update_template` | Update a template |
| `delete_template` | Delete a template |
| `preview_template` | Preview rendered template |
| `render_template` | Render a template |

### Webhook Deliveries (4 tools)

Monitor and retry webhook deliveries.

| Tool | Description |
|------|-------------|
| `get_webhook_stats` | Get delivery statistics |
| `list_webhook_deliveries` | List deliveries |
| `get_webhook_delivery` | Get a specific delivery |
| `retry_webhook_delivery` | Retry a failed delivery |

### Account Management (45+ tools)

Assets, settings, profiles, projects, emails, phones, API keys, 2FA, domains, DNS credentials, billing, and more.

See the [API Reference](/docs/api) for the full list of account management endpoints, all of which are available as MCP tools.

---

## Example: Verify a Phone Number

```
Use the create_verification tool to verify phone +37069199199 via WhatsApp.
Then use wait_for_verification to poll until the user completes it.
```

The agent will call:

1. `create_verification` with `{ type: "phone", channel: "whatsapp", identifier: "+37069199199" }`
2. `wait_for_verification` with the returned verification ID
3. The result includes the signed proof token

## Example: Request Human Approval

```
Use create_confirmation to ask the human to approve deploying v2.1 to production.
Use HITL config ID "abc123". Set a 1-hour timeout.
```

The agent will call:

1. `create_confirmation` with `{ hitl_id: "abc123", message: "Deploy v2.1 to production?", timeout: 3600 }`
2. `get_confirmation` to check the result — approved, denied, or expired

---

## Test Mode

Use a `pk_test_*` API key and all tools work in test mode:

- No real messages are sent
- Use `test_verify` to auto-complete verifications
- Webhooks still fire for integration testing
- Test verifications don't count toward your quota

---

## Resources

- [npm package](https://www.npmjs.com/package/@proof-holdings/mcp-server) — Install from npm
- [API Reference](/docs/api) — Complete endpoint documentation
- [SDKs](/docs/sdks) — Official client libraries for JavaScript, Python, Go, PHP
- [OpenAPI Spec](https://api.proof.holdings/api/openapi.json) — Machine-readable specification
