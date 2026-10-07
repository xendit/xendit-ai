# XenPlatform API

## When to use the XenPlatform API

XenPlatform lets you create and manage subaccounts, transfer funds between them, and split payments.

Use it when:

- You're building a marketplace or platform with multiple merchants
- You need to hold funds on behalf of sellers
- You want to automatically split payments between yourself and merchants

## Important Rules

IMPORTANT: All these endpoints are called from the server using your master account API key. Never call them from the client.

IMPORTANT: Most Xendit APIs support the `for-user-id` header to act on behalf of a subaccount. Use the subaccount's `id` from the create/list response.

## Account Types: MANAGED vs OWNED

When creating a subaccount, you choose a `type`:

| Type | Description |
| ------- | ----------- |
| `MANAGED` | You (the platform) manage KYC and verification on behalf of the subaccount. The subaccount does not log into the Xendit dashboard. Webhooks are not automatically inherited from the master — configure them explicitly per subaccount. |
| `OWNED` | The subaccount owns their Xendit dashboard access. KYC is completed by the merchant directly. Webhooks are automatically routed to the master account by default. |

Choose `MANAGED` for fully embedded marketplace flows where merchants never interact with Xendit directly. Choose `OWNED` when merchants have their own Xendit dashboard access.

> **Webhook note:** For `MANAGED` subaccounts, webhook URLs must be configured manually per subaccount after creation — there is no automatic inheritance from the master. For `OWNED` subaccounts, webhooks are routed to the master account by default.

## Account endpoints

| Endpoint            | Method | Description         | Reference                                                            |
| ------------------- | ------ | ------------------- | -------------------------------------------------------------------- |
| `/v3/accounts`      | POST   | Create a subaccount | [API Reference](https://docs.xendit.co/apidocs/create-account-v3.md) |
| `/v2/accounts`      | GET    | List subaccounts    | [API Reference](https://docs.xendit.co/apidocs/list-accounts.md)     |
| `/v2/accounts/{id}` | GET    | Fetch a subaccount  | [API Reference](https://docs.xendit.co/apidocs/get-account.md)       |

> **API version note:** Account creation uses `/v3/accounts` (current). List and fetch use `/v2/accounts` — the v3 equivalents are not yet publicly available.

### Create Account

Creates a subaccount linked to your master account. Requires `name`, `email`, and `identity` (country + entity type). Set `configuration.users.send_email_invite: true` to send a dashboard invitation.

Subaccounts go through KYC verification. Test mode subaccounts are created with status `LIVE` immediately — no KYC is required in test mode.

**Account status flow:**

| Status | Meaning |
| ------ | ------- |
| `PENDING_VERIFICATION` | Account created, KYC submitted, under review |
| `LIVE` | KYC passed, account can transact |
| `AWAITING_RESUBMISSION` | Documents need correction |
| `FAILED` | KYC failed |
| `SUSPENDED` | Account suspended (fraud or compliance) |

### List Accounts

Returns paginated subaccounts. Filter by `email`, `status`, `type` (MANAGED or OWNED), and date ranges.

### Get Account

Fetches a subaccount by ID.

## Split Rule endpoints

| Endpoint       | Method | Description         | Reference                                                            |
| -------------- | ------ | ------------------- | -------------------------------------------------------------------- |
| `/split_rules` | POST   | Create a split rule | [API Reference](https://docs.xendit.co/apidocs/create-split-rule.md) |

### Create Split Rule

Defines how payments are split between accounts. Each route specifies a `destination_account_id` and either `flat_amount` or `percent_amount`. Include the returned `split_rule_id` in payment requests via the `with-split-rule` header.

**Constraints:**
- Transfers can only be made between accounts within the same master account hierarchy — not to external Xendit accounts.
- In test mode, split rules work but no real funds are moved.

## Transfer endpoints

| Endpoint                           | Method | Description           | Reference                                                                    |
| ---------------------------------- | ------ | --------------------- | ---------------------------------------------------------------------------- |
| `/transfers`                       | POST   | Create a transfer     | [API Reference](https://docs.xendit.co/apidocs/create-transfers.md)          |
| `/transfers/reference={reference}` | GET    | Fetch transfer by ref | [API Reference](https://docs.xendit.co/apidocs/get-transfer-by-reference.md) |

### Create Transfer

Moves funds between master and subaccounts, or between subaccounts. Requires unique `reference`, `amount`, `source_user_id`, and `destination_user_id`.

### Get Transfer by Reference

Fetches a transfer by your reference ID to check its status.

## Webhooks

Reference: [Account Suspension](https://docs.xendit.co/apidocs/account-suspension-webhook-notification.md), [Account Verification](https://docs.xendit.co/apidocs/account-verification-webhook-notification.md), [Split Payment](https://docs.xendit.co/apidocs/split-payment-status-notification-webhook.md)

| Event                  | Description                            |
| ---------------------- | -------------------------------------- |
| `account.suspected`    | Subaccount flagged for suspected fraud |
| `account.suspended`    | Subaccount suspended for fraud         |
| `account.cleared`      | Subaccount cleared after review        |
| `account.verification` | Subaccount KYC status changed          |
| `split.payment`        | Split payment completed or failed      |

### account.suspected

Fires when Xendit suspects a subaccount may be involved in fraudulent activity. An investigation is ongoing. The subaccount status is `SUSPECTED`.

A `account.suspended` or `account.cleared` event will follow after Xendit's review.

### account.suspended

Fires when a subaccount is suspended due to confirmed or high-confidence fraud detection. The subaccount status is `SUSPENDED`. The `reason` field provides context.

Use this event to immediately disable the account in your platform.

### account.cleared

Fires when a subaccount is cleared after investigation.

### account.verification

Fires when the KYC verification status of a subaccount changes. The `kyc.status` field indicates the new status:

- `PENDING_VERIFICATION`: KYC submitted and under review
- `VERIFICATION_IN_PROGRESS`: KYC is being actively processed
- `PASSED`: KYC completed successfully, subaccount can transact
- `AWAITING_RESUBMISSION`: Documents need correction (check `kyc.failure_reasons`)
- `FAILED`: KYC failed

Use this event to track onboarding progress. For `AWAITING_RESUBMISSION`, prompt the merchant to correct and re-upload the flagged documents. For `PASSED`, enable full functionality for the merchant.

> **Webhook delivery note:** When a subaccount is configured to route webhooks to the master account's URL, the HTTP delivery goes to the master URL but the webhook log entry remains on the **subaccount's** dashboard — not the master's. This is expected behavior.

### split.payment

Fires when a split payment is completed or fails. The `status` field is either `COMPLETED` (funds routed successfully) or `FAILED` (split failed, check `failure_code`).

## Common Errors

| Error code | Cause | Fix |
| ---------- | ----- | --- |
| `INVALID_OR_NOT_FOUND_ACCOUNT_ID` | Subaccount ID doesn't exist or belongs to a different master | Verify the subaccount `id` from the create/list response |
| `ACCOUNT_ALREADY_EXISTS` | Email already registered under this master | Use a unique email per subaccount |
| `FORBIDDEN_OPERATION` | Action not permitted for this account type (e.g. deleting a MANAGED account via an OWNED-only path) | Check `type` and use the appropriate flow |
| `INSUFFICIENT_BALANCE` | Not enough funds in source account for transfer | Check balance before initiating transfer |
