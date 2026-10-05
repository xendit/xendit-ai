---
name: xendit-integrate
description: Guide Xendit payment API integrations. Use when the user wants to integrate with Xendit, accept payments, send payouts, manage subscriptions, handle disputes programmatically, manage Xendit customers, transactions, and balances, or manage XenPlatform subaccounts.
license: proprietary
metadata:
  author: xendit
  version: "1.0"
---

## Test Mode

Always develop using Xendit's test mode. Test mode is a wholly separate environment from live mode that cannot affect any real money. Resources in test mode cannot reference resources in live mode and vice versa. Connections to third parties (i.e. payment methods) are replaced with mocks.

## Auth

Auth is done with the `Authorization` header. If you don't have an API key, ask the user to create one on the dashboard.

Development API keys start with `xnd_development_`. Production API keys start with `xnd_production_`. Never use a production API key.

The API key is used as the username of basic auth i.e. `Authorization: Basic base64(api_key + ":")` where password is empty.

## Webhooks

Webhooks are configured on the Xendit dashboard and are fired in response to certain events.

Webhooks are authenticated with the `x-callback-token` header, which must be configured in the dashboard.

They all have the structure `{ event: string, data: unknown }` where the type of `data` depends on the event type.

Respond with a 200 in a timely manner, unless you want the webhook to be resent. It will be resent up to 6 times with increasing delays.

Events can arrive out of order.

In rare cases, such as partner downtime, automated reconciliation, or incidents, events can arrive significantly late.

IMPORTANT: Always validate the `x-callback-token` header to ensure the webhook is only called by Xendit.

IMPORTANT: You must handle out of order or duplicate events. Ensure your processing logic is atomic and idempotent or store events and discard duplicates. Identify duplicates using the event name and the ID of the respective entity.

## Conventions

The Xendit API follows standard REST conventions. It uses JSON request and response bodies.

Errors are always of the form `{ "error_code": string, "message": string }`. The `message` field is always a technical error and should never be shown to customers. Prefer showing a generic error (you can show a specific error message if you recognize the `error_code`). If using components, use the provided end-user-appropriate error strings. Always log payment flow errors, even if they are client errors.

Coding agents should not be concerned with rate limiting unless the user requests it. Rate limits are generally 60/min in test mode and 600/min in live mode. Use the headers `rate-limit-limit` (max), `rate-limit-remaining` (current), `rate-limit-reset` (timer) if you need to read the rate limit status.

## Terminology

"Payment Method" refers to the kind of payment being made (Cards, QRIS, SGQR, etc). Our APIs sometimes call this "Payment Channel". These are interchangeable terms. "Payment Method" is preferred.

"Payment Request" and "Payment Token" refer to merchant requests to charge or tokenize a payment method. These are often called "Payment Intent" and "Setup Intent" in other PSPs.

"Channel Properties" refers to the set of payment-method-specific inputs (usually user inputs but sometimes merchant config) passed to the Payment API, Payment Sessions API, and Payouts API.

"Payment" refers to a payment or authorization that has already occurred.

"Session" or "Payment Session" is an entity that abstracts the payment flow, handling retries, payment method selection, and UI.

"V3" is sometimes used to refer to the Payments API. "Payments API" is preferred.

## Discover what API is being used

Always ensure you know which Xendit integration the user intends to use. The user likely knows which APIs they intend to use. If they don't you should guide them.

- Accept payments
  - Low level API, do everything yourself -> Payments API
  - High level API, already built payment method selection, retry flow, UI -> Payment Sessions API
  - Manage payment disputes -> Disputes API
- Subscriptions
  - Create subscription for new user -> Payment Sessions API
  - Create subscription from existing token -> Subscriptions API
  - Manipulate existing subscription -> Subscriptions API
- Send payments -> Payouts API, FX API
- Manage subaccounts or act on behalf of a subaccount -> XenPlatform API
- Read your own account data or subaccount data -> Balance, Transactions APIs

## Routing by API type

IMPORTANT: Referenced files are local, relative to this file. Not remote files.

| API Type         | Local Reference Doc             | Description                                                                                                                                                                          |
| ---------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Payments         | `./references/payments.md`      | Low level inbound payment API. One time payment, tokenization, pay with token, capture, refund.                                                                                      |
| Payment Sessions | `./references/sessions.md`      | High level inbound payment API. Entry point API for payment links, components, and subscriptions. Creates resources, lower level operations on created resources require other APIs. |
| Components       | `./references/components.md`    | Frontend API for embedded payments on your website, iOS, or Android app. Requires as input an SDK key created by the Sessions API.                                                   |
| Subscriptions    | `./references/subscriptions.md` | For manipulating subscriptions. New subscriptions are created with the sessions API (unless using an existing customer and token).                                                   |
| Customers        | `./references/customers.md`     | For manipulating customer records.                                                                                                                                                   |
| Payouts          | `./references/payouts.md`       | Outbound payments API.                                                                                                                                                               |
| FX               | `./references/fx.md`            | Currency conversion quotes (for payouts) and conversions (for merchant balance).                                                                                                     |
| Disputes         | `./references/disputes.md`      | Fetch and manipulate disputes filed by customers. Submit evidence (text or image) or accept disputes.                                                                                |
| XenPlatform      | `./references/xenplatform.md`   | For creating and manipulating subaccounts, transfers, and split payments.                                                                                                            |
| Transactions     | `./references/transactions.md`  | Fetch account transaction history.                                                                                                                                                   |
| Balance          | `./references/balance.md`       | Fetch account balances.                                                                                                                                                              |

## Subaccounts

Subaccounts are directly manipulated with the XenPlatform APIs but most APIs can be called as a subaccount.

If you have the master account, use a master account API key with the `for-user-id` header to change your identity to your subaccount. e.g. the 'fetch balance' endpoint fetches the master account balance but with `for-user-id` it can fetch a subaccount balance.

## Remote API Docs (docs.xendit.co/\*.md)

Always include the the .md extension or it will return garbage.

API reference docs contain openapi specs. They are large, avoid reading the whole file at once if possible.

## Legacy APIs

APIs not documented here are legacy. Do not use for new integrations.
