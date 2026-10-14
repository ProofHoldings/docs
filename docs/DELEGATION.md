---
title: Proof of Delegation
lastUpdated: "2026-09-05"
status: current
---

# Proof of Delegation

> A domain you have proven authorizes a named artifact — a package or an endpoint — to act within a stated scope, and can withdraw that authorization at any time.

---

## What this answers

An agent is about to install an MCP server, or call an API endpoint, that says it works on behalf of `example.com`. Nothing in the package or the endpoint can settle whether that is true: every field inside an artifact is written by whoever published the artifact.

Proof of Delegation puts the statement where the artifact cannot write it. The holder of `example.com` proves control of the domain, then issues a signed delegation naming the artifact and the scopes it may act within. A verifier checks the signature against our published keys and checks that the artifact identity **it resolved independently** matches the one inside the token.

What a verified delegation establishes: the domain you expected authorized this artifact for these scopes, and has not withdrawn that authorization. It does not establish that the artifact is safe, well-written, or the only one that domain has authorized. And it answers a question you must already have: you need to know which domain to expect before the answer means anything.

---

## The parts

| Piece | Who runs it | What it does |
|---|---|---|
| The delegation | The domain holder | A signed token naming an artifact, its scopes and an expiry |
| [`@proof-holdings/delegation-verifier`](https://github.com/ProofHoldings/delegation-verifier) | Anyone checking | Verifies a delegation. No default trust list — you supply the issuers you trust |
| [`@proof-holdings/delegation-self-refusal`](https://github.com/ProofHoldings/delegation-self-refusal) | The publisher of the artifact | The artifact refuses its own calls once the delegation stops being valid |
| `@proof-holdings/delegation-publisher` | The domain holder | Builds and validates the card that carries the delegation |

**Not published yet.** Of the three, `@proof-holdings/delegation-publisher` alone is not on npm, so
it cannot be installed; the other two install normally. Building a card by hand is the way
in until it ships — the format is specified below — or ask us.

---

## Issuing one

1. **Prove the domain.** Verify `example.com` from the dashboard or the API. That produces a control proof with a public handle (`ph_ctl_…`).
2. **Name the artifact.** Either a package (`pkg:npm/your-package`, `pkg:pypi/your-package`) or an endpoint (`https://api.example.com/mcp`). The identifier is canonicalized, so it has one spelling.
3. **State the scopes.** Free-form capability tokens such as `send-email` — what this artifact may do in your name.
4. **Set the lifetime.** A delegation can never outlive the domain proof underneath it.

```
POST /api/v1/delegations
{
  "control_proof": "ph_ctl_…",
  "delegate": { "type": "purl", "value": "pkg:npm/your-package" },
  "scope": ["send-email"],
  "expires_in": 31536000
}
```

The response carries the signed token and a public handle (`ph_dlg_…`). All four SDKs and the MCP server expose the same operation.

An endpoint's host is not checked against the issuing domain. `example.com` can delegate to `https://other.com/mcp` — deliberately, because an endpoint may run on a hosting provider's domain — and the same holds for packages, whose names say nothing about a domain at all. So what binds an artifact to you is the verifier, not the issuer: a verifier that passes `expectedPrincipal: "example.com"` rejects a delegation minted by any other domain. Without it, the verifier learns only that *some* proven domain claims the artifact.

---

## Publishing it

The token has to travel with the artifact, so a verifier finds it without asking you. There are four sanctioned carriers: three are places inside a card that hold the publication itself — an MCP server card, the same card as the MCP registry stores it, and an A2A agent card — and the fourth, a DNS record, holds only the address of such a card. The verifier package reads the three card locations itself; for the DNS record it parses the value, and looking the record up and fetching the card it names is the caller's job.

### MCP server card

For an MCP server, the delegation goes in the server card's `_meta`, under the `holdings.proof/delegation` key:

```json
{
  "name": "your-package",
  "_meta": {
    "holdings.proof/delegation": {
      "token": "eyJhbGc…",
      "principal": "example.com",
      "delegate": "pkg:npm/your-package"
    }
  }
}
```

The MCP registry nests publisher-supplied metadata one level deeper, under `_meta["io.modelcontextprotocol.registry/publisher-provided"]`, so there the same object sits at `_meta["io.modelcontextprotocol.registry/publisher-provided"]["holdings.proof/delegation"]`. The verifier reads both shapes.

### A2A agent card

For an A2A agent, it goes in `capabilities.extensions[]`, identified by uri:

```json
{
  "capabilities": {
    "extensions": [
      {
        "uri": "https://proof.holdings/delegation",
        "params": {
          "token": "eyJhbGc…",
          "principal": "example.com",
          "delegate": "https://api.example.com/mcp"
        }
      }
    ]
  }
}
```

### DNS pointer

For the case where you control the domain but not the artifact's card: a `_mcp.<domain>` TXT record pointing at the card. The parser that reads it ships in [`@proof-holdings/delegation-verifier`](https://github.com/ProofHoldings/delegation-verifier), which is the executable statement of the format — ask us if you need the prose specification.

Building any of these by hand is error-prone — the token's claims must match the canonicalized artifact identity byte for byte. `@proof-holdings/delegation-publisher` will build and validate the card for you once it is on npm; until then, assemble it from the shapes above and read it back with `verifyPublishedDelegation` from `@proof-holdings/delegation-verifier` before you publish.

---

## Withdrawing it

Revocation is a single call, and it is not reversible:

```
POST /api/v1/delegations/:id/revoke
```

A pause is available too — `suspend` and `unsuspend` on the proof surface — for the case where you want verification to stop temporarily rather than permanently.

Withdrawal reaches a verifier three ways: the public status lookup on the handle, the revocation list, and the status list. What it cannot do is reach into a card somebody already published. The token stays wherever it was put until that carrier removes it — which is why the dashboard reports, per delegation, where its token might still be visible.

### There is no token rotation

A delegation's token cannot be replaced while keeping the delegation. There is no rotate operation, and none is planned in this version.

If a token needs to change — it leaked, or the artifact identity changed — the sequence is: revoke the delegation, issue a new one, and republish the new token in every carrier that held the old one. Anything holding the old token is holding a revoked one, which is the correct outcome; but it also means a rotation is visible to every verifier as a revocation, not as a quiet key change.

---

## Self-refusal: enforcement that does not depend on the caller

A revoked delegation only reaches a verifier who asks, and a substantial share of callers never ask. So the delegated artifact carries the check itself.

`@proof-holdings/delegation-self-refusal` installs in the publisher's own MCP server. It checks the delegation's status when a tool is called, reuses the answer for about a minute (cached on disk), and gates the server's own tool dispatch: once the delegation is revoked, suspended or expired, every tool call refuses with a message naming Proof, the domain and what happened. The caller does not have to check anything, and cannot skip the check by not asking.

It binds a **cooperative** publisher. A hostile operator can delete the wiring — that case is the verifier's job, not this package's, and the two are deliberately different products.

---

## The showcase

A publisher who installs the layer with `installProofLayer` (instead of the gate-only `guardDelegation`) also registers three tools inside their own MCP server:

- `proof_check_this_server` — reports this server's own delegation status, the artifact and domain its token names (to compare with what you connected to), and the exact refusal text its callers would see
- `proof_verify_delegation` — checks any other artifact's delegation
- `proof_connect` — the current instructions for connecting to the full Proof server

These are registered through the un-gated registrar on purpose: a revoked delegation must not be able to silence the tool that reports the revocation.

---

## Verifying, from the other side

Three ways, depending on what you are:

**An agent with an MCP client.** Call `verify_delegation` on the Proof MCP server. It needs no account. Give it the artifact identity **you** resolved — the package you are about to install, the endpoint you are about to call — and the domain you expect. Never an identity read out of a file inside the artifact itself: that is the artifact describing itself, which is the thing being checked.

**A program.** Install `@proof-holdings/delegation-verifier`. Zero runtime dependencies, and no default trust list — you pass the issuers you are willing to trust on every call.

**Anything that can make an HTTP request.** `POST /api/v1/proofs/validate` with the token. Public, no key needed.

```
POST /api/v1/proofs/validate
{ "token": "eyJhbGc…" }
```

---

## Connecting an agent to all of this

The Proof MCP server exposes the delegation operations along with the rest of the platform, and the hosted server needs no install — any MCP client with HTTP transport connects to it:

```json
{ "mcpServers": { "proof": { "type": "http", "url": "https://api.proof.holdings/mcp" } } }
```

`validate_proof`, `list_revoked_proofs` and `verify_delegation` work with no account at all — the party checking a delegation is usually not our customer. Issuing and revoking need an account. See [MCP Server](/docs/mcp) for the full surface.

---

## Resources

- [MCP Server](/docs/mcp) — the 172 tools, including the delegation ones
- [API Reference](/docs/api) — the delegation endpoints in full
- [Core Primitives](/docs/primitives) — where delegation sits among the other proofs
- [Threat Model](/docs/threat-model) — what these proofs do and do not defend against
