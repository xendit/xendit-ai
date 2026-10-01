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
| `/v3/payment_requests/{id}/refunds`           | POST   | Refund a payment                  | [API Reference](https://docs.xendit.co/apidocs/refund-payment-request.md)            |

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

Deactivates a payment token. It can no longer be used for payments.

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

| Event                       | Description                                   |
| --------------------------- | --------------------------------------------- |
| `payment_request.completed` | Payment succeeded                             |
| `payment_request.failed`    | Payment failed                                |
| `payment_request.pending`   | Payment is pending (awaiting customer action) |
| `payment_request.cancelled` | Payment request was cancelled                 |
| `payment_token.activated`   | Token is ready for use                        |
| `payment_token.failed`      | Token creation failed                         |
| `payment_token.expired`     | Token has expired                             |
| `refund.succeeded`          | Refund completed successfully                 |
| `refund.failed`             | Refund failed                                 |
