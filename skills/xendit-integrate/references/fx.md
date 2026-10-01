# FX API

## When to use the FX API

The FX API handles currency conversion.

Use it when:

- Checking exchange rates before making a conversion or payout

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

IMPORTANT: Use the header `api-version: 2025-06-06` for all FX API requests.

IMPORTANT: Quotes expire quickly. Create the quote immediately before using it. Quotes can only be used if the source amount matches. Quotes can only be used once.

IMPORTANT: Amounts are in minor units. Most currencies have 2 decimal places (PHP 10.00 = 1000), but some have none (IDR 10 = 10).

## Quote endpoints

Quotes provide exchange rates for a currency pair. Use quotes for cross-currency payouts.

| Endpoint       | Method | Description    | Reference                                                          |
| -------------- | ------ | -------------- | ------------------------------------------------------------------ |
| `/quotes`      | POST   | Create a quote | [API Reference](https://docs.xendit.co/apidocs/create-quote.md)    |
| `/quotes/{id}` | GET    | Fetch a quote  | [API Reference](https://docs.xendit.co/apidocs/get-quote-by-id.md) |

### Create Quote

Creates an FX quote. Specify source and destination currencies, and either source or destination amount (not both). Returns the exchange rate and calculated amount for the other side.

Set `type: PAYOUT` for cross-currency payouts. Set `type: CONVERSION` for balance conversions.

### Get Quote

Fetches a Quote.

## Conversion endpoints

Conversions move funds between currency balances in your Xendit account.

| Endpoint            | Method | Description         | Reference                                                               |
| ------------------- | ------ | ------------------- | ----------------------------------------------------------------------- |
| `/conversions`      | POST   | Create a conversion | [API Reference](https://docs.xendit.co/apidocs/create-conversion-1.md)  |
| `/conversions/{id}` | GET    | Fetch a conversion  | [API Reference](https://docs.xendit.co/apidocs/get-conversion-by-id.md) |

### Create Conversion

Executes a currency conversion using a quote. Deducts from your source currency balance and credits your destination currency balance.

### Get Conversion

Fetches a Conversion.
