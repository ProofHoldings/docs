---
title: MCP Server
lastUpdated: "2026-11-12"
status: current
---

# MCP Server

> 193 tools for AI agents — verify assets, manage approvals, and automate trust workflows

---

## What is MCP?

The [Model Context Protocol](https://modelcontextprotocol.io) (MCP) lets AI assistants call tools directly. The Proof Holdings MCP server wraps the REST API into 193 structured tools — 192 for the API plus one delegation status tool — that any MCP-compatible agent can use — Claude, LangChain, CrewAI, OpenAI Agents SDK, Cursor, Windsurf, and more.

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

It starts with no configuration. To give it an account from the first call, set `PROOF_API_KEY`, or sign in once with `npx @proof-holdings/mcp-server login`; the server reads the saved key. `PROOF_API_KEY` wins when both are present.

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PROOF_API_KEY` | No | — | Your API key (`pk_live_*` or `pk_test_*`). Without it the server still starts. |
| `PROOF_BASE_URL` | No | `https://api.proof.holdings` | API base URL |

---

## What works without an account

The server starts and lists all 193 tools whether or not it holds a credential — an agent can always see what is available. Eleven tools then work with no account at all, in four groups:

- **Checking somebody else's proof** — `validate_proof` verifies a signed token against our published keys, and `list_revoked_proofs` reads the revocation list.
- **Checking somebody else's delegation** — `verify_delegation` answers whether one artifact is authorized by the domain it claims, and `verify_delegations` re-checks a set of them. This is the half an agent reaches for before installing something. `get_delegation_token` fetches an active delegation's current token by its public handle — the token is published with the delegation anyway.
- **Signing in** — `start_login` and `wait_for_login` run a browser or messenger sign-in and return a session.
- **Creating an account** — `create_account`, `send_account_email` and `wait_for_account_creation` bootstrap a new account from inside the agent.

Plus `render_auth_link`, which formats a sign-in link locally and calls nothing.

The verification tools are open on purpose: the party checking a proof or a delegation is usually not our customer, and a check that required an account would not get run.

Everything else needs an account. Those tools do not fail obscurely: each says an account is needed and how to get one.

Signing in with `start_login` gives a session, and a session reaches 58 of the tools — the ones that act on your own account through the dashboard's own authentication. The remaining tools need an API key (or the hosted server's OAuth flow, which mints one).

---

## Client setup

### Claude Code CLI

```bash
# hosted
claude mcp add --transport http proof https://api.proof.holdings/mcp

# local
claude mcp add proof-holdings --env PROOF_API_KEY=pk_live_YOUR_API_KEY -- npx -y @proof-holdings/mcp-server
```

### Claude Desktop: the hosted server as a connector

Claude Desktop adds the hosted server as a custom connector. Two of the options the dialog picks for you do not work with our server, so change them before you save. These steps were checked in Claude Desktop 5.1231.

1. Open **Settings → Customize → Connectors**, choose **+ Add**, then the custom connector. The dialog is titled **Add custom connector**.
2. Give it a name and the URL `https://api.proof.holdings/mcp`.
3. Under **Authentication**, choose **Sign in when needed**. Desktop picks **No sign-in** for you (marked "Detected"), which is for servers without OAuth sign-in; ours signs in with OAuth, and it offers some tools without an account.
4. Under **OAuth client**, which appears once you have done step 3, choose **Register automatically**. Desktop picks **Use Claude's published identity** for you, and our server does not support that yet.
5. Leave **Request headers** empty and choose **Add**.

The first tool that needs an API key (for example `get_self_api_key`) then shows **Authentication required to use this tool** with a **Connect** button. Claude asks you to finish connecting, then our page **Connect an MCP client** opens with the request. Approve it and confirm with the code we send you; the tool then runs.

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

Config file locations:

