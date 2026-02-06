---
title: SDKs
lastUpdated: "2026-02-06"
status: current
---

# Official SDKs

Install an official SDK to start verifying assets in minutes. All SDKs provide the same features: typed clients, automatic retries with exponential backoff, polling helpers, and structured error handling.

| Language | Package | Install |
|----------|---------|---------|
| JavaScript/TypeScript | `@proof-holdings/sdk` | `npm install @proof-holdings/sdk` |
| Python | `proof-sdk` | `pip install proof-sdk` |
| PHP | `proof-holdings/sdk` | `composer require proof-holdings/sdk` |
| Go | `github.com/ProofHoldings/sdk-go` | `go get github.com/ProofHoldings/sdk-go` |

---

## Quick Start

### JavaScript / TypeScript

```typescript
import { ProofHoldings } from '@proof-holdings/sdk';

const proof = new ProofHoldings('pk_live_your_key');

// Create a phone verification
const verification = await proof.verifications.create({
  type: 'phone',
  channel: 'whatsapp',
  identifier: '+37069199199',
});

console.log(verification.challenge.deep_link);
// "https://wa.me/...?text=X7K2M9"

// Poll until verified (or timeout)
const result = await proof.verifications.waitForCompletion(verification.id);
console.log(result.proof.token);
// "eyJhbGciOiJSUzI1NiI..."
```

### Python

```python
import asyncio
from proof_sdk import ProofHoldings

async def main():
    async with ProofHoldings("pk_live_your_key") as proof:
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
use ProofHoldings\ProofHoldings;

$proof = new ProofHoldings('pk_live_your_key');

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

All SDKs expose the same five resources that map to the REST API:

| Resource | Methods |
|----------|---------|
| `verifications` | `create`, `retrieve`, `list`, `verify`, `submit`, `waitForCompletion` |
| `sessions` | `create`, `retrieve`, `waitForCompletion` |
| `verificationRequests` | `create`, `retrieve`, `list`, `cancel`, `waitForCompletion` |
| `proofs` | `validate`, `revoke`, `getRevocationList`, `checkRevocation` |
| `webhookDeliveries` | `list`, `retrieve`, `retry` |

---

## Create a Verification

All verification types follow the same pattern: create a challenge, present it to the user, then poll or listen for completion.

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | Yes | `phone`, `email`, `domain`, `social`, `wallet` |
| `channel` | string | Yes | Channel for verification |
| `identifier` | string | Yes | The asset to verify |
| `external_user_id` | string | No | Your reference ID for the user |
| `callback_url` | string | No | Webhook URL for status updates |
| `client_metadata` | object | No | Custom metadata |

### JavaScript

```typescript
const verification = await proof.verifications.create({
  type: 'email',
  channel: 'email',
  identifier: 'user@example.com',
  external_user_id: 'usr_123',
  callback_url: 'https://yourapp.com/webhook',
});
```

### Python

```python
verification = await proof.verifications.create(
    type="email",
    channel="email",
    identifier="user@example.com",
    external_user_id="usr_123",
    callback_url="https://yourapp.com/webhook",
)
```

### PHP

```php
$verification = $proof->verifications->create([
    'type' => 'email',
    'channel' => 'email',
    'identifier' => 'user@example.com',
    'external_user_id' => 'usr_123',
    'callback_url' => 'https://yourapp.com/webhook',
]);
```

### Go

```go
verification, err := client.Verifications.Create(ctx, map[string]any{
    "type":              "email",
    "channel":           "email",
    "identifier":        "user@example.com",
    "external_user_id":  "usr_123",
    "callback_url":      "https://yourapp.com/webhook",
})
```

---

## Polling

Every resource with a lifecycle (verifications, sessions, verification requests) provides a `waitForCompletion` method that polls until a terminal status is reached.

### Default Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Interval | 3 seconds | Time between polls |
| Timeout | 10 minutes | Maximum wait time |

### Customizing Poll Options

**JavaScript:**

```typescript
const result = await proof.verifications.waitForCompletion('ver_123', {
  interval: 1000,    // 1 second
  timeout: 30000,    // 30 seconds
});
```

**Python:**

```python
result = await proof.verifications.wait_for_completion(
    "ver_123",
    interval=1.0,   # 1 second
    timeout=30.0,    # 30 seconds
)
```

**PHP:**

```php
$result = $proof->verifications->waitForCompletion(
    'ver_123',
    interval: 1.0,   // 1 second
    timeout: 30.0,    // 30 seconds
);
```

**Go:**

```go
result, err := client.Verifications.WaitForCompletion(ctx, "ver_123", &proof.WaitOptions{
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

All SDKs throw typed errors that map to HTTP status codes. Every error includes `code`, `message`, `statusCode`, and optional `details` and `requestId`.

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
    print(f"Not found: {e.message}")
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

result, err := client.Verifications.Retrieve(ctx, "invalid_id")
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
const proof = new ProofHoldings('pk_live_...', {
  maxRetries: 5,
  timeout: 60000,
});
```

**Python:**

```python
proof = ProofHoldings(
    "pk_live_...",
    max_retries=5,
    timeout=60.0,
)
```

**PHP:**

```php
$proof = new ProofHoldings(
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

## Proof Validation

After a verification succeeds, you receive a signed JWT proof token. You can validate it via the API or offline.

### Online Validation

```typescript
// JavaScript
const result = await proof.proofs.validate('eyJhbGciOiJSUzI1NiI...');
console.log(result.valid);       // true
console.log(result.verification); // { type, channel, verified_at, ... }
```

```python
# Python
result = await proof.proofs.validate("eyJhbGciOiJSUzI1NiI...")
print(result["valid"])  # True
```

```php
// PHP
$result = $proof->proofs->validate('eyJhbGciOiJSUzI1NiI...');
echo $result['valid'];  // true
```

```go
// Go
result, _ := client.Proofs.Validate(ctx, "eyJhbGciOiJSUzI1NiI...")
fmt.Println(result["valid"])  // true
```

### Offline Validation

Proof tokens are RS256-signed JWTs. For offline validation without API calls:

1. Fetch public keys from `https://api.proof.holdings/.well-known/jwks.json` (cache 24h)
2. Fetch revocation list from `/api/v1/proofs/revoked` (cache 5min)
3. Verify JWT signature, check expiry, and check revocation status

See the [API Reference](/docs/api) for detailed offline verification examples.

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

Monitor webhook delivery status and retry failed deliveries.

```typescript
// List failed deliveries
const failed = await proof.webhookDeliveries.list({ status: 'failed' });

// Retry a delivery
await proof.webhookDeliveries.retry('del_123');
```

---

## Source Code

All SDKs are open source:

- [JavaScript/TypeScript SDK](https://github.com/ProofHoldings/sdk-js)
- [Python SDK](https://github.com/ProofHoldings/sdk-python)
- [PHP SDK](https://github.com/ProofHoldings/sdk-php)
- [Go SDK](https://github.com/ProofHoldings/sdk-go)
