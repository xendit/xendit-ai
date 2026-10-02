# Payments API

## When to use the Payments API

The Payments API provides low level control over payment flows.

Use it when:

- You need granular control over payment method selection and configuration
- You want to collect user input yourself and handle actions (3DS, QR) yourself

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

IMPORTANT: Use the header `api-version: 2024-11-11` for all Payments API requests.

IMPORTANT: When handling actions, handle ONE action only. Read the `type` AND `descriptor` fields and handle the first action that has a `type` AND `descriptor` that you support. Ignore unknown actions. You MUST support ignoring unexpected actions.

IMPORTANT: Amounts are _not_ in minor units.

IMPORTANT: Creating a new payment request, replacing an existing one, can result in double payment. Cancelling helps, but not all payment methods support cancellation so you must handle unexpected payments.

## Payment Request endpoints

| Endpoint                                      | Method | Description                       | Reference                                                                            |
| --------------------------------------------- | ------ | --------------------------------- | ------------------------------------------------------------------------------------ |
| `/v3/payment_requests`                        | POST   | Create a payment request          | [API Reference](https://docs.xendit.co/apidocs/create-payment-request.md)            |
| `/v3/payment_requests/{id}`                   | GET    | Fetch payment request             | [API Reference](https://docs.xendit.co/apidocs/get-payment-request.md)               |
| `/v3/payment_requests/{id}`                   | PATCH  | Update payment request            | [API Reference](https://docs.xendit.co/apidocs/update-a-payment-request.md)          |
| `/v3/payment_requests/{id}/cancel`            | POST   | Cancel payment request            | [API Reference](https://docs.xendit.co/apidocs/cancel-payment-request.md)            |
| `/v3/payment_requests/{id}/simulate`          | POST   | Simulate payment (test mode)      | [API Reference](https://docs.xendit.co/apidocs/simulate-payment-test-mode.md)        |
| `/v3/payment_requests/{id}/installment_plans` | GET    | Check available installment plans | [API Reference](https://docs.xendit.co/apidocs/check-available-installment-plans.md) |
| `/refunds`                                    | POST   | Refund a payment                  | [API Reference](https://docs.xendit.co/apidocs/refund-payment-request.md)            |

### Create Payment Request

Initiates a payment. Specify the payment method type and method-specific properties. Returns actions the customer must complete (e.g. redirect URLs, QR codes, bank transfer instructions).

### Get Payment Request

Fetches a Payment Request.

### Update Payment Request

Updates a payment request.

Currently only updates the description and amount for VA channels.

### Cancel Payment Request

Cancels an active payment request.

### Simulate Payment

Test mode only. Simulates certain customer actions (QR, VA, OTC payments) and makes the transaction successful.

### Payment Options

Retrieves additional data about a payment channel.

Currently this returns only one thing: for the CARDS channel, returns the list of available installment plans based on the card BIN and amount.

### Refund Payment Request

Issues a full or partial refund for a completed payment. Support varies by payment method.

## Payment Token endpoints

| Endpoint                         | Method | Description            | Reference                                                               |
| -------------------------------- | ------ | ---------------------- | ----------------------------------------------------------------------- |
| `/v3/payment_tokens`             | POST   | Create a payment token | [API Reference](https://docs.xendit.co/apidocs/create-payment-token.md) |
| `/v3/payment_tokens/{id}`        | GET    | Fetch payment token    | [API Reference](https://docs.xendit.co/apidocs/get-payment-token.md)    |
| `/v3/payment_tokens/{id}/cancel` | POST   | Cancel payment token   | [API Reference](https://docs.xendit.co/apidocs/cancel-payment-token.md) |

### Create Payment Token

Tokenizes a payment method for future use. For cards, this stores the card details securely. The token can then be used in subsequent payment requests without re-entering card details.

### Get Payment Token

Fetches a Payment Token.

### Cancel Payment Token

Deactivates a payment token (if already active) or cancels a tokenization flow (if not active yet).

Deactivated tokens can no longer be used in payment requests.

## Payment endpoints

Payment objects represent payments that have already occurred or have been authorized.

Payment objects are not created directly. Payment objects are created by Xendit automatically when an underlying payment (or authorization) occurs.

| Endpoint                    | Method | Description                | Reference                                                          |
| --------------------------- | ------ | -------------------------- | ------------------------------------------------------------------ |
| `/v3/payments/{id}`         | GET    | Fetch payment              | [API Reference](https://docs.xendit.co/apidocs/get-payment.md)     |
| `/v3/payments/{id}/cancel`  | POST   | Cancel payment             | [API Reference](https://docs.xendit.co/apidocs/cancel-payment.md)  |
| `/v3/payments/{id}/capture` | POST   | Capture authorized payment | [API Reference](https://docs.xendit.co/apidocs/capture-payment.md) |

### Get Payment

Fetches a Payment.

### Cancel Payment

For payments with `capture_method: MANUAL`, performs an authorization reversal.

### Capture Payment

For payments with `capture_method: MANUAL`, captures the payment.

## Webhooks

Reference: [Payment](https://docs.xendit.co/apidocs/payment-webhook-notification.md), [Payment Token](https://docs.xendit.co/apidocs/payment-token-webhook-notification.md), [Refund](https://docs.xendit.co/apidocs/refund-webhook-notification.md)

IMPORTANT: There aren't events for the complete payment request lifecycle, by design. Use `payment.*` events.

| Event                      | Description                         |
| -------------------------- | ----------------------------------- |
| `payment_request.expiry`   | Payment request expired             |
| `payment.capture`          | Payment captured (funds collected)  |
| `payment.authorization`    | Payment authorized (manual capture) |
| `payment.expiry`           | Payment authorization expired       |
| `payment.failure`          | Payment failed                      |
| `payment_token.activation` | Token is ready for use              |
| `payment_token.failure`    | Token creation failed               |
| `payment_token.expiry`     | Token has expired                   |
| `refund.succeeded`         | Refund completed successfully       |
| `refund.failed`            | Refund failed                       |

### payment_request.expiry

Fires when a payment request expires without receiving payment. An expired payment request will no longer accept new payments.

IMPORTANT: Since some payment methods can't be cancelled, it's possible to recieve payment after expiry. You must listent to the `payment.*` webhooks.

### payment.capture

Fires when a Payment object transitions to `SUCCEEDED` status and funds are collected.

Use this event to fulfil orders.

### payment.authorization

Fires when a Payment object transitions to `AUTHORIZED` status. This only occurs for payments with `capture_method: MANUAL`. The funds are reserved on the customer's account but not yet collected.

Use this event to fulfil orders if you use a manual capture flow.

You must call the Capture Payment endpoint to collect the funds before the authorization expires.

### payment.expiry

Fires when a successful payment authorization expires before being captured. This only occurs for payments with `capture_method: MANUAL` where you did not call the Capture Payment endpoint in time.

You can no longer capture the funds.

### payment.failure

Fires when a Payment object transitions to `FAILED` status. The `failure_code` field indicates the reason (e.g., `INSUFFICIENT_BALANCE`, `CARD_DECLINED`, `AUTHENTICATION_FAILED`).

Use this event to fail orders or initiate a retry.

### payment_token.activation

Fires when a payment token can be used.

The payload includes `token_details` with account information when available (account name, balance).

### payment_token.failure

Fires when token activation fails.

Use this to fail your save payment method flow.

### payment_token.expiry

Fires when a previously active token expires and can no longer be used for payments. Tokens can expire due to reaching their natural expiry period.

Use this to remove the link between your customer and this saved token.

### refund.succeeded

Fires when a refund is successfully processed.

### refund.failed

Fires when a refund fails. Check the `failure_code` field for the reason.

Use this event to flag the failed refund for manual review.