| Client | Config file |
|---|---|
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Desktop (Windows) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Cursor | `.cursor/mcp.json` in your project |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` |

## Tool Overview (193 tools)

**Renamed in 1.10.0.** 23 of the tools took the dashboard's names for what they act on — approval channel, Your proofs, sub-account, Public page. Their old names still answer, hosted and local alike, as deprecated aliases listed in the changelog, until 2.0.0: update any permission rule, skill or script that names a tool by hand before then.

Group totals below are exact. The tables name the tools you are most likely to reach for rather than all 193 — an MCP client's own `tools/list` is the complete, current list, and it is the one the server answers from.

### Verifications & requests (28 tools)

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

### Account, settings & billing (42 tools)

Everything about your own account: usage, keys, contact identifiers, 2FA, and the agent account bootstrap.

| Tool | Description |
|------|-------------|
| `get_platform_summary` | One-call snapshot of the account |
| `get_usage` | Quota and usage for the period |
| `get_balance` | The prepaid balance and the auto top-up setting |
| `search` | Search across the account |
| `create_account` | Bootstrap a new account from inside an agent |
| `create_api_key` | Create a scoped API key |
| `list_my_proofs` | List verified assets and their proof handles |
| `start_2fa` | Begin a 2FA step-up |

### Approval channels & consent (23 tools)

Human-in-the-loop approvals, their end-to-end encrypted key material, and the consent layer underneath them.

| Tool | Description |
|------|-------------|
| `create_approval_channel` | Create a human-approval configuration |
| `create_confirmation` | Send an approval request to a person |
| `resend_confirmation` | Send a pending approval request again (a charged send) |
| `wait_for_confirmation` | Poll until a person approves or denies |
| `get_confirmation_approval_link` | Get the approval link the config owner decides with |
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

### Sub-accounts and the Public page (14 tools)

Public verification pages and the proofs they display. Message templates belong to each sub-account and are in the Templates & webhooks group.

| Tool | Description |
|------|-------------|
| `create_sub_account` | Create a public profile |
| `claim_username` | Claim a public username |
| `update_public_page_proofs` | Choose which proofs a profile shows |
| `rotate_callback_secret` | Replace a sub-account's callback signing secret (returned once) |

### Templates & webhooks (17 tools)

Message templates for outbound channels, webhook endpoints, and the webhook delivery log. Templates belong to one sub-account: the template tools take an optional `profile_id`, and without one a sub-account key works on its own sub-account and an account-wide key on the primary sub-account.

| Tool | Description |
|------|-------------|
| `list_templates` | List a sub-account's message templates |
| `update_template` | Change a template |
| `render_template` | Render a template with variables |
| `list_webhook_deliveries` | List webhook deliveries |
| `retry_webhook_delivery` | Retry a failed delivery |
| `create_webhook_endpoint` | Add an endpoint for all sub-accounts or selected ones; returns its signing secret once |
| `update_webhook_endpoint` | Change an endpoint's URL, events, sub-accounts or status |
| `rotate_webhook_endpoint_secret` | Replace an endpoint's signing secret (returned once) |

### Delegations & proofs (16 tools)

Proof of Delegation — a proven domain authorizing an artifact — plus the proof lifecycle and the public verifier surface.

| Tool | Description |
|------|-------------|
| `create_delegation` | Authorize an artifact from a domain you have proven |
| `list_delegations` | List your delegations |
| `revoke_delegation` | Revoke a delegation |
| `verify_delegation` | Check whether an artifact is authorized by the domain it claims |
| `validate_proof` | Verify a signed proof token (no account needed) |
| `get_proof_status` | Read a proof's status by its public handle |
| `pause_proof` | Pause a proof or delegation (reversible) |
| `resume_proof` | Resume a paused proof or delegation |
| `list_revoked_proofs` | Read the revocation list (no account needed) |
| `get_proof_token` | Fetch the current (re-signed) token of a proof you own (not one of your own verified assets: that is read in the dashboard) |
| `list_reissued_proofs` | List your proofs re-signed since an instant, except those of your own verified assets |
| `get_delegation_token` | Fetch an active delegation's current token by handle (no account needed) |

`verify_delegation` is the one to reach for when an agent is deciding whether to trust a server or package. It answers about an artifact identity **you** resolved independently — a package you are installing, an endpoint you are calling — never one read out of the artifact's own files.

### Lookalike Watch (5 tools)

Watch newly published packages and MCP-registry entries for names confusable with your brand, with alerts and a monthly report.

| Tool | Description |
|------|-------------|
| `subscribe_lookalike_watch` | Turn the watch on for your brand tokens |
| `list_lookalike_alerts` | Read the alerts it raised |
| `list_lookalike_reports` | Read the monthly reports |

### Sign-in & sessions (9 tools)

Browser and messenger sign-in, and the sessions it produces.

| Tool | Description |
|------|-------------|
| `start_login` | Begin a sign-in (no account needed) |
| `wait_for_login` | Poll until sign-in completes |
| `get_current_user` | Who the current session belongs to |
| `list_auth_sessions` | List active sessions |
| `render_auth_link` | Render a sign-in link for the user to open |

### Delegation status (1 tool)

The server's own delegation check: this package is delegated by proof.holdings, and its Proof gate refuses every tool call once that delegation is revoked, suspended or expired. Registered by the gate, outside the API tools.

| Tool | Description |
|------|-------------|
| `proof_holdings_status` | Who delegated this server, and whether that delegation is in force (no account needed) |

---

## Example: Verify a phone number

```
Agent: I need to verify that +1-555-0123 belongs to this user.

→ create_verification({ type: "phone", identifier: "+15550123", channel: "sms" })
← { id: "6650f1c2a9e4b3d2c1a0f9e8", status: "pending", challenge: { ... } }

→ wait_for_verification({ id: "6650f1c2a9e4b3d2c1a0f9e8" })
← { status: "verified", proof: { token: "eyJhbGc...", expires_at: "..." } }
```

## Example: Request human approval

The wait ends `approved` or `denied`.

```
Agent: This wire transfer needs a human to approve it.

→ create_confirmation({ hitl_id: "6650f1c2a9e4b3d2c1a0f9e1", message: "<ciphertext envelope>" })
← { id: "6650f1c2a9e4b3d2c1a0f9e2", status: "pending", timeout_at: "..." }

→ wait_for_confirmation({ id: "6650f1c2a9e4b3d2c1a0f9e2" })
← { status: "approved", response: { action: "approve", ... }, proof_token: "eyJhbGc..." }
```

## Example: Check whether a server may act for a domain

Pass exactly one of `card` or `token`, the artifact identity you resolved yourself as `delegate`, and the domain you expect as `expected_principal`.

```
Agent: This MCP server says it works on behalf of proof.holdings. Is that true?

→ verify_delegation({
    card: { ...the server card... },
    delegate: { type: "purl", value: "pkg:npm/@proof-holdings/mcp-server" },
    expected_principal: "proof.holdings"
  })
← { verified: true, status_checked: true, principal_checked: true,
    attests: "proof.holdings authorized pkg:npm/@proof-holdings/mcp-server",
    delegation: { principal: "proof.holdings", delegate: "pkg:npm/@proof-holdings/mcp-server",
                  expires_at: "..." }, ... }
```

---

## Test mode

A `pk_test_*` key puts every tool in test mode: verifications complete without sending real messages, and test data is kept apart from production data. `test_verify` completes a test verification directly.

---

## Resources

- [API Reference](/docs/api) — the REST API these tools wrap
- [Proof of Delegation](/docs/delegation) — the delegation module in full
- [SDKs](/docs/sdks) — JavaScript, Python, Go and PHP clients
- [How it works](/docs/primitives) — the mental model in one page
