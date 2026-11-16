---
title: SDKs
lastUpdated: "2026-10-26"
status: current
---

# Official SDKs

Install an official SDK to start verifying assets in minutes. All SDKs provide the same features: typed clients, automatic retries with exponential backoff, polling helpers, and structured error handling.

| Language | Package | Install |
|----------|---------|---------|
| JavaScript/TypeScript | `@proof-holdings/sdk` | `npm install @proof-holdings/sdk` |
| Python | `proof-holdings-sdk` | `pip install proof-holdings-sdk` |
| PHP | `proof-holdings/sdk` | `composer require proof-holdings/sdk` |
| Go | `github.com/ProofHoldings/sdk-go` | `go get github.com/ProofHoldings/sdk-go` |

---

## Quick Start

### JavaScript / TypeScript

```typescript
import { Proof } from '@proof-holdings/sdk';

const proof = new Proof('pk_live_your_key');

// Create a phone verification
const verification = await proof.verifications.create({
  type: 'phone',
  channel: 'whatsapp',
  identifier: '+37069199199',
});

console.log(verification.challenge.deep_link);
// "https://wa.me/15557766633?text=Verify%20X7K2M9"

// Poll until verified (or timeout)
const result = await proof.verifications.waitForCompletion(verification.id);
console.log(result.proof.token);
// "eyJhbGciOiJFUzI1NiI..."
```

### Python

```python
import asyncio
from proof_sdk import Proof

async def main():
    async with Proof("pk_live_your_key") as proof:
        # Create a phone verification
        verification = await proof.verifications.create(
            type="phone",
            channel="telegram",
            identifier="+37069199199",
        )

        print(verification["challenge"]["deep_link"])

        # Poll until verified
        result = await proof.verifications.wait_for_completion(verification["id"])
        print(result["proof"]["token"])

asyncio.run(main())
```

### PHP

```php
use ProofHoldings\Proof;

$proof = new Proof('pk_live_your_key');

// Create a phone verification
$verification = $proof->verifications->create([
    'type' => 'phone',
    'channel' => 'sms',
    'identifier' => '+37069199199',
]);

echo $verification['challenge']['code'];

// Poll until verified
$result = $proof->verifications->waitForCompletion($verification['id']);
echo $result['proof']['token'];
```

### Go

```go
package main

import (
    "context"
    "fmt"
    proof "github.com/ProofHoldings/sdk-go"
)

func main() {
    client, _ := proof.NewClient("pk_live_your_key")

    // Create a phone verification
    verification, _ := client.Verifications.Create(context.Background(), map[string]any{
        "type":       "phone",
        "channel":    "whatsapp",
        "identifier": "+37069199199",
    })

    fmt.Println(verification["challenge"].(map[string]any)["deep_link"])

    // Poll until verified
    result, _ := client.Verifications.WaitForCompletion(
        context.Background(),
        verification["id"].(string),
        nil,
    )
    fmt.Println(result["proof"].(map[string]any)["token"])
}
```

---

## Resources

These are the five resources most integrations start with. Every client has more — confirmations, HITL approvals, circles, delegations, Proof of Me, public profiles, templates and others — each mapping to the REST API.

A **session** is a phone-first verification: the person proves a phone number over Telegram, WhatsApp or SMS without you supplying the number first.

| Resource | Methods |
|----------|---------|
| `verifications` | `create`, `retrieve`, `list`, `verify`, `submit`, `testVerify`, `waitForCompletion` |
| `sessions` | `create`, `retrieve`, `waitForCompletion` |
| `verificationRequests` | `create`, `retrieve`, `list`, `cancel`, `waitForCompletion` |
| `proofs` | `validate`, `revoke`, `status`, `listRevoked`, `token`, `listReissued`, `delegationToken`, `verifyOffline`, `refreshJWKS` |
| `webhookDeliveries` | `list`, `retrieve`, `retry` |

---

