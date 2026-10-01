# Xendit AI

An agent skill that guides AI-assisted Xendit payment API integrations. Use this skill when working with Xendit to accept payments, send payouts, manage subscriptions, handle disputes, and more.

## Installation

Install using Vercel's skills CLI (npm package `skills`) or similar tool.

```bash
npx skills add xendit/xendit-ai
```

## What's Included

This skill provides API reference and helpful tips for:

| API              | Description                                                                      |
| ---------------- | -------------------------------------------------------------------------------- |
| Payments         | Low-level inbound payment API (one-time payments, tokenization, capture, refund) |
| Payment Sessions | High-level payment API for payment links, components, and subscriptions          |
| Components       | Frontend API for embedded payments (web, iOS, Android)                           |
| Subscriptions    | Manage recurring payment subscriptions                                           |
| Customers        | Manage customer records                                                          |
| Payouts          | Outbound payments API                                                            |
| FX               | Currency conversion quotes and conversions                                       |
| Disputes         | Fetch and respond to customer disputes                                           |
| XenPlatform      | Subaccount management, transfers, and split payments                             |
| Transactions     | Account transaction history                                                      |
| Balance          | Account balances                                                                 |

## Usage

Once installed, the skill activates automatically when you ask your coding agent to help with Xendit integrations. The AI will guide you to the appropriate API based on your use case and provide accurate integration code.

## License

Proprietary - Xendit
