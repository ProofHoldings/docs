---
title: Platform Integrations
lastUpdated: "2026-02-27"
status: current
---

# Platform Integrations

Connect proof.holdings to your automation workflows with official integrations for Zapier, n8n, and Make.com. Each platform provides the same 17 capabilities: triggers, actions, and searches for identity verification.

## Capabilities

All three platforms share the same capability set:

| Type | Capabilities |
|------|-------------|
| **Triggers** (4) | Verification Completed, Verification Failed, Verification Expired, Request Completed |
| **Actions** (8) | Create Phone/Email/Domain/Telegram Bot Verification, Create Verification Request, Validate Proof, Revoke Proof, Update Profile |
| **Searches** (5) | Check Verification, Check Verification Request, List Verifications, List Revoked Proofs, Get Profile |

## Zapier

The Zapier connector wraps the official JavaScript SDK to provide polling triggers, create actions, and searches.

### Setup

1. Open a Zap and search for **Proof Holdings**
2. Connect your account with your API key (`pk_live_...` or `pk_test_...`)
3. Select a trigger or action

### Triggers

Zapier triggers use polling to check for new events:

- **Verification Completed** -- fires when a verification reaches `verified` status
- **Verification Failed** -- fires when a verification fails
- **Verification Expired** -- fires when a verification expires
- **Request Completed** -- fires when a multi-asset request is fully completed

### Actions

- **Create Phone Verification** -- verify via SMS, WhatsApp, Telegram, or Viber
- **Create Email Verification** -- verify an email address
- **Create Domain Verification** -- verify domain ownership via DNS or HTTP
- **Create Telegram Bot Verification** -- verify a Telegram bot token
- **Create Verification Request** -- multi-asset B2B verification request
- **Validate Proof Token** -- validate a JWT proof token
- **Revoke Proof** -- revoke a proof by verification ID
- **Update Profile** -- update a public profile

### Searches

- **Check Verification Status** -- get verification by ID
- **Check Verification Request** -- get request by ID
- **List Verifications** -- filter by status and type
- **List Revoked Proofs** -- get all revoked proofs
- **Get Public Profile** -- look up by username

## n8n

The n8n community node provides the same capabilities as a native n8n node with a trigger companion.

### Installation

Follow the [n8n community nodes installation guide](https://docs.n8n.io/integrations/community-nodes/installation/) to install `n8n-nodes-proof-holdings`.

### Credentials

Add a **Proof Holdings API** credential in n8n:

- **API Key**: Your `pk_live_...` (production) or `pk_test_...` (test) key
- **Base URL**: Defaults to `https://api.proof.holdings`

### Nodes

**Proof Holdings** -- main node with resource/operation selector:

| Resource | Operations |
|----------|-----------|
| Verification | Create Phone, Create Email, Create Domain, Create Telegram Bot, Check Status, List |
| Verification Request | Create, Check Status |
| Proof | Validate, List Revoked, Revoke |
| Profile | Get, Update |

**Proof Holdings Trigger** -- polling trigger node for Verification Completed, Failed, Expired, and Request Completed events.

## Make.com

The Make.com custom app uses declarative JSON modules to call the API directly -- no SDK dependency.

### Setup

1. Install the **Proof Holdings** custom app from the Make.com Apps Editor
2. Create a connection with your API key
3. Add Proof Holdings modules to your scenario

### Modules

Make.com modules are organized as actions, searches, and triggers:

**Actions (8):** Create Phone/Email/Domain/Telegram Bot Verification, Create Verification Request, Validate Proof, Revoke Proof, Update Profile

**Searches (5):** Check Verification, Check Verification Request, List Verifications, List Revoked Proofs, Get Profile

**Triggers (4):** Verification Completed, Verification Failed, Verification Expired, Request Completed

## Authentication

All three platforms use API key authentication:

- **Production**: `pk_live_...` keys for live verifications
- **Test mode**: `pk_test_...` keys for development (no real messages sent)

Get your API key from the [dashboard](/dashboard/api-keys).

## Test Mode

All integrations support test mode. Use a `pk_test_...` API key to:

- Create verifications without sending real messages
- Auto-complete verifications with the test-verify endpoint
- Validate proof tokens in a sandboxed environment

## Next Steps

- [Get an API key](/dashboard/api-keys) from the dashboard
- Read the [API Reference](/docs/api) for endpoint details
- Explore the [SDKs](/docs/sdks) for programmatic integration
- Set up the [MCP Server](/docs/mcp) for AI agent access
