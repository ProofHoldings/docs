---
title: MCP Server
lastUpdated: "2026-02-27"
status: current
---

# MCP Server

The proof.holdings MCP (Model Context Protocol) server exposes 118 tools for AI agents to interact with the entire verification API. It works with Claude Desktop, Claude Code, Cursor, and any MCP-compatible client.

## Installation

```bash
npm install -g @proof/mcp-server
```

Or run directly without installing:

```bash
npx @proof/mcp-server
```

## Configuration

The server requires a `PROOF_API_KEY` environment variable. Optionally set `PROOF_BASE_URL` to target a staging or local instance (defaults to `https://api.proof.holdings`).

Get your API key from the [dashboard](/dashboard/api-keys).

## Client Setup

Add the following to your MCP client config file:

```json
{
  "mcpServers": {
    "proof-holdings": {
      "command": "npx",
      "args": ["-y", "@proof/mcp-server"],
      "env": {
        "PROOF_API_KEY": "pk_live_your_key_here"
      }
    }
  }
}
```

Config file locations:

| Client | Config file |
|--------|-------------|
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Desktop (Windows) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Cursor | `.cursor/mcp.json` in your project root |
| Claude Code | `.mcp.json` in your project root |

If installed globally, use `"command": "proof-mcp"` and remove the `"args"` field.

## Tools Overview

The server provides 118 tools organized into 22 resource groups. Every tool maps directly to a REST API endpoint.

### Core Verification (25 tools)

**Verifications (12)** -- Create, retrieve, list, trigger, submit codes, resend, test-verify, domain verification, and polling.

**Verification Requests (6)** -- Create multi-asset requests, retrieve by ID or reference, list, cancel, and poll until complete.

**Sessions (3)** -- Create phone verification sessions, check status, and poll until terminal state.

**Proofs (4)** -- Validate proof tokens (checks revocation), get status, revoke, and list revoked proofs.

### Account Management (37 tools)

**Assets (3)** -- List, get, and revoke verified assets.

**Auth (3)** -- Get current user, list sessions, revoke sessions.

**Settings (4)** -- Usage stats, account settings, and data export.

**Emails (7)** -- List, remove, set primary, add with OTP verification flow.

**Phones (5)** -- List, remove, set primary, add with status polling.

**API Keys (4)** -- List, create, revoke, regenerate (2FA protected).

**2FA (4)** -- Start challenges, check status, verify codes, magic link verification.

**Billing (1)** -- Get subscription details.

**User Requests (7)** -- Manage verification requests from the user side: list, create, claim, cancel, extend, share.

### Customization (26 tools)

**Templates (7)** -- List, get defaults, get/update/delete by channel, preview, and render with variables.

**Projects (9)** -- Full project CRUD plus project-level template management.

**Profiles (10)** -- Profile CRUD, primary selection, proof display management.

### Infrastructure (30 tools)

**Domains (17)** -- Full domain lifecycle: add, verify (DNS/HTTP/email), connect providers (Cloudflare, GoDaddy, generic), credential management, email setup.

**DNS Credentials (3)** -- Store, list, and delete DNS provider credentials.

**Webhook Deliveries (4)** -- Stats, list, get details, retry failed deliveries.

**User Domain Verify (3)** -- User-side domain verification flow.

**Public Profiles (2)** -- Claim username, update public proof display.

## Transport

The server communicates over stdio (stdin/stdout) using the MCP protocol. This is compatible with all major MCP clients.

## Requirements

- Node.js >= 18.0.0
- A proof.holdings API key

## Next Steps

- [Get an API key](/dashboard/api-keys) from the dashboard
- Read the [API Reference](/docs/api) for endpoint details
- Explore the [SDKs](/docs/sdks) for programmatic access