## Test Mode (Sandbox)

Use `pk_test_` keys to test your integration without sending real messages or consuming quota. The SDK methods are the same, with one addition — `testVerify` for auto-completing verifications — and one difference: a test-mode proof token does not validate (see the note below).

```typescript
// JavaScript — full test flow
const proof = new Proof('pk_test_your_key');

const verification = await proof.verifications.create({
  type: 'phone',
  channel: 'whatsapp',
  identifier: '+37069199199',
});

// Auto-complete without waiting for user action
const result = await proof.verifications.testVerify(verification.id);
console.log(result.proof_token); // Signed test-mode JWT: /proofs/validate answers it valid: false
```

```python
# Python — full test flow
async with Proof("pk_test_your_key") as proof:
    verification = await proof.verifications.create(
        type="phone", channel="telegram", identifier="+37069199199",
    )
    result = await proof.verifications.test_verify(verification["id"])
    print(result["proof_token"])
```

> **Note:** Verification, verification-request and approval webhooks still fire in test mode; verification webhooks carry `"test": true`, and so does a verification-request webhook once a test verification is behind it. Proof tokens are signed JWTs in the production format, but `/proofs/validate` answers them `valid: false` with `reason: "test_environment"`. Once the test tier is in place in an environment, they are signed by its keys, published only at `/.well-known/jwks-test.json`, so offline verification, which reads `/.well-known/jwks.json`, never answers one valid: `unknown_key`, or `status_unavailable` within 30 seconds after an earlier unknown key. A test token issued before that tier was in place carries a production key: the status list marks an active one's slot unavailable from the first rebuild after its issuance (a list built earlier, served or cached for up to about two minutes, still reads it as valid), and a signature-only check, or an offline check of one that carries no status-list slot, cannot tell it from a production token.

---

## Create a Verification

All verification types follow the same pattern: create a challenge, present it to the user, then poll or listen for completion.

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `telegram_bot` (`social` and `wallet` are coming soon) |
| `channel` | string | Yes | Channel for verification |
| `identifier` | string | Yes | The asset to verify |
| `external_user_id` | string | No | Your reference ID for the user |
| `client_metadata` | object | No | Custom metadata |

