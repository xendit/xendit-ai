# Customers API

## When to use the Customers API

The Customers API stores customer information for use with payments.

Use it when:

- Saving payment tokens, tokens must be attached to a customer

If using payment links, some limited customer information is reflected to the customer. If using components, the same information is available through `getCustomer`.

Customers can be either INDIVIDUAL or BUSINESS. The type cannot be changed after creation.

## Important Rules

IMPORTANT: All these endpoints are called from the server. Never call them from the client. Never expose the API key to the client.

IMPORTANT: Use the header `api-version: 2020-10-31` for all Customers API requests.

## Customer endpoints

| Endpoint          | Method | Description       | Reference                                                                  |
| ----------------- | ------ | ----------------- | -------------------------------------------------------------------------- |
| `/customers`      | GET    | List customers    | [API Reference](https://docs.xendit.co/apidocs/get-customers-list.md)      |
| `/customers`      | POST   | Create a customer | [API Reference](https://docs.xendit.co/apidocs/create-customer-request.md) |
| `/customers/{id}` | GET    | Fetch a customer  | [API Reference](https://docs.xendit.co/apidocs/get-customer-id.md)         |
| `/customers/{id}` | PATCH  | Update a customer | [API Reference](https://docs.xendit.co/apidocs/update-customer.md)         |

### List Customers

Returns customers matching the provided `reference_id` query parameter. Required parameter.

### Create Customer

Creates a customer record. Set `type` to INDIVIDUAL or BUSINESS. Provide corresponding details in `individual_detail` or `business_detail`.

### Get Customer

Fetches a Customer by Xendit ID (`cust-xxx`).

### Update Customer

Updates customer fields. Only fields in the request are updated. Array fields (addresses, kyc_documents) are replaced entirely, not merged. Pass null to clear a field.
