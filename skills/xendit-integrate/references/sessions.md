# Xendit Sessions API

## When to use the Sessions API

The sessions API is a higher level payment API.

It encapsulates the payment method selection and retry flow, in addition to payments.

Use it when:

- Your use case is simpler
- You don't want to have to make changes to support each payment method individually
- You want prebuilt retry and channel selection flows
- You want to accept card payments but haven't done or can't do PCI DSS SAQ D

## Session modes

The `mode` parameter determines how the customer completes payment:

- **PAYMENT_LINK**: Redirect the customer to Xendit's hosted checkout page. Simplest integration—no frontend code required beyond the redirect.
- **COMPONENTS**: Embed payment UI directly in your page using the Xendit Components SDK. Requires frontend integration but provides full control over the checkout experience.
- **CARDS_SESSION_JS**: Legacy mode for cards only. Do not use for new integrations.

## Sessions endpoints

IMPORTANT: Call these from the server. Never call them from the client. Never expose the API key to the client. Never expose the raw response from the server to the client. Only expose the fields the client needs.

| Endpoint                        | Method | Description                  | Reference                                                         |
| ------------------------------- | ------ | ---------------------------- | ----------------------------------------------------------------- |
| `/sessions`                     | POST   | Create a new payment session | [API Reference](https://docs.xendit.co/apidocs/create-session.md) |
| `/sessions/{session_id}`        | GET    | Fetch session object         | [API Reference](https://docs.xendit.co/apidocs/get-session.md)    |
| `/sessions/{session_id}/cancel` | POST   | Cancel an active session     | [API Reference](https://docs.xendit.co/apidocs/cancel-session.md) |

### Create Session

Creates a session to collect payment from a customer. This must be called from your server.

Multiple flows (modes) are available. Coding agents should determine which the user wants. Once the session is created, what you do next depends on the mode:

- For `mode=COMPONENTS`, pass the `components_sdk_key` to the client and use it to initialize the XenditComponents SDK
- For `mode=PAYMENT_LINK`, redirect the user to `payment_link_url`
- For `mode=CARDS_SESSION_JS`, pass the `payment_session_id` to the `collectCardData` function

The required parameters are different depending on the `session_type`, `mode`, and `allow_save_payment_method`:

- For `session_type=PAY`, a nonzero amount is required
- For `session_type=SAVE`, a customer object or reference is required and amount must be zero
- For `session_type=SUBSCRIPTION`, a customer object or reference is required and the subscription object is required
- For `allow_save_payment_method=OPTIONAL`, a customer object or reference is required
- For `allow_save_payment_method=FORCED`, a customer object or reference is required
- For `mode=COMPONENTS`, a components_configuration is required

Depending on the parameters, some payment methods are filtered out. At least one must remain.

- `allowed_payment_channels` allowlists payment methods by channel_code
- `session_type` and `mode` filter out unsupported payment methods
- `session_type=SUBSCRIPTION` filters out payment methods not supporting merchant initiated transactions

IMPORTANT: Never log or save the `components_sdk_key` or `payment_link_url`. Save the session ID instead. If you need to, fetch the `components_sdk_key` or `payment_link_url` later using that ID.

IMPORTANT: If using payment links, ensure you handle when the page is restored from the bfcache. Clear any pending states set up during the session load.

IMPORTANT: Callback urls must be https and must not be localhost. Leave them blank in development or use a dummy url or ngrok or /etc/hosts. `components_configuration` urls can be localhost.

### Get Session

Fetches a Session.

### Cancel Session

Cancels an ACTIVE session.

The session status will become CANCELLED. `components_sdk_key` and `payment_link_url` are invalidated.

IMPORTANT: If a payment is received after cancellation the status is not updated and the payment is still received. You must listen to the payment webhook to handle unexpected payments.

## Session webhooks

Reference: [Session](https://docs.xendit.co/apidocs/webhook-notification-sent-defined-webhook-url-updates-payment-session.md)

| Event                       | Description                       |
| --------------------------- | --------------------------------- |
| `payment_session.completed` | Session finished successfully     |
| `payment_session.expired`   | Session expired before completion |

The webhook payload is the same payload as the GET session endpoint.