When a verification completes, `verification.completed` goes to the webhook URL set on your account, unless your [event filter](/docs/api#webhook-event-types) leaves it out; or wait for it with `waitForCompletion`. A [verification request](#verification-requests) sends its own events to its `callback_url` only.

### JavaScript

```typescript
const verification = await proof.verifications.create({
  type: 'email',
  channel: 'email',
  identifier: 'user@example.com',
  external_user_id: 'usr_123',
});
```

### Python

```python
verification = await proof.verifications.create(
    type="email",
    channel="email",
    identifier="user@example.com",
    external_user_id="usr_123",
)
```

### PHP

```php
$verification = $proof->verifications->create([
    'type' => 'email',
    'channel' => 'email',
    'identifier' => 'user@example.com',
    'external_user_id' => 'usr_123',
]);
```

### Go

```go
verification, err := client.Verifications.Create(ctx, map[string]any{
    "type":              "email",
    "channel":           "email",
    "identifier":        "user@example.com",
    "external_user_id":  "usr_123",
})
```

---

## Polling

Every resource with a lifecycle (verifications, sessions, verification requests) provides a `waitForCompletion` method that polls until a terminal status is reached.

### Default Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Interval | 2 seconds | First wait between polls |
| Backoff | ×1.5 | Each wait is 1.5 times the last |
| Max interval | 30 seconds | Longest wait between polls |
| Jitter | up to 0.5 seconds | Random delay added to each wait |
| Timeout | 10 minutes | Maximum wait time |

### Customizing Poll Options

**JavaScript:**

```typescript
const result = await proof.verifications.waitForCompletion('6650f1c2a9e4b3d2c1a0f9e8', {
  interval: 1000,    // 1 second
  timeout: 30000,    // 30 seconds
});
```

**Python:**

```python
result = await proof.verifications.wait_for_completion(
    "6650f1c2a9e4b3d2c1a0f9e8",
    interval=1.0,   # 1 second
    timeout=30.0,    # 30 seconds
)
```

**PHP:**

```php
$result = $proof->verifications->waitForCompletion(
    '6650f1c2a9e4b3d2c1a0f9e8',
    interval: 1.0,   // 1 second
    timeout: 30.0,    // 30 seconds
);
```

**Go:**

```go
result, err := client.Verifications.WaitForCompletion(ctx, "6650f1c2a9e4b3d2c1a0f9e8", &proof.WaitOptions{
    Interval: 1 * time.Second,
    Timeout:  30 * time.Second,
})
```

### Terminal States

| Resource | Terminal States |
|----------|----------------|
| Verifications | `verified`, `failed`, `expired`, `revoked` |
| Sessions | `verified`, `failed`, `expired` |
| Verification Requests | `completed`, `expired`, `cancelled` |

If the timeout is reached before a terminal state, a `PollingTimeoutError` is thrown.

---

## Error Handling

All SDKs throw typed errors that map to HTTP status codes. Every error carries the error code, the message, the HTTP status, and optional details and request id, named in each language's style:

| Language | Code | Message | Status | Details | Request id |
|----------|------|---------|--------|---------|------------|
| JavaScript | `code` | `message` | `statusCode` | `details` | `requestId` |
| Python | `code` | `str(e)` | `status_code` | `details` | `request_id` |
| PHP | `errorCode` | `getMessage()` | `statusCode` | `details` | `requestId` |
| Go | `Code` | `Message` | `StatusCode` | `Details` | `RequestID` |

In PHP the types end in `Exception` (`ValidationException`, `NotFoundException`, …).

| HTTP Status | Error Type | Meaning |
|-------------|-----------|---------|
| 400 | `ValidationError` | Invalid request parameters |
| 401 | `AuthenticationError` | Invalid or missing API key |
| 403 | `ForbiddenError` | Insufficient permissions |
| 404 | `NotFoundError` | Resource not found |
| 409 | `ConflictError` | Resource conflict |
| 429 | `RateLimitError` | Too many requests |
| 500+ | `ServerError` | Server-side error |

### JavaScript

```typescript
import { ValidationError, NotFoundError } from '@proof-holdings/sdk';

try {
  await proof.verifications.retrieve('invalid_id');
} catch (err) {
  if (err instanceof NotFoundError) {
    console.log('Not found:', err.message);
  } else if (err instanceof ValidationError) {
    console.log('Validation:', err.details);
  }
}
```

### Python

```python
from proof_sdk.errors import NotFoundError, ValidationError

try:
    await proof.verifications.retrieve("invalid_id")
except NotFoundError as e:
    print(f"Not found: {e}")
except ValidationError as e:
    print(f"Validation: {e.details}")
```

### PHP

```php
use ProofHoldings\Exceptions\NotFoundException;
use ProofHoldings\Exceptions\ValidationException;

try {
    $proof->verifications->retrieve('invalid_id');
} catch (NotFoundException $e) {
    echo "Not found: " . $e->getMessage();
} catch (ValidationException $e) {
    echo "Validation: " . print_r($e->details, true);
}
```

### Go

```go
import "errors"

_, err := client.Verifications.Retrieve(ctx, "invalid_id")
if err != nil {
    var nfErr *proof.NotFoundError
    var valErr *proof.ValidationError
    if errors.As(err, &nfErr) {
        fmt.Println("Not found:", nfErr.Message)
    } else if errors.As(err, &valErr) {
        fmt.Println("Validation:", valErr.Details)
    }
}
```

---

## Automatic Retries

All SDKs automatically retry on server errors (5xx) and rate limits (429) with exponential backoff.

| Setting | Default | Description |
|---------|---------|-------------|
| Max retries | 2 | Number of retry attempts |
| Initial backoff | 1 second | First retry delay |
| Max backoff | 10 seconds | Maximum retry delay |
| Backoff formula | `min(1s * 2^attempt, 10s)` | Exponential with cap |

Rate limit responses with a `Retry-After` header are respected automatically.

### Configuring Retries

**JavaScript:**

```typescript
const proof = new Proof('pk_live_...', {
  maxRetries: 5,
  timeout: 60000,
});
```

**Python:**

```python
proof = Proof(
    "pk_live_...",
    max_retries=5,
    timeout=60.0,
)
```

**PHP:**

```php
$proof = new Proof(
    'pk_live_...',
    maxRetries: 5,
    timeout: 60.0,
);
```

**Go:**

```go
client, _ := proof.NewClient("pk_live_...",
    proof.WithMaxRetries(5),
    proof.WithTimeout(60 * time.Second),
)
```

---

## Credentials

Sign in once with `npx @proof-holdings/mcp-server login` and construct the client with no key. When the key argument is omitted, every SDK resolves it the same way: `PROOF_API_KEY` when set and non-empty, else the entry for the client's base URL in `~/.proof-holdings/credentials` — the file the login command writes, keyed by base URL so a key saved against staging is never sent to production — else construction fails with a message naming the three roads (pass the key, set `PROOF_API_KEY`, or run the login command). The base URL is the explicit option, else a non-empty `PROOF_BASE_URL`, else `https://api.proof.holdings`. An explicit key is used as given, and an explicit empty key is an error: neither reads the environment or the file.

**JavaScript:**

```typescript
const proof = new Proof();
const verification = await proof.verifications.retrieve('6650f1c2a9e4b3d2c1a0f9e8');
```

**Python:**

```python
proof = Proof()
verification = await proof.verifications.retrieve("6650f1c2a9e4b3d2c1a0f9e8")
```

**PHP:**

```php
$proof = new Proof();
$verification = $proof->verifications->retrieve('6650f1c2a9e4b3d2c1a0f9e8');
```

**Go:**

```go
client, err := proof.NewDefaultClient()
if err != nil {
    log.Fatal(err) // names PROOF_API_KEY and the login command
}
verification, err := client.Verifications.Retrieve(ctx, "6650f1c2a9e4b3d2c1a0f9e8")
```

---

## Proof Validation

After a verification succeeds, you receive a signed JWT proof token. You can validate it via the API or offline.

### Online Validation

```typescript
// JavaScript
const result = await proof.proofs.validate('eyJhbGciOiJFUzI1NiI...');
console.log(result.valid);       // true
console.log(result.verification); // { type, channel, verified_at, ... }
```

```python
# Python
result = await proof.proofs.validate("eyJhbGciOiJFUzI1NiI...")
print(result["valid"])  # True
```

```php
// PHP
$result = $proof->proofs->validate('eyJhbGciOiJFUzI1NiI...');
echo $result['valid'];  // true
```

```go
// Go
result, _ := client.Proofs.Validate(ctx, "eyJhbGciOiJFUzI1NiI...", "")
fmt.Println(result["valid"])  // true
```

### Offline Validation

Proof tokens are ES256-signed JWTs. Every SDK checks one without your API key: `proofs.verifyOffline` reads the public keys and the revocation status list, then checks the signature, expiry and revocation. A token that fails answers `valid: false` with a `reason` (`expired`, `revoked`, `status_unavailable`, ...) instead of throwing.

```typescript
// JavaScript
const result = await proof.proofs.verifyOffline('eyJhbGciOiJFUzI1NiI...');
console.log(result.valid, result.reason);
```

```python
# Python
result = await proof.proofs.verify_offline("eyJhbGciOiJFUzI1NiI...")
print(result["valid"], result.get("reason"))
```

```php
// PHP (needs: composer require "firebase/php-jwt:^7.0")
$result = $proof->proofs->verifyOffline('eyJhbGciOiJFUzI1NiI...');
echo $result['valid'] ? 'valid' : $result['reason'];
```

```go
// Go
result, err := client.Proofs.VerifyOffline(ctx, "eyJhbGciOiJFUzI1NiI...")
if err != nil {
    // the key set could not be read, so the token was not checked
}
fmt.Println(result.Valid, result.Reason)
```

To check the signature alone, pass `{ checkRevocation: false }` (JavaScript), `check_revocation=False` (Python), `checkRevocation: false` (PHP) or `proof.WithoutRevocationCheck()` (Go).

Without an SDK, the same check by hand:

1. Fetch public keys from `https://api.proof.holdings/.well-known/jwks.json` (cache up to 1 hour)
2. Fetch revocation list from `/api/v1/proofs/revoked` — reuse it for up to 5 minutes, then fetch again; offline, its signature stays verifiable for 1 hour
3. Verify JWT signature, check expiry, and check revocation status

See the [API Reference](/docs/api) for detailed offline verification examples.

---

## Key rotation

Proof replaces the keys it signs proofs with every 30 days. A token signed by an earlier key keeps verifying: the key stays published for as long as a token it signed can live. The SDKs' offline verification (`verifyOffline` in JavaScript and PHP, `verify_offline` in Python, `VerifyOffline` in Go) fetches the key set itself, refetches it every 10 minutes, and applies these rules.

**Retiring the legacy keys.** The keys Proof signed with before this rotation scheme — the legacy keys — are retired once. Proof re-signs the live tokens they signed under a current key, and revokes the legacy keys no earlier than 30 days after that re-signing. From then on an old copy that names its key answers `key_revoked`, and one with no `kid` does not verify at all. A token that was already superseded, revoked or expired when they were re-signed, or that changed while it was read, or that the re-signing refused or could not match to its record, stays on the legacy key and stops verifying when they are revoked. Replace every copy you handed out or published with the re-signed token — `key_revoked` below says where to fetch it.

**`iat` and `verified_at`.** `iat` is when the token was signed; `verified_at` is when the control it attests was verified. A token re-signed after a key revocation keeps every claim but `iat` — the same `jti`, `exp` and `verified_at` — under a new `kid`; a proof token from before `jti` existed also gains its row's `jti` and `proof_id`. Judge a proof's age by `verified_at`.

**A generic JOSE or JWT library gives signature-only assurance.** Checking a token's signature against the JWKS skips everything else the SDKs check: a key's purpose, its validity window and lifetime limit, revoked keys, and the status list. Use the SDK's offline verification, or validate online.

**`maxJwksAge`.** When the key set cannot be refreshed, a cached copy keeps answering until it is 24 hours old by default, at most 25 days (`maxJwksAgeMs` in JavaScript, `max_jwks_age` in Python, `maxJwksAge` in PHP, `WithMaxJWKSAge` in Go). Past that the answer is `jwks_stale`, a refusal to guess — refresh the key set once the issuer is reachable.

**`key_revoked`.** The issuer revoked the key that signed the token — after a suspected compromise, or at the retirement of the legacy keys; it re-signs the live tokens that key signed. The result carries `pull_hint` (`PullHint` in Go): the issuer origin and the handle to fetch the re-signed token by — `proofs.token(handle)` for a proof your own account owns, the public `proofs.delegationToken(handle)` for a delegation (`delegation_token` in Python, `Token` / `DelegationToken` in Go). An API key is refused the token of one of the account's own verified assets (403 `owner_session_required`), and `listReissued` does not list those for a key: read it signed in, in the dashboard. A revocation takes effect offline once two key-set fetches at least 5 minutes apart both carry it, and is then kept for a year.

**The current `jti` generation is an online check.** Once a proof is verified again, `proofs.validate()` (`POST /proofs/validate`) answers `superseded` for the earlier token. Offline verification still accepts that token until its own `exp`: it carries the proof's status-list slot, shared with the current token, so revoking or suspending the proof stops both.

**Refresh does not revoke.** A token re-issued within its generation — a renewal — leaves the earlier copy valid until its own `exp`; only a revoked key ends it sooner. To withdraw a token, revoke the proof.

**Minimum versions.** These checks are in version 1.0.0 and later of each SDK: `@proof-holdings/sdk`, `proof-holdings-sdk`, `proof-holdings/sdk` and `github.com/ProofHoldings/sdk-go`; on npm and PyPI the first release is 1.1.0. The pull methods — `token`, `listReissued` and `delegationToken` (`list_reissued` and `delegation_token` in Python, `Token`, `ListReissued` and `DelegationToken` in Go) — are in version 1.4.0 and later of each SDK.

---

## Verification Requests

Verification requests let you bundle multiple asset verifications into a single flow. Define required assets and let the user complete them at their own pace.

```typescript
// JavaScript
const request = await proof.verificationRequests.create({
  assets: [
    { type: 'phone', required: true },
    { type: 'email', required: true },
    { type: 'domain', required: false },
  ],
  reference_id: 'onboarding_user_123',
});

// Poll for completion
const completed = await proof.verificationRequests.waitForCompletion(request.id);
```

```python
# Python
request = await proof.verification_requests.create(
    assets=[
        {"type": "phone", "required": True},
        {"type": "email", "required": True},
        {"type": "domain", "required": False},
    ],
    reference_id="onboarding_user_123",
)

completed = await proof.verification_requests.wait_for_completion(request["id"])
```

---

## Webhooks

Manage the endpoints events are delivered to, monitor delivery status and retry failed deliveries.

An endpoint receives events for every sub-account (`scope: 'all'`) or only the ones you list (`scope: 'selected'`). Its signing secret is returned once — when you create the endpoint and when you rotate the secret — so store it then; reads show only `secret_set`.

```typescript
// Add an endpoint for every sub-account and keep its signing secret
const endpoint = await proof.webhookEndpoints.create({
  url: 'https://example.com/webhooks/proof',
  scope: 'all',
  events: ['verification.completed', 'confirmation.*'],
});
console.log(endpoint.secret);

// Replace the secret later; deliveries queued from now on are signed with the new one
const rotated = await proof.webhookEndpoints.rotateSecret(endpoint.id);

// Rotate the secret a sub-account's callback_url deliveries are signed with
const callback = await proof.profiles.rotateCallbackSecret('6650f1c2a9e4b3d2c1a0f9e1');

// List failed deliveries
const failed = await proof.webhookDeliveries.list({ status: 'failed' });

// Retry a delivery
await proof.webhookDeliveries.retry('6650f1c2a9e4b3d2c1a0f9e9');
```

```python
endpoint = await proof.webhook_endpoints.create(
    url="https://example.com/webhooks/proof",
    scope="all",
    events=["verification.completed", "confirmation.*"],
)
print(endpoint["secret"])

callback = await proof.profiles.rotate_callback_secret("6650f1c2a9e4b3d2c1a0f9e1")
```

The methods are `list`, `create`, `retrieve`, `update`, `delete` and `rotateSecret` on `webhookEndpoints` — `webhook_endpoints` with `rotate_secret` in Python, `WebhookEndpoints` with `Retrieve` and `RotateSecret` in Go, and `$proof->webhookEndpoints` with `get` in PHP.

---

## Source Code

All SDKs are open source:

- [JavaScript/TypeScript SDK](https://github.com/ProofHoldings/sdk-js)
- [Python SDK](https://github.com/ProofHoldings/sdk-python)
- [PHP SDK](https://github.com/ProofHoldings/sdk-php)
- [Go SDK](https://github.com/ProofHoldings/sdk-go)
