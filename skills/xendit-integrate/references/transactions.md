# Transactions API

## When to use the Transactions API

The Transactions API provides a unified view of all money movements in your account.

Use it to:

- Build transaction history views
- Reconcile payments and payouts
- Track fees and settlements

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

IMPORTANT: Use the `product_id` field to link transactions back to their source (payment request, payout, etc.).

## Transaction endpoints

| Endpoint             | Method | Description         | Reference                                                            |
| -------------------- | ------ | ------------------- | -------------------------------------------------------------------- |
| `/transactions`      | GET    | List transactions   | [API Reference](https://docs.xendit.co/apidocs/list-transactions.md) |
| `/transactions/{id}` | GET    | Fetch a transaction | [API Reference](https://docs.xendit.co/apidocs/get-transaction.md)   |

### List Transactions

Returns paginated transactions. Filter by `types` (PAYMENT, DISBURSEMENT, REFUND, etc.), `statuses` (PENDING, SUCCESS, FAILED), `channel_categories`, `currency`, date ranges, and amount. Search by `reference_id` or `product_id`.

### Get Transaction

Fetches a single transaction by ID. Includes fee breakdown and settlement status.
