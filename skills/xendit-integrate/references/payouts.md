# Payouts API

## When to use the Payouts API

The Payouts API sends money to external bank accounts and wallets.

Use it to:

- Allow your platform customers to withdraw funds
- Disburse escrowed funds
- Process refunds outside the payment system (e.g. refund to a different payment method)
- Process payroll

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

IMPORTANT: Use the header `api-version: 2025-09-01` for all Payouts API requests.

IMPORTANT: Always use an idempotency key. Always ensure your idempotency key is properly persisted and reused when retrying the same payout. Use a new idempotency key when retrying a payout that was rejected for a good reason.

IMPORTANT: Amounts are in minor units. Most currencies have 2 decimal places (PHP 10.00 = 1000), but some have none (IDR 10 = 10).

## Payout endpoints

| Endpoint           | Method | Description     | Reference                                                              |
| ------------------ | ------ | --------------- | ---------------------------------------------------------------------- |
| `/v3/payouts`      | POST   | Create a payout | [API Reference](https://docs.xendit.co/apidocs/create-payout-v3.md)    |
| `/v3/payouts/{id}` | GET    | Fetch a payout  | [API Reference](https://docs.xendit.co/apidocs/get-payout-v3-by-id.md) |

### Create Payout

Creates a payout to a bank account or e-wallet. Creating a payout initiates the transaction.

The payout is processed asynchronously; use webhooks to track status.

### Get Payout

Fetches a Payout.

## Webhooks

Reference: [Payout](https://docs.xendit.co/apidocs/payout-v3-webhook.md)

| Event                          | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| `v3_payout.succeeded`          | Funds credited to beneficiary                                |
| `v3_payout.failed`             | Payout rejected or processing failed                         |
| `v3_payout.reversed`           | Previously succeeded payout was bounced back; funds refunded |
| `v3_payout.rejected`           | Payout rejected for compliance reasons; funds refunded       |
| `v3_payout.pending_compliance` | Payout requires compliance review; expect follow-up requests |

### v3_payout.succeeded

Fires when the payout succeeds and the funds have been transferred.

### v3_payout.failed

Fires when the payout fails.

IMPORTANT: Only retry the following error codes: INSUFFICIENT_BALANCE, REJECTED_BY_CHANNEL, TEMPORARY_TRANSFER_ERROR.

### v3_payout.reversed

Fires when a previously succeeded payout receives a bounceback or reversal from the partner bank. The funds have been automatically refunded to your available balance.

This can happen days after the original payout succeeded.

### v3_payout.rejected

Fires when a payout is rejected due to compliance assessment. Funds have been refunded to your available balance. This may occur for sanctions list matches or other compliance concerns.

Use this event to flag the transaction for review.

### v3_payout.pending_compliance

Fires when the payout requires further compliance assessment before it can proceed. The payout is on hold, not failed.
