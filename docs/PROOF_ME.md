# Proof of Me

Proof of Me is anti-impersonation **reverse verification**. Instead of proving your own identity, you let trusted contacts confirm that a person reaching out to them is really you.

An account holder enrolls named **members** (trusted contacts — colleagues, family, clients) on a **Circle**. When someone suspicious contacts a member claiming to be you, the member taps a button and you get a CONFIRM request on a **different channel** than the impersonator used. You approve or deny; the member is told the result. Drills exercise the flow on a schedule so members stay trained.

> Proof of Me is built on `Circle`, the people primitive. It was originally attached to a HITL config and enrolled "challengers"; that surface was removed and the member-management endpoints now live under `/api/v1/circles`.

## How it works

1. **Enroll** — add a member to a Circle, then mint a single-use invite. The member taps the Telegram/WhatsApp deep link, binding their messenger identity to your Circle.
2. **Challenge** — when a member is contacted by a suspicious party, they trigger an identity challenge. You receive a cross-channel CONFIRM and approve/deny.
3. **Drill** — automated daily drills and monthly reinforcement messages keep members sharp. You can also fire an on-demand drill.

The Circle endpoints are dual-auth (JWT dashboard session or API key). An API key needs `circles:create` to create a Circle, `circles:read` to read one, and `circles:write` for every member mutation. The identity-challenge endpoints are also dual-auth and use `hitl:write` (create) and `hitl:read` (poll).

## Endpoints

| Operation | Method | Path |
| --- | --- | --- |
| Add member | `POST` | `/api/v1/circles/:id/members` |
| List members | `GET` | `/api/v1/circles/:id/members` |
| Invite member | `POST` | `/api/v1/circles/:id/members/invite` |
| Trigger drill | `POST` | `/api/v1/circles/:id/members/:memberId/drill` |
| Remove member | `DELETE` | `/api/v1/circles/:id/members/:memberId` |
| Create identity challenge | `POST` | `/api/v1/identity-challenges` |
| List identity challenges | `GET` | `/api/v1/identity-challenges` |
| Get identity challenge | `GET` | `/api/v1/identity-challenges/:id` |

An identity challenge reports one status on every read: `pending`, `confirmed`, `denied`, `cancelled` (whoever started the check withdrew it), `failed` (it could not be delivered), `expired` (nobody answered in time), `revoked` or `suspended` (the confirmation's proof was revoked or suspended by `POST /api/v1/proofs/{proof_id}/revoke` or `/suspend` — revoking a denial's proof leaves it `denied` — or, for `revoked` only, the member being checked was removed from the Circle while the check was still pending, in which case there is no `proof_id`). Removing a member is not guaranteed to revoke a confirmation they already gave; revoke that one by its `proof_id`, which the response carries. Identity challenges are not listed by `/api/v1/authorizations`, which serves consents only.

A contact who sent `/opt_out` receives no drills, reminders or checks they did not start themselves; naming them in `POST /api/v1/identity-challenges` or a drill answers `409 member_opted_out`. Set a Circle's `owner_name` to how its members know you — the bot names you by it, and by nobody while it is empty.

A member can also carry more than one messenger. The per-member channel endpoints are `POST`/`GET` `/api/v1/circles/:id/members/:memberId/channels` and `DELETE /api/v1/circles/:id/members/:memberId/channels/:channelId`.

## SDK usage

Member management lives on the `circles` / `Circles` resource in every official SDK.

### JavaScript / TypeScript

```ts
import { Proof } from '@proof-holdings/sdk';

const proof = new Proof('pk_live_...');

// Enroll a member and invite them
const member = await proof.circles.addMember(circleId, { name: 'Alex' });
const invite = await proof.circles.inviteMember(circleId, member.id);
console.log(invite.telegram_deep_link); // send this to Alex

// Keep them trained
await proof.circles.triggerDrill(circleId, member.id);
```

### Python

```python
from proof_sdk import Proof

async with Proof("pk_live_...") as proof:
    member = await proof.circles.add_member(circle_id, name="Alex")
    invite = await proof.circles.invite_member(circle_id, member["id"])

    await proof.circles.trigger_drill(circle_id, member["id"])
```

### Go

```go
client, _ := proof.NewClient("pk_live_...")

member, _ := client.Circles.AddMember(ctx, circleID, map[string]any{
    "name": "Alex",
})
invite, _ := client.Circles.InviteMember(ctx, circleID, member["id"].(string))

_, _ = client.Circles.TriggerDrill(ctx, circleID, member["id"].(string))
```

### PHP

```php
use ProofHoldings\Proof;

$proof = new Proof('pk_live_...');

$member = $proof->circles->addMember($circleId, ['name' => 'Alex']);
$invite = $proof->circles->inviteMember($circleId, $member['id']);

$proof->circles->triggerDrill($circleId, $member['id']);
```

### Raising an identity challenge

The challenge itself is one POST naming the Circle and the member, then a poll until it reaches a terminal state:

```bash
curl -X POST https://api.proof.holdings/api/v1/identity-challenges \
  -H "Authorization: Bearer pk_live_YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "circle_id": "CIRCLE_ID",
    "member_id": "MEMBER_ID"
  }'
```

```bash
curl https://api.proof.holdings/api/v1/identity-challenges/CHALLENGE_ID \
  -H "Authorization: Bearer pk_live_YOUR_API_KEY"
```

`suspicious_channel` is an optional third field naming the channel the impersonator used, so the CONFIRM is routed somewhere else.

## MCP tools

The MCP server exposes the same surface for AI agents:

- `add_circle_member`
- `list_circle_members`
- `invite_circle_member`
- `create_circle_member_drill`
- `remove_circle_member`
- `create_identity_challenge`
- `get_identity_challenge`

After `create_identity_challenge`, poll `get_identity_challenge` until it leaves `pending` (`confirmed`, `denied`, `cancelled`, `failed`, `expired` or `revoked`).
