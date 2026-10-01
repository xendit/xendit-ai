# Balance API

## When to use the Balance API

The Balance API shows how much money is in your Xendit account.

Use it to:

- Check available funds before making payouts
- Display balance to your subaccount holders

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

## Balance endpoints

| Endpoint   | Method | Description   | Reference                                                      |
| ---------- | ------ | ------------- | -------------------------------------------------------------- |
| `/balance` | GET    | Fetch balance | [API Reference](https://docs.xendit.co/apidocs/get-balance.md) |

### Get Balance

Returns your current balance. Filter by `account_type` (CASH or HOLDING) and `currency`. Use `at_timestamp` to query historical balance at a specific point in time.
