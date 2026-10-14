---
title: MCP Server
lastUpdated: "2026-09-05"
status: current
---

# MCP Server

> 172 tools for AI agents — verify assets, manage approvals, and automate trust workflows

---

## What is MCP?

The [Model Context Protocol](https://modelcontextprotocol.io) (MCP) lets AI assistants call tools directly. The proof.holdings MCP server wraps our entire REST API into 172 structured tools that any MCP-compatible agent can use — Claude, LangChain, CrewAI, OpenAI Agents SDK, Cursor, Windsurf, and more.

---

## Two ways to connect

There is a hosted server and a local one. They expose the same tools; what differs is what you have to install and how you authenticate.

| | Hosted (remote) | Local (npm) |
|---|---|---|
| Install | nothing | `npx -y @proof-holdings/mcp-server` |
| Credential to start | none | none (`PROOF_API_KEY` is optional) |
| How an account is reached | browser sign-in (OAuth) from the client | `PROOF_API_KEY` in the environment |
| Transport | HTTP | stdio |

Start with the hosted server unless you need a local process — it needs no install, and signing in happens in the client rather than by pasting a key into a config file.

### Hosted server

The hosted server is at `https://api.proof.holdings/mcp`. Any MCP client or agent framework with HTTP transport can connect — add the URL as a remote MCP server in its settings (the config key varies by client). As a config fragment:

```json
{ "mcpServers": { "proof": { "type": "http", "url": "https://api.proof.holdings/mcp" } } }
```

The connection succeeds immediately and anonymously. The first tool that needs an account answers with a `401` carrying an OAuth challenge, and the client offers to authenticate — in Claude Code that is `/mcp` → **Authenticate**. You sign in and approve the request in a browser; the client stores the result and sends it from then on.

Approving that request creates a scoped API key on your account, named after the client. Revoking access is deleting that key in the dashboard.

### Local server

```bash
npx -y @proof-holdings/mcp-server
```

It starts with no configuration. Supplying `PROOF_API_KEY` is what gives it an account from the first call.

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PROOF_API_KEY` | No | — | Your API key (`pk_live_*` or `pk_test_*`). Without it the server still starts. |
| `PROOF_BASE_URL` | No | `https://api.proof.holdings` | API base URL |

---

## What works without an account

The server starts and lists all 172 tools whether or not it holds a credential — an agent can always see what is available. Ten tools then work with no account at all, in four groups:

- **Checking somebody else's proof** — `validate_proof` verifies a signed token against our published keys, and `list_revoked_proofs` reads the revocation list.
- **Checking somebody else's delegation** — `verify_delegation` answers whether one artifact is authorized by the domain it claims, and `verify_delegations` re-checks a set of them. This is the half an agent reaches for before installing something.
- **Signing in** — `start_login` and `wait_for_login` run a browser or messenger sign-in and return a session.
- **Creating an account** — `create_account`, `send_account_email` and `wait_for_account_creation` bootstrap a new account from inside the agent.

Plus `render_auth_link`, which formats a sign-in link locally and calls nothing.

The verification tools are open on purpose: the party checking a proof or a delegation is usually not our customer, and a check that required an account would not get run.

Everything else needs an account. Those tools do not fail obscurely: each says an account is needed and how to get one.

Signing in with `start_login` gives a session, and a session reaches 57 of the tools — the ones that act on your own account through the dashboard's own authentication. The remaining tools need an API key (or the hosted server's OAuth flow, which mints one).

---

## Client setup

### Claude Code CLI

```bash
# hosted
claude mcp add --transport http proof https://api.proof.holdings/mcp

# local
claude mcp add proof-holdings -- npx -y @proof-holdings/mcp-server \
  --env PROOF_API_KEY=pk_live_YOUR_API_KEY
```

### Claude Desktop, Cursor, Windsurf

Every client below takes the same two shapes. The hosted one needs no key:

```json
{
  "mcpServers": {
    "proof": {
      "type": "http",
      "url": "https://api.proof.holdings/mcp"
    }
  }
}
```

The local one runs the package, and `PROOF_API_KEY` is optional — without it the server still starts and the keyless tools above work:

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

`GET /api/v1/mcp/connect` serves both fragments live, so an agent can read the current ones rather than copying from here.

Config file locations:

| Client | Config file |
|---|---|
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Desktop (Windows) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Cursor | `.cursor/mcp.json` in your project |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` |

## Tool Overview (172 tools)

Group totals below are exact. The tables name the tools you are most likely to reach for rather than all 171 — an MCP client's own `tools/list` is the complete, current list, and it is the one the server answers from.

### Verifications & requests (29 tools)

Asset control challenges, multi-channel phone verification, and the multi-asset B2B request flow.

| Tool | Description |
|------|-------------|
| `create_verification` | Create a new verification challenge |
| `get_verification` | Get verification status |
| `submit_verification_code` | Submit a verification code |
| `trigger_verification` | Trigger a verification check |
| `wait_for_verification` | Poll until verification completes |
| `create_multi_channel_verification` | Challenge one phone across up to three channels, first completion wins |
| `create_verification_request` | Create a multi-asset request |
| `get_request_by_reference` | Get a request by reference ID |
| `share_request_email` | Email a request to its recipient |

### Domains & DNS (23 tools)

Domain control proofs, DNS provider credentials, and the email-based domain flow.

| Tool | Description |
|------|-------------|
| `add_domain` | Add a domain to verify |
| `verify_domain` | Check the challenge record and mint the proof |
| `connect_cloudflare` | Connect Cloudflare so records are written for you |
| `verify_domain_with_credentials` | Prove control using stored provider credentials |
| `setup_domain_email` | Set up sending from the domain |
| `list_dns_credentials` | List stored DNS provider credentials |

### Account, settings & billing (37 tools)

Everything about your own account: usage, keys, contact identifiers, 2FA, and the agent account bootstrap.

| Tool | Description |
|------|-------------|
| `get_platform_summary` | One-call snapshot of the account |
| `get_usage` | Quota and usage for the period |
| `search` | Search across the account |
| `create_account` | Bootstrap a new account from inside an agent |
| `create_api_key` | Create a scoped API key |
| `list_assets` | List verified assets and their proof handles |
| `start_2fa` | Begin a 2FA step-up |

### HITL approvals & consent (22 tools)

Human-in-the-loop approvals, their end-to-end encrypted key material, and the consent layer underneath them.

| Tool | Description |
|------|-------------|
| `create_hitl` | Create a human-approval configuration |
| `create_confirmation` | Send an approval request to a person |
| `wait_for_confirmation` | Poll until a person approves or denies |
| `get_confirmation_approval_link` | Get a fresh approval link |
| `create_authorization` | Ask a person to consent to being contacted |
| `revoke_authorization` | Withdraw a consent |

### Circles & Proof of Me (15 tools)

The people primitive: circles of trusted contacts, their channels, and cross-channel identity challenges.

| Tool | Description |
|------|-------------|
| `create_circle` | Create a circle |
| `add_circle_member` | Add a trusted contact |
| `invite_circle_member` | Send a single-use enrollment link |
| `add_circle_member_channel` | Add a channel to a member |
| `create_identity_challenge` | Run a cross-channel identity check |

### Public profiles (16 tools)

Public verification pages, the proofs they display, and their templates.

| Tool | Description |
|------|-------------|
| `create_profile` | Create a public profile |
| `claim_username` | Claim a public username |
| `update_public_proofs` | Choose which proofs a profile shows |
| `preview_profile_template` | Preview a profile template |

### Templates & webhooks (11 tools)

Message templates for outbound channels, and the webhook delivery log.

| Tool | Description |
|------|-------------|
| `list_templates` | List message templates |
| `update_template` | Change a template |
| `render_template` | Render a template with variables |
| `list_webhook_deliveries` | List webhook deliveries |
| `retry_webhook_delivery` | Retry a failed delivery |

### Delegations & proofs (10 tools)

Proof of Delegation — a proven domain authorizing an artifact — plus the proof lifecycle and the public verifier surface.

| Tool | Description |
|------|-------------|
| `create_delegation` | Authorize an artifact from a domain you have proven |
| `list_delegations` | List your delegations |
| `revoke_delegation` | Revoke a delegation |
| `verify_delegation` | Check whether an artifact is authorized by the domain it claims |
| `validate_proof` | Verify a signed proof token (no account needed) |
| `get_proof_status` | Read a proof's status by its public handle |
| `list_revoked_proofs` | Read the revocation list (no account needed) |

`verify_delegation` is the one to reach for when an agent is deciding whether to trust a server or package. It answers about an artifact identity **you** resolved independently — a package you are installing, an endpoint you are calling — never one read out of the artifact's own files.

### Sign-in & sessions (9 tools)

Browser and messenger sign-in, and the sessions it produces.

| Tool | Description |
|------|-------------|
| `start_login` | Begin a sign-in (no account needed) |
| `wait_for_login` | Poll until sign-in completes |
| `get_current_user` | Who the current session belongs to |
| `list_auth_sessions` | List active sessions |
| `render_auth_link` | Render a sign-in link for the user to open |

---

## Example: Verify a phone number

```
Agent: I need to verify that +1-555-0123 belongs to this user.

→ create_verification({ type: "phone", identifier: "+15550123", channel: "sms" })
← { id: "ver_abc123", status: "pending", expires_at: "..." }

→ wait_for_verification({ id: "ver_abc123" })
← { status: "verified", proof_token: "eyJhbGc..." }
```

## Example: Request human approval

```
Agent: This wire transfer needs a human to approve it.

→ create_confirmation({ hitl_id: "hitl_abc", message: "<ciphertext>" })
← { id: "conf_xyz", status: "pending", timeout_at: "..." }

→ wait_for_confirmation({ id: "conf_xyz" })
← { status: "resolved", response: { action: "approve" }, proof_token: "eyJhbGc..." }
```

## Example: Check whether a server may act for a domain

```
Agent: This MCP server says it works on behalf of example.com. Is that true?

→ verify_delegation({ delegate: "pkg:npm/their-package", principal: "example.com" })
← { verified: true, scope: ["send-email"], expires_at: "..." }
```

---

## Test mode

A `pk_test_*` key puts every tool in test mode: verifications complete without sending real messages, and test data is kept apart from production data. `test_verify` completes a test verification directly.

---

## Resources

- [API Reference](/docs/api) — the REST API these tools wrap
- [Proof of Delegation](/docs/delegation) — the delegation module in full
- [SDKs](/docs/sdks) — JavaScript, Python, Go and PHP clients
- [Core Primitives](/docs/primitives) — the mental model in one page
