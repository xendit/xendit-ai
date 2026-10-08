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

IMPORTANT: In live mode, `for-user-id` calls are rejected with 404 `XEN_PLATFORM_SUB_ACCOUNT_NOT_LIVE` until the subaccount is `LIVE` (a few endpoints are configured to allow it). Test mode subaccounts are `LIVE` immediately, so code that works in test can still fail in live. Wait for the `account.verification` webhook with `PASSED` before transacting.

## Account endpoints

| Endpoint            | Method | Description         | Reference                                                            |
| ------------------- | ------ | ------------------- | -------------------------------------------------------------------- |
| `/v3/accounts`      | POST   | Create a subaccount | [API Reference](https://docs.xendit.co/apidocs/create-account-v3.md) |
| `/v2/accounts`      | GET    | List subaccounts    | [API Reference](https://docs.xendit.co/apidocs/list-accounts.md)     |
| `/v2/accounts/{id}` | GET    | Fetch a subaccount  | [API Reference](https://docs.xendit.co/apidocs/get-account.md)       |

### Create Account

Creates a subaccount linked to your master account. Requires `name`, `email`, and `identity` (country + entity type). Set `configuration.users.send_email_invite: true` to send a dashboard invitation.

Subaccounts go through KYC verification. Test mode subaccounts are created with status LIVE immediately. They exist only in test mode and never go live.

Constraints:

- `identity.country_of_incorporation` must match the master account's country of operation, unless cross-country subaccounts are enabled for the master (ask the account manager).
- Available `identity.entity_type` values depend on the country.
- With a test API key, `entity_type` must be `CORPORATION`, the country must match the master account's country, and `configuration` must be omitted or use the defaults (`send_email_invite: false`, `webhooks.recipient: MASTER_ACCOUNT`).
- `configuration.webhooks.recipient` is `MASTER_ACCOUNT` (default, events go to the master account's webhook URLs) or `SUB_ACCOUNT` (events go to the subaccount's own webhook URLs, which must be configured). The v2 values `MASTER` and `SUB` are no longer used.

Payment methods are activated per subaccount for its own country of operation. A subaccount does not inherit the master account's payment methods, even when cross-country subaccounts are enabled.

### List Accounts

Returns paginated subaccounts. Filter by `email`, `status`, `type` (MANAGED or OWNED), and date ranges.

`type` describes how the subaccount is run ([help article](https://help.xendit.co/hc/en-us/articles/6787784288665-What-is-the-difference-between-managed-and-owned-sub-accounts)):

- `OWNED`: fully controlled by the master account and invisible to the partner by default. It has no dashboard users unless the master invites them after creation (they get access except developer settings). The partner submits no onboarding information, and the platform's details are shown to end customers. Disabled by default for Indonesia-based accounts.
- `MANAGED`: the partner gets a full Xendit account with dashboard access from creation, submits their own onboarding information, and their details are shown to end customers. The master can still transact on its behalf with `for-user-id`.

The type cannot be changed after creation. The v2 list and fetch responses use a different shape from the v3 create response (no `identity` / `configuration` objects).

### Get Account

Fetches a subaccount by ID.

## Split Rule endpoints

| Endpoint       | Method | Description         | Reference                                                            |
| -------------- | ------ | ------------------- | -------------------------------------------------------------------- |
| `/split_rules` | POST   | Create a split rule | [API Reference](https://docs.xendit.co/apidocs/create-split-rule.md) |

### Create Split Rule

Defines how payments are split between accounts. Each route specifies a `destination_account_id` and either `flat_amount` or `percent_amount`. Include the returned `split_rule_id` in payment requests via the `with-split-rule` header.

## Transfer endpoints

| Endpoint                           | Method | Description           | Reference                                                                    |
| ---------------------------------- | ------ | --------------------- | ---------------------------------------------------------------------------- |
| `/transfers`                       | POST   | Create a transfer     | [API Reference](https://docs.xendit.co/apidocs/create-transfers.md)          |
| `/transfers/reference={reference}` | GET    | Fetch transfer by ref | [API Reference](https://docs.xendit.co/apidocs/get-transfer-by-reference.md) |

### Create Transfer

Moves funds between master and subaccounts, or between subaccounts. Requires unique `reference`, `amount`, `source_user_id`, and `destination_user_id`. In live mode, both accounts must be `LIVE` or the transfer fails with `XEN_PLATFORM_SUB_ACCOUNT_NOT_LIVE`.

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

- `VERIFICATION_IN_PROGRESS`: KYC is being processed
- `PASSED`: KYC completed successfully, subaccount can transact
- `AWAITING_RESUBMISSION`: Documents need correction (check `kyc.failure_reasons`)
- `FAILED`: KYC failed

Use this event to track onboarding progress. For `AWAITING_RESUBMISSION`, prompt the merchant to correct and re-upload the flagged documents. For `PASSED`, enable full functionality for the merchant.

### split.payment

Fires when a split payment is completed or fails. The `status` field is either `COMPLETED` (funds routed successfully) or `FAILED` (split failed, check `failure_code`).
