# Subscriptions API

## When to use the Subscriptions API

The Subscriptions API automates recurring billing.

Use it when:

- You need to charge customers on a regular schedule (daily, weekly, monthly, yearly)
- You want automatic retry handling for failed payments
- You need to manage subscription lifecycles (pause, resume, cancel)

Concepts:

- Plan - Holds one or more Payment Tokens. While active, charges a token on a schedule.
- Cycle - One billing period. Owned by plan.
- Attempt - One attempt to charge the user across all the user's payment methods. Owned by cycle.
- Attempt action - One action taken as a result of an attempt (e.g. charge one token, create payment link). Owned by Attempt.

The next cycle to be billed is a materialized object - a Cycle in the SCHEDULED state. It is updated automatically if you change the subscription. Its amount can be explicitly updated.

All get list endpoints have pagination with `limit`, `before_id`, and `after_id`.

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

IMPORTANT: For new integrations, always use the header `api-version: 2026-01-01`. For integrations using the old version, continue using the old version, or upgrade with care. They are not compatible. Subscriptions must be manipulated with the API version they were created with. You must track the version used if migrating.

IMPORTANT: The subscriptions API does NOT handle creation of a subscription from a new customer payment method. The Session API with `type: "SUBSCRIPTION"` does that. The subscriptions API _does_ handle creation of new subscriptions from existing Payment Tokens (no customer interaction i.e. resubscribe previously subscribed user).

## Subscription Plan endpoints

| Endpoint                           | Method | Description                                       | Reference                                                                         |
| ---------------------------------- | ------ | ------------------------------------------------- | --------------------------------------------------------------------------------- |
| `/recurring/plans`                 | POST   | Create a subscription plan with an existing token | [API Reference](https://docs.xendit.co/apidocs/create-recurring-plan.md)          |
| `/recurring/plans/{id}`            | GET    | Fetch a plan                                      | [API Reference](https://docs.xendit.co/apidocs/get-subscription-plan-2.md)        |
| `/recurring/plans/{id}`            | PATCH  | Update a plan                                     | [API Reference](https://docs.xendit.co/apidocs/update-subscription-plan-2.md)     |
| `/recurring/plans/{id}/deactivate` | POST   | Deactivate (cancel) a plan                        | [API Reference](https://docs.xendit.co/apidocs/deactivate-subscription-plan-2.md) |

### Create Subscription Plan

Creates a recurring billing plan. Requires a customer ID, payment tokens, amount, currency, and schedule configuration. The schedule defines interval (DAY/WEEK/MONTH/YEAR), interval count, and optional retry settings.

Set `immediate_payment: true` to charge immediately on creation. Set `failed_cycle_action: STOP` to deactivate the plan if a cycle fails.

IMPORTANT: Subscriptions created from existing tokens must use tokens capable of merchant initiated transactions. (It will not validate on creation, the next charge will fail).

### Get Subscription Plan

Fetches a Subscription Plan.

### Update Subscription Plan

Updates plan. Automatically updates the next scheduled cycle (if no attempt to charge it has been made yet).

### Deactivate Subscription Plan

Stops the plan. It cannot be reactivated.

## Subscription Cycle endpoints

| Endpoint                                               | Method | Description                   | Reference                                                                             |
| ------------------------------------------------------ | ------ | ----------------------------- | ------------------------------------------------------------------------------------- |
| `/recurring/plans/{id}/cycles`                         | GET    | List cycles for a plan        | [API Reference](https://docs.xendit.co/apidocs/get-list-of-subscription-cycles-2.md)  |
| `/recurring/plans/{plan_id}/cycles/{id}`               | GET    | Fetch a cycle                 | [API Reference](https://docs.xendit.co/apidocs/get-subscription-cycle-2.md)           |
| `/recurring/plans/{plan_id}/cycles/{id}`               | PATCH  | Update a cycle                | [API Reference](https://docs.xendit.co/apidocs/update-subscription-cycle-2.md)        |
| `/recurring/plans/{plan_id}/cycles/{id}/cancel`        | POST   | Cancel a cycle                | [API Reference](https://docs.xendit.co/apidocs/cancel-subscription-cycle-2.md)        |
| `/recurring/plans/{plan_id}/cycles/{id}/force_attempt` | POST   | Force a payment attempt       | [API Reference](https://docs.xendit.co/apidocs/force-attempt-subscription-cycle-2.md) |
| `/recurring/plans/{plan_id}/cycles/{id}/simulate`      | POST   | Simulate cycle payment (test) | [API Reference](https://docs.xendit.co/apidocs/simulate-cycle-payment.md)             |

### List Subscription Cycles

Returns all cycles for a plan.

### Get Subscription Cycle

Fetches a Subscription Cycle. Includes Attempt history.

### Update Subscription Cycle

Updates a scheduled cycle's timestamp, amount, or metadata. Only works for cycles in SCHEDULED status.

### Cancel Subscription Cycle

Cancels a scheduled cycle. The customer will not be charged for this billing period. Only works for cycles in SCHEDULED status.

### Force Attempt Subscription Cycle

Triggers an immediate payment attempt on the next scheduled cycle. This brings forward the payment date of the scheduled cycle. The date of the cycle after that is unaffected.

Forced retries have a separate retry limit from automatic retries.

### Simulate Cycle Payment

Test mode only. Simulates a successful payment for a scheduled cycle.

## Webhooks

Reference: [Subscription](https://docs.xendit.co/apidocs/subscription-webhook.md)

| Event                                  | Description                                    |
| -------------------------------------- | ---------------------------------------------- |
| `recurring.plan.activated`             | Plan successfully activated                    |
| `recurring.plan.inactivated`           | Plan deactivated (by request or failed cycles) |
| `recurring.cycle.created`              | New cycle created for the plan                 |
| `recurring.cycle.retrying`             | Cycle payment failed, retry scheduled          |
| `recurring.cycle.succeeded`            | Cycle payment completed                        |
| `recurring.cycle.failed`               | Cycle payment failed after all retries         |
| `recurring.cycle.force_attempt_failed` | Forced payment attempt failed                  |

### recurring.plan.activated

Fires when a recurring plan is successfully activated and ready to charge. This happens after plan creation when all prerequisites are met (e.g., payment token is active).

Use this to activate the product or features your customer is subscribing to.

### recurring.plan.inactivated

Fires when a plan is deactivated. A deactivated plan cannot be reactivated. Use this event to revoke access or notify customers their subscription has ended.

### recurring.cycle.created

Fires when a new billing cycle is created for a plan. The cycle is in `SCHEDULED` status with a `scheduled_timestamp` indicating when payment will be attempted.

### recurring.cycle.retrying

Fires when a cycle payment fails and a retry is scheduled. The `attempt_details` array shows the failure reason and `next_retry_timestamp` indicates when the next attempt will occur.

### recurring.cycle.succeeded

Fires when a cycle payment completes successfully. The `attempt_details` includes the successful `payment_id`.

### recurring.cycle.failed

Fires when a cycle payment fails after all retry attempts are exhausted. If `failed_cycle_action: STOP`, the plan becomes inactive. If `failed_cycle_action: RESUME`, the plan continues to the next cycle. The unpaid amount does not carry over.

### recurring.cycle.force_attempt_failed

Fires when a forced payment attempt (triggered via Force Attempt endpoint) fails.
