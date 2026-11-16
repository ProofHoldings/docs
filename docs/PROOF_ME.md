# Proof of Me

Proof of Me protects the people who know you from someone pretending to be you. When someone contacts your people pretending to be you, they check with you on a channel they already know is yours.

You add the people who should be able to check — colleagues, family, clients — as **members** of a **Circle**. When someone claiming to be you reaches a member, the member taps a button and you get a request to confirm on a **different channel** than the one that person used. You approve or deny; the member is told the result. Practice checks (drills) run on a schedule so members know what a real one looks like.

## How it works

1. **Set up** — create a Circle (`POST /api/v1/circles`, `circles.create` in every SDK) and verify your own phone. The request to confirm goes to that phone; without a verified one, a check answers `400 no_channel`.
2. **Enroll** — add a member to the Circle, then mint a single-use invite. The member taps the Telegram/WhatsApp deep link, which links their messenger account to your Circle.
3. **Check** — when someone suspicious contacts a member, the member starts a check. You get a request to confirm on another channel and approve or deny it.
4. **Drill** — about once a month, at a random point 25 to 35 days after the last one, a member is picked to run a real check against you; roughly one drill in four you are asked not to answer, so the member sees what an unanswered check looks like. Every 30 days members also get a reminder message. You can also fire an on-demand drill. Drills are part of the paid plans: on Pay as you go live drills do not run, and asking for one answers `422 plan_upgrade_required`. Test-mode drills run on every plan; on Pay as you go they reach members on Telegram only, so a member with no verified Telegram channel answers `400 no_channel`.

The Circle endpoints are dual-auth (JWT dashboard session or API key). An API key needs `circles:create` to create a Circle, `circles:read` to read one, and `circles:write` for every member mutation. The identity-challenge endpoints are also dual-auth and use `hitl:write` (create) and `hitl:read` (poll).

## Endpoints

| Operation | Method | Path |
| --- | --- | --- |
| Create Circle | `POST` | `/api/v1/circles` |
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

Circles and members live on the `circles` / `Circles` resource in every official SDK; the check itself is `proofMe.createIdentityChallenge`, then `getIdentityChallenge` until it leaves `pending`.

### JavaScript / TypeScript

```ts
import { Proof } from '@proof-holdings/sdk';

const proof = new Proof('pk_live_...');

// Create a Circle, enroll a member and invite them
const circle = await proof.circles.create({ name: 'Family' });
const member = await proof.circles.addMember(circle.id, { name: 'Alex' });
const invite = await proof.circles.inviteMember(circle.id, member.id);
console.log(invite.telegram_deep_link); // send this to Alex

// Keep them trained
await proof.circles.triggerDrill(circle.id, member.id);

// A check started for that member
const check = await proof.proofMe.createIdentityChallenge({
  circle_id: circle.id,
  member_id: member.id,
});
const latest = await proof.proofMe.getIdentityChallenge(check.id);
console.log(latest.status); // pending, then confirmed / denied / expired ...
```

### Python

```python
from proof_sdk import Proof

async with Proof("pk_live_...") as proof:
    circle = await proof.circles.create(name="Family")
    member = await proof.circles.add_member(circle["id"], name="Alex")
    invite = await proof.circles.invite_member(circle["id"], member["id"])

    await proof.circles.trigger_drill(circle["id"], member["id"])

    check = await proof.proof_me.create_identity_challenge(
        circle_id=circle["id"],
        member_id=member["id"],
    )
    latest = await proof.proof_me.get_identity_challenge(check["id"])
    print(latest["status"])
```

### Go

```go
client, _ := proof.NewClient("pk_live_...")

circle, _ := client.Circles.Create(ctx, map[string]any{"name": "Family"})
circleID := circle["id"].(string)

member, _ := client.Circles.AddMember(ctx, circleID, map[string]any{
    "name": "Alex",
})
invite, _ := client.Circles.InviteMember(ctx, circleID, member["id"].(string))
fmt.Println(invite["telegram_deep_link"]) // send this to Alex

_, _ = client.Circles.TriggerDrill(ctx, circleID, member["id"].(string))

check, _ := client.ProofMe.CreateIdentityChallenge(ctx, map[string]any{
    "circle_id": circleID,
    "member_id": member["id"],
})
latest, _ := client.ProofMe.GetIdentityChallenge(ctx, check["id"].(string))
fmt.Println(latest["status"])
```

### PHP

```php
use ProofHoldings\Proof;

$proof = new Proof('pk_live_...');

$circle = $proof->circles->create(['name' => 'Family']);
$member = $proof->circles->addMember($circle['id'], ['name' => 'Alex']);
$invite = $proof->circles->inviteMember($circle['id'], $member['id']);

$proof->circles->triggerDrill($circle['id'], $member['id']);

$check = $proof->proofMe->createIdentityChallenge([
    'circle_id' => $circle['id'],
    'member_id' => $member['id'],
]);
$latest = $proof->proofMe->getIdentityChallenge($check->id);
echo $latest->status;
```

`suspicious_channel` is an optional third field naming the channel the impersonator used, so the request to confirm is sent somewhere else.

## MCP tools

The MCP server exposes the same surface for AI agents:

- `add_circle_member`
- `list_circle_members`
- `invite_circle_member`
- `create_circle_member_drill`
- `remove_circle_member`
- `create_identity_challenge`
- `get_identity_challenge`

After `create_identity_challenge`, poll `get_identity_challenge` until it leaves `pending` (`confirmed`, `denied`, `cancelled`, `failed`, `expired`, `revoked` or `suspended`).
