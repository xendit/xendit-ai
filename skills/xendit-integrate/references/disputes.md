# Disputes API

## When to use the Disputes API

The Disputes API handles chargebacks and payment disputes.

Use it to:

- Be informed of and respond to customer disputes
- Submit evidence to contest disputes

You only need to do this if you want your own internal UI or AI agent to interact with disputes.
In small volumes, it's usually preferable to handle disputes manually using the Xendit dashboard UI.

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

IMPORTANT: Disputes have deadlines. Webhooks are how you learn disputes exist. You will lose disputes if you miss the webhook. You must implement the webhooks.

IMPORTANT: Evidence submission and challenge are two separate steps. You must upload evidence THEN call the challenge endpoint. You will lose disputes that are not challenged.

## Dispute lifecycle

1. **ACTION_REQUIRED** - Dispute raised, you must respond before `due_date`
2. **UNDER_REVIEW** - Challenge submitted, awaiting decision
3. **WON** - Ruled in your favor, funds returned
4. **LOST** - Ruled against you, funds debited

## Dispute endpoints

| Endpoint                      | Method | Description       | Reference                                                                            |
| ----------------------------- | ------ | ----------------- | ------------------------------------------------------------------------------------ |
| `/v1/disputes`                | GET    | List disputes     | [API Reference](https://docs.xendit.co/apidocs/list-all-disputes-1.md)               |
| `/v1/disputes/{id}`           | GET    | Get dispute       | [API Reference](https://docs.xendit.co/apidocs/get-dispute-details-1.md)             |
| `/v1/disputes/{id}/accept`    | POST   | Accept dispute    | [API Reference](https://docs.xendit.co/apidocs/accept-a-dispute-1.md)                |
| `/v1/disputes/{id}/challenge` | POST   | Challenge dispute | [API Reference](https://docs.xendit.co/apidocs/finalize-and-submit-evidence-1.md)    |
| `/v1/disputes/simulate`       | POST   | Simulate (test)   | [API Reference](https://docs.xendit.co/apidocs/simulate-a-dispute-sandbox-only-1.md) |

### List Disputes

Returns paginated disputes. Filter by `status=ACTION_REQUIRED` to find disputes needing response. Filter by `due_date_before` to find disputes expiring soon.

### Get Dispute

Fetches a Dispute. Includes `category.allowed_evidence` which lists the evidence types you can submit. Use `include_timeline=true` for audit trail.

### Accept Dispute

Concedes the dispute. Irreversible. The disputed amount is debited and status becomes LOST.

### Challenge Dispute

Finalizes and submits all uploaded evidence. Status moves to UNDER_REVIEW. Evidence can no longer be modified after this.

### Simulate Dispute

Test mode only. Creates test disputes or resolves them as WON/LOST. Use `CREATE_DISPUTE` to create, then `DISPUTE_WON` or `DISPUTE_LOST` to resolve.

## Evidence endpoints

| Endpoint                                    | Method | Description     | Reference                                                                                 |
| ------------------------------------------- | ------ | --------------- | ----------------------------------------------------------------------------------------- |
| `/v1/disputes/{id}/evidences`               | POST   | Upload evidence | [API Reference](https://docs.xendit.co/apidocs/submit-evidence-via-multipart-upload-1.md) |
| `/v1/disputes/{id}/evidences/{evidence_id}` | PATCH  | Update text     | [API Reference](https://docs.xendit.co/apidocs/update-text-evidence-1.md)                 |
| `/v1/disputes/{id}/evidences/{evidence_id}` | DELETE | Delete evidence | [API Reference](https://docs.xendit.co/apidocs/delete-a-submitted-evidence-1.md)          |

### Upload Evidence

Uploads multiple evidence items at once via a multipart/form-data request.

Read `category.allowed_evidence` to see what you're allowed to upload. Each item in `allowed_evidence` corresponds to one allowed field in the multipart upload. All fields are optional.

- `name` is the field name in the multipart upload.
- `type` tells you if you can upload text (just a string) or a file (png, jpg, pdf).
- `max_length` tells you the max characters for string, or max bytes for files.

Upload zero or one multipart fields for each item in `allowed_evidence`, each having the field name set to the corresponding `name`.

For file uploads, attach the file as binary data. For text fields, attach the text as a plain string.

### Update Text Evidence

Updates content of TEXT evidence (using evidence ID, not `allowed_evidence.name`).

You cannot replace files in-place, they must be deleted first.

### Delete Evidence

Removes evidence. Only while dispute is ACTION_REQUIRED.

## Webhooks

Reference: [Dispute](https://docs.xendit.co/apidocs/dispute-webhook-notifications.md)

| Event                          | Description                            |
| ------------------------------ | -------------------------------------- |
| `dispute.action_required`      | New dispute raised, response needed    |
| `dispute.under_review`         | Challenge submitted, awaiting decision |
| `dispute.won`                  | Ruled in your favor                    |
| `dispute.lost`                 | Ruled against you                      |
| `dispute.due_date_approaching` | Deadline is near, respond soon         |
