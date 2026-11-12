---
title: Proof of Delegation
lastUpdated: "2026-10-26"
status: current
---

# Proof of Delegation

> A domain you have proven authorizes a named artifact — a package or an endpoint — and can withdraw that authorization at any time.

---

## What this answers

An agent is about to install an MCP server, or call an API endpoint, that says it works on behalf of `example.com`. Nothing in the package or the endpoint can settle whether that is true: every field inside an artifact is written by whoever published the artifact.

Proof of Delegation puts the statement where the artifact cannot write it. The holder of `example.com` proves control of the domain, then issues a signed delegation naming the artifact. A verifier checks the signature against our published keys and checks that the artifact identity **it resolved independently** matches the one inside the token.

What a verified delegation establishes: the domain you expected authorized this artifact, and has not withdrawn that authorization. It does not establish that the artifact is safe, well-written, or the only one that domain has authorized. And it answers a question you must already have: you need to know which domain to expect before the answer means anything.

---

## The parts

| Piece | Who runs it | What it does |
|---|---|---|
| The delegation | The domain holder | A signed token naming an artifact and an expiry |
| [`@proof-holdings/delegation-verifier`](https://github.com/ProofHoldings/delegation-verifier) | Anyone checking | Verifies a delegation. No default trust list — you supply the issuers you trust |
| [`@proof-holdings/delegation-self-refusal`](https://github.com/ProofHoldings/delegation-self-refusal) | The publisher of the artifact | The artifact refuses its own calls once the delegation stops being valid |
| `@proof-holdings/delegation-publisher` | The domain holder | Builds and validates the card that carries the delegation |

**Not published yet.** `@proof-holdings/delegation-publisher` is not on npm, so it cannot be
installed; building a card by hand is the way in until it ships — the format is specified below —
or ask us. Nor are the Python and Go ports of the Proof gate this page describes under self-refusal:
`proof-holdings-mcp-gate` (PyPI) and `github.com/ProofHoldings/mcp-gate-go`. Its TypeScript package is on
npm.

---

## Issuing one

1. **Prove the domain.** Verify `example.com` from the dashboard or the API. That produces a control proof with a public handle (`ph_ctl_…`).
2. **Name the artifact.** Either a package (`pkg:npm/your-package`, `pkg:pypi/your-package`, `pkg:golang/github.com/you/module`, `pkg:composer/vendor/package`) or an endpoint (`https://api.example.com/mcp`). The identifier is canonicalized, so it has one spelling: npm, PyPI and Composer names are lowercased (a PyPI name also turns every run of `-`, `_` and `.` into one `-`, as PyPI itself does), while a Go module path keeps its capitals, because in Go a different case is a different module. A token's own `delegate` is canonicalized the same way before it is compared, so a delegation issued as `pkg:composer/Acme/Tool` still verifies for `pkg:composer/acme/tool`.
3. **Set the lifetime.** A delegation can never outlive the domain proof underneath it.

```
POST /api/v1/delegations
{
  "control_proof": "ph_ctl_…",
  "delegate": { "type": "purl", "value": "pkg:npm/your-package" },
  "expires_in": 31536000
}
```

The response carries the signed token and a public handle (`ph_dlg_…`). All four SDKs and the MCP server expose the same operation.

An endpoint's host is not checked against the issuing domain. `example.com` can delegate to `https://other.com/mcp` — deliberately, because an endpoint may run on a hosting provider's domain — and the same holds for packages, whose names say nothing about a domain at all. So what binds an artifact to you is the verifier, not the issuer: a verifier that passes `expectedPrincipal: "example.com"` rejects a delegation minted by any other domain. Without it, the verifier learns only that *some* proven domain claims the artifact.

---

## Publishing it

The token has to travel with the artifact, so a verifier finds it without asking you. There are five sanctioned carriers. Four are places inside a JSON document that hold the publication itself: `_meta` (an MCP server card, or a package's own file), the same `_meta` as the MCP registry stores it, an A2A agent card's extensions, and a `composer.json`'s `extra`. The fifth, a DNS record, holds only the address of such a card. The verifier package reads the four document locations itself; for the DNS record it parses the value, and looking the record up and fetching the card it names is the caller's job.

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

### Inside the package itself

A package can carry its own delegation, so it reaches everyone who installs it. The claim check on proof.holdings reads each of these from the package's latest release:

- **npm**: `package.json`, the same top-level `_meta["holdings.proof/delegation"]` object as an MCP server card. It is read from the published tarball, because `npm publish` drops `_`-prefixed keys from the registry's own manifest.
- **Go**: a `proof-delegation.json` file at the module root, holding `{"_meta": {"holdings.proof/delegation": {…}}}`. It is read from the module zip on the Go module proxy. Not `go.mod`, whose comments belong to the Go toolchain.
- **Composer**: `composer.json`, under `extra`:

```json
{
  "name": "vendor/package",
  "extra": {
    "holdings.proof/delegation": {
      "token": "eyJhbGc…",
      "principal": "example.com",
      "delegate": "pkg:composer/vendor/package"
    }
  }
}
```

Packagist serves `extra` as written and leaves out unknown top-level keys, so `_meta` would never reach a reader there.

A token in a package file ships with a release and changes with the next one. Renewing a delegation keeps it valid for Proof Holdings' own checks without a new release (see [Renewing it](#renewing-it)).

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

For the case where you control the domain but not the artifact's card: a `_mcp.<domain>` TXT record pointing at the card. The parser that reads it ships in [`@proof-holdings/delegation-verifier`](https://github.com/ProofHoldings/delegation-verifier), which is the executable statement of the format; its README describes the record in prose ([The fourth carrier: a DNS pointer](https://github.com/ProofHoldings/delegation-verifier#the-fourth-carrier-a-dns-pointer)).

Building any of these by hand is error-prone — the token's claims must name the canonicalized artifact identity. `@proof-holdings/delegation-publisher` will build and validate the card for you once it is on npm; until then, assemble it from the shapes above and read it back with `verifyPublishedDelegation` from `@proof-holdings/delegation-verifier` before you publish.

---

## Withdrawing it

Revocation is a single call, and it is not reversible:

```
POST /api/v1/delegations/:id/revoke
```

A pause is available too — `suspend` and `unsuspend` on the proof surface — for the case where you want verification to stop temporarily rather than permanently.

Withdrawal reaches a verifier three ways: the public validation endpoint (`POST /api/v1/proofs/validate`), the revocation list, and the status list. What it cannot do is reach into a card somebody already published. The token stays wherever it was put until that carrier removes it — which is why the dashboard reports, per delegation, where its token might still be visible.

### Renewing it

A delegation runs until its own expiry date, and never past the verification of its domain. To keep it, renew it before then — **Renew** on the Delegations page, or:

```
POST /api/v1/delegations/:id/renew
```

Renewing keeps the delegation: the same `ph_dlg_…` handle, principal, artifact and `jti`, with a new token whose `exp` is later. The new expiry is a full delegation lifetime from now, capped at the expiry of the domain's verification. When that verification is itself due, or its end is what would stop the delegation from running any longer, the renewal checks it again first, from the record you published, and renews the verification with it (same handle): if the record is still there, nothing else is asked of you; if it is gone, the renewal is refused (`409 domain_evidence_missing`) until you add it back on the Domains page. A delegation that has already expired can still be renewed, on control shown now: the renewal looks for the record you published (or a DNS-provider connection you saved) first, whatever the verification's own date, and refuses with `409 domain_evidence_missing` when it is gone; a domain that is not on your Domains page cannot be checked, so add it there first (`409 domain_recheck_required`). A renewal that would not add at least a day is refused (`409 delegation_not_extendable`) — verify the domain again first. In test mode the domain is not checked again: a test-mode delegation renews only within its test control proof's lifetime, and past it needs a new delegation from a new test verification. A revoked or suspended delegation cannot be renewed. A renewal is billed like issuing a delegation.

Every token carries `renewal_url`, where its current version is served (`GET /api/v1/proofs/delegation-token/:handle`), and it is the same for every token of the delegation, so an expired published copy points at its current version. A token issued before the claim existed gets it at its next renewal. The old token keeps verifying until its own `exp`; nothing is revoked. After a renewal, `GET /api/v1/proofs/delegation-token/:handle` serves the new token, and nothing needs republishing for Proof's own checks: the artifact lookup, its badge and `proof-check` judge a published copy by the delegation's current token, and from `@proof-holdings/delegation-self-refusal` 0.5.0 the TypeScript gate fetches the renewed token itself in the 14 days before its token expires. A checker that reads only the copy still stops accepting it on that copy's own date — a verifier of your own, or a gate before 0.5.0. For those, put the new token in every card that holds the old one (`@proof-holdings/delegation-publisher refresh --handle <handle> --in <card>` once that package is on npm; until then, by hand), or restart a server with an earlier gate with the new token. A `_mcp.<domain>` DNS pointer names a card, not a token, so it needs no change.

### Replacing a token

If a token has to stop verifying — it leaked, or the artifact identity changed — renewing does not help: the old token stays valid until it expires. Revoke the delegation, issue a new one, and republish the new token in every carrier that held the old one. Anything holding the old token is holding a revoked one, which is the correct outcome; but it also means the change is visible to every verifier as a revocation.

---

## Key rotation

This is about Proof's own signing keys, not your token. Proof signs delegations with a key it replaces every 30 days. A scheduled replacement changes nothing for you: the old key stays published for as long as a token it signed can live, so your token keeps verifying.

The keys Proof signed delegations with before this rotation scheme — the legacy keys — are retired once: Proof re-signs the live delegations they signed, as below, and revokes the legacy keys no earlier than 30 days after that re-signing. From then on a token you published before the re-signing answers `key_revoked` until you replace it with the re-signed one; a delegation the re-signing left on a legacy key has no re-signed token, and its token stops verifying when the legacy keys are revoked.

When Proof **revokes** a signing key — after a suspected compromise, or at the retirement of the legacy keys — the live delegations that key signed are re-signed (for the legacy keys, at least 30 days before): the same delegation and the same claims (`jti`, `exp`, principal, artifact), with a new signature and a new `iat`. The token you published then answers `key_revoked`: online within seconds (`POST /api/v1/proofs/validate` returns `valid: false, reason: "key_revoked"`), and offline in `@proof-holdings/delegation-verifier` once it has seen the revocation in two key-set fetches at least 5 minutes apart — its answer carries `pullHint: { issuerOrigin, handle }`, where the current token is. The handle is the token's own `sub`, and the current token is public:

```
GET /api/v1/proofs/delegation-token/:handle
```

It answers 404 for a delegation that is revoked, suspended, expired or unknown — there is no current token to serve.

What happens to each place your token sits:

- **A `_mcp.<domain>` DNS pointer** survives the rotation untouched. It names a card, never a token, so there is nothing in DNS to change.
- **An MCP server card or A2A agent card** carries the token itself. Put the current one in: `@proof-holdings/delegation-publisher refresh --handle <handle> --in <card>` does it once the package is on npm; until then, pull the token above and replace it in the card yourself.
- **Your own MCP server, running the Proof gate,** fetches the re-signed token by itself on its first `key_revoked` answer, in TypeScript, Python and Go alike. While it fetches, its tool calls are refused with a message saying so. The Python and Go releases are not on PyPI or the Go module proxy yet, so until they are, a Python or Go server pulls the token above and restarts with it.

---

## Self-refusal: enforcement that does not depend on the caller

A revoked delegation only reaches a verifier who asks, and a substantial share of callers never ask. So the delegated artifact carries the check itself.

The Proof gate installs in the publisher's own MCP server with one call: `@proof-holdings/delegation-self-refusal` for TypeScript, `proof-holdings-mcp-gate` for Python, `github.com/ProofHoldings/mcp-gate-go` for Go — see the note at the top of this page for which of them is not published yet. It checks the delegation's status when a tool is called, reuses the answer for five minutes by default (cached on disk, and refreshed in the background while it is under fifteen minutes old), and gates the server's own tool dispatch: once the delegation is revoked, suspended or expired, every tool call refuses with a message naming us, the domain and what happened. The caller does not have to check anything, and cannot skip the check by not asking.

While Proof is reachable, a revocation takes effect within about six minutes at the default interval, and at most one more call runs, only within fifteen minutes. While Proof cannot be reached, the last answer stands — a server that was valid keeps running, one that was refused keeps refusing — so a revocation made during an outage takes effect once Proof can be reached again.

It binds a **cooperative** publisher. A hostile operator can delete the wiring — that case is the verifier's job, not this package's, and the two are deliberately different products.

---

## What the gate adds to a server

Besides gating, the gate adds one tool and one line to the server's instructions:

- `proof_holdings_status` — reports the delegation's status (`valid`, `refused`, `unresolved` or `not_configured`), the domain and artifact its token names (to compare with the server you installed or connected to — the token is public, so another server can carry the same one), and when Proof last answered. It sits outside the gate, so it keeps answering while every other tool refuses. What each value means is set out at [proof.holdings/llms.txt](https://proof.holdings/llms.txt).
- `Verified by Proof Holdings (delegated by <domain>). Call proof_holdings_status for details.` — appended to the server's instructions, only when a token is configured. It is written once, when the server starts, and does not change when the delegation does: whether the delegation is in force is what `proof_holdings_status` reports.

Publishing the token is part of the setup, not an alternative to it: directories, `proof-check` and verifying clients read the published card (or DNS pointer) whatever the gate does.

---

## Verifying, from the other side

Three ways, depending on what you are:

**An agent with an MCP client.** Call `verify_delegation` on the Proof Holdings MCP server. It needs no account. Give it the artifact identity **you** resolved — the package you are about to install, the endpoint you are about to call — and the domain you expect. Never an identity read out of a file inside the artifact itself: that is the artifact describing itself, which is the thing being checked.

**A program.** Install `@proof-holdings/delegation-verifier`. Zero runtime dependencies, and no default trust list — you pass the issuers you are willing to trust on every call.

**Anything that can make an HTTP request.** `POST /api/v1/proofs/validate` with the token. Public, no key needed.

```
POST /api/v1/proofs/validate
{ "proof_token": "eyJhbGc…" }
```

---

## Connecting an agent to all of this

The Proof Holdings MCP server exposes the delegation operations along with the rest of the platform, and the hosted server needs no install — any MCP client with HTTP transport connects to it:

```json
{ "mcpServers": { "proof": { "type": "http", "url": "https://api.proof.holdings/mcp" } } }
```

`validate_proof`, `list_revoked_proofs`, `verify_delegation` and `get_delegation_token` work with no account at all — the party checking a delegation is usually not our customer. Issuing and revoking need an account. See [MCP Server](/docs/mcp) for the full surface.

---

## Resources

- [MCP Server](/docs/mcp) — the 193 tools, including the delegation ones
- [API Reference](/docs/api) — the delegation endpoints in full
- [How it works](/docs/primitives) — where delegation sits among the other proofs
- [Threat Model](/docs/threat-model) — what these proofs do and do not defend against
