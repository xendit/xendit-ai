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

## Account endpoints

| Endpoint            | Method | Description         | Reference                                                            |
| ------------------- | ------ | ------------------- | -------------------------------------------------------------------- |
| `/v3/accounts`      | POST   | Create a subaccount | [API Reference](https://docs.xendit.co/apidocs/create-account-v3.md) |
| `/v2/accounts`      | GET    | List subaccounts    | [API Reference](https://docs.xendit.co/apidocs/list-accounts.md)     |
| `/v2/accounts/{id}` | GET    | Fetch a subaccount  | [API Reference](https://docs.xendit.co/apidocs/get-account.md)       |

### Create Account

Creates a subaccount linked to your master account. Requires `name`, `email`, and `identity` (country + entity type). Set `configuration.users.send_email_invite: true` to send a dashboard invitation.

Subaccounts go through KYC verification. Test mode subaccounts are created with status LIVE immediately.

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
