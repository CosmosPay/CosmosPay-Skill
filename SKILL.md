---
name: cosmospay
description: Integrate Stellar payments with CosmosPay using @cosmosapp/pay_sdk. Use when building SEP-7 payment intents, browser wallet checkout, transaction validation, signed webhooks, swaps, liquidity-pool flows, fiat on/off-ramps, customer billing, or a custom Cosmos Wallet adapter.
---

# CosmosPay Integration

Use the public `@cosmosapp/pay_sdk` package. Keep the CosmosPay API key on the server and complete wallet approval in the browser.

## Choose the flow

| Need | Read |
| --- | --- |
| Create and settle a Stellar checkout | [references/payments.md](references/payments.md) |
| Connect Cosmos Wallet or another browser wallet | [references/wallets.md](references/wallets.md) |
| Use swaps, liquidity, ramps, KYC, or other managers | [references/operations.md](references/operations.md) |

## Install

```bash
npm install @cosmosapp/pay_sdk
```

Install `@stellar/stellar-sdk` beside it when the browser must build or submit a transaction:

```bash
npm install @cosmosapp/pay_sdk @stellar/stellar-sdk
```

## Apply the security boundary

1. Instantiate `Client` only on the server with `COSMOS_PAY_API_KEY`.
2. Send payment-intent payloads, never the API key, to the browser.
3. Let a wallet display and approve the transaction before signing.
4. Validate the resulting transaction hash on the server.
5. Fulfil an order from a verified webhook or server-side validation result, never from a browser success message alone.
6. Match assets by code and issuer. Use the package's network-specific asset constants.
7. Treat all amounts as decimal strings with no more than seven fractional digits.

## Verify before finishing

- Exercise the flow on testnet with a `dv_` key.
- Confirm that the connected wallet and intent use the same network.
- Confirm the rendered destination, asset, amount, memo, and fee.
- Reject a mismatched transaction during server-side validation.
- Verify webhook signatures and deduplicate stable event IDs.
- Run the consuming application's typecheck and tests.

Use the package documentation under `node_modules/@cosmosapp/pay_sdk/llms/` as the version-matched source of truth when an API shape differs from this skill.
