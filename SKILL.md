---
name: cosmospay
description: Integrate Cosmos Pay, a stablecoin payments API with hosted checkout, payment links, webhooks, swaps and a non-custodial wallet, on Stellar, Solana and Monad. Use when a JavaScript/TypeScript or REST integration must accept crypto or stablecoin (USDC) payments, create payment intents or payment links, embed a pay button or checkout, verify signed webhooks, swap assets on one chain or across chains, send USDT0 from Stellar to EVM networks, create private (amount-hidden) payments, or run fiat on/off-ramps. Covers the @cosmosapp/pay_sdk SDK (2.3) and the Payments REST API.
---

# Cosmos Pay Integration

[Cosmos Pay](https://cosmospay.lat) is a multi-chain payments gateway for Stellar, Solana and Monad. You create a payment intent on your server, the payer completes it from their own wallet, and you confirm it on-chain. Around that core it offers a hosted checkout and payment links, signed webhooks, same-chain and cross-chain swaps, USDT0 transfers to EVM networks, private payments, fiat on/off-ramps, products, customers, analytics and a non-custodial Cosmos Pay Wallet.

## Sources of truth

- Docs: https://dev.cosmospay.lat/docs
- Docs index for agents: https://dev.cosmospay.lat/docs/llms.txt (only the SDK: `/docs/llms-sdk.txt`; only the REST API: `/docs/llms-api.txt`)
- OpenAPI spec: https://dev.cosmospay.lat/docs/openapi.json
- Any docs page as Markdown: add `.md` to its path, for example https://dev.cosmospay.lat/docs/sdk/server/quickstart.md
- SDK on npm: [`@cosmosapp/pay_sdk`](https://www.npmjs.com/package/@cosmosapp/pay_sdk); version-matched guides ship inside it at `node_modules/@cosmosapp/pay_sdk/llms/llms-full.txt`
- SDK source and runnable examples: https://github.com/CosmosPay/CosmosJS_SDK

This skill matches `@cosmosapp/pay_sdk` 2.3. When it and the live docs or OpenAPI spec disagree, the spec wins.

## What is supported per chain

| Capability | Stellar | Solana | Monad |
| --- | --- | --- | --- |
| Payment links (`pay` intents: payer link + QR) | SDK and REST (SEP-7) | SDK and REST with `chain: 'solana'` (Solana Pay) | SDK and REST with `chain: 'monad'` (EIP-681) |
| `tx` intents (unsigned XDR for a known payer) | SDK and REST | | |
| Browser wallet signing (`WebClient`) | Freighter, xBull, Rabet, LOBSTR, Albedo, Cosmos Pay Wallet | payer pays the link from any Solana wallet | payer pays the link from any Monad wallet |
| Same-chain swaps (`client.swaps`) | Stellar DEX path payment | Jupiter (mainnet only) | Kuru Flow (mainnet only) |
| Cross-chain swaps (`client.crossChainSwaps`) | NEAR Intents between all three chains (mainnet only) | | |
| USDT0 to EVM networks (`client.usdt0`) | LayerZero OFT from Stellar (mainnet only) | | |
| Private payments (`client.privatePayments`) | amount hidden on-chain (Stellar testnet only for now) | | |

`client.paymentIntents.createPay` takes `chain` (`'stellar'` by default, `'solana'` or `'monad'`); `createTx` is Stellar only. Webhooks, validation and the hosted checkout work the same on every chain.

## Choose the flow

| Need | Read |
| --- | --- |
| Create and settle a checkout or payment link on any chain | [references/payments.md](references/payments.md) |
| Connect Cosmos Pay Wallet or another Stellar browser wallet | [references/wallets.md](references/wallets.md) |
| Swaps, USDT0, private payments, liquidity, ramps, KYC, or other managers | [references/operations.md](references/operations.md) |

## Install and authenticate

```bash
npm install @cosmosapp/pay_sdk
# only when the browser must build or submit a Stellar transaction:
npm install @stellar/stellar-sdk
```

Node.js 18 or newer. The package ships ESM and CommonJS with types. Mint an API key at https://dev.cosmospay.lat (docs: `/docs/api-keys`) and keep it on the server.

```ts
import { Client } from '@cosmosapp/pay_sdk';

const client = new Client({ apiKey: process.env.COSMOS_PAY_API_KEY! });
```

The key prefix selects the network. Do not configure it anywhere else:

| Key prefix | Network |
| --- | --- |
| `dv_...` | testnet (Stellar testnet, Solana devnet, Monad testnet) |
| `prod_...` | mainnet |

The REST API lives under `https://api.cosmospay.lat/cosmos-api/v1` and takes `Authorization: Bearer <apiKey>`.

## The payment flow

1. Your server creates an intent: `client.paymentIntents.createPay({ chain, destination, amount, asset, msg })` or `POST /v1/payment-intents/pay` with `chain`. The response carries `uri` and `qr`.
2. The payer pays from their wallet. On Stellar in the browser, `await new WebClient().pay(intent)` from `@cosmosapp/pay_sdk/web` returns `txHash`. Every intent also has a hosted checkout at `https://payments.cosmospay.lat/<intent id>`, or you can embed it with `<cosmos-pay-button link="<id>">` or `<cosmos-pay-checkout link="<id>">` after loading `https://payments.cosmospay.lat/embed/cosmos-pay.js` (no API key in the page).
3. Your server confirms it: `client.paymentIntents.validate(id, { txHash })` (or `POST /v1/payment-intents/{id}/validate`), or reacts to a signed webhook.

```ts
import { Assets, Client } from '@cosmosapp/pay_sdk';

const client = new Client({ apiKey: process.env.COSMOS_PAY_API_KEY! });
const intent = await client.paymentIntents.createPay({
  destination: 'G...MERCHANT',
  amount: '10',
  asset: Assets.USDC, // use TestnetAssets.USDC with a dv_ key
  msg: 'Order #24',
});
intent.uri; // web+stellar:pay?destination=...
intent.qr;  // data:image/png;base64,...

// Solana (Monad: chain 'monad', a 0x... destination and an ERC-20 contract)
const sol = await client.paymentIntents.createPay({
  chain: 'solana',
  destination: '<merchant Solana address>',
  amount: '10',            // required on Monad
  assetCode: 'USDC',
  assetIssuer: '<USDC SPL mint for your key network>',
});
sol.uri; // solana:...
```

## Webhooks

```ts
import express from 'express';
import { Client, Webhooks, WebhookEventType } from '@cosmosapp/pay_sdk';

const client = new Client({ apiKey: process.env.COSMOS_PAY_API_KEY! });

// Once: register the endpoint. endpoint.secret (whsec_...) is returned only here, so store it.
await client.webhooks.create({
  url: 'https://example.com/hooks/cosmos',
  eventTypes: [WebhookEventType.PaymentIntentSucceeded],
});

const app = express();
app.post('/hooks/cosmos', express.raw({ type: '*/*' }), (req, res) => { // MUST be the raw body
  // Verifies X-Cosmos-Signature (t=<unixSeconds>,v1=<hmacSha256>); throws WebhookSignatureError.
  const event = Webhooks.constructEvent(
    req.body,
    req.get('x-cosmos-signature') ?? '',
    process.env.COSMOS_WEBHOOK_SECRET!,
  );
  if (event.type === WebhookEventType.PaymentIntentSucceeded) {
    // event.id is stable: use it for idempotency
  }
  res.sendStatus(200);
});
```

`WebhookEventType` lists all 30 event types: payment intents (`PAYMENT_INTENT_SUCCEEDED`, `PAYMENT_INTENT_FAILED`, ...), private payments (`ZK_PAYMENT_SUCCEEDED`), same-chain swaps, cross-chain swaps, liquidity and fiat ramps (`PAYIN_COMPLETED`, `PAYOUT_COMPLETED`, ...). Omit `eventTypes` to receive all events. `SWAP_EXPIRED` and `LIQUIDITY_EXPIRED` are not final: a late transaction can still produce `SWAP_SUCCEEDED` or `LIQUIDITY_SUCCEEDED`. Without Express: `createServer(client.webhooks.createHandler()).listen(4242)`.

## Beyond payments

The same `client` exposes `swaps`, `crossChainSwaps`, `usdt0`, `privatePayments`, `liquidity`, `onramp`, `offramp`, `kyc`, `products`, `customers`, `analytics`, `assets`, `aliases` and more. See [references/operations.md](references/operations.md), and read the docs page before writing code against a ramp, swap or transfer.

## Apply the security boundary

1. Instantiate `Client` only on the server with `COSMOS_PAY_API_KEY`. Never ship a `prod_` key to the browser or a web page.
2. Send payment-intent payloads, never the API key, to the browser.
3. Let a wallet display and approve the transaction before signing.
4. Validate the resulting transaction hash on the server. Never fulfil an order from a browser success message alone.
5. Fulfil from a verified webhook or a server-side validation result, and deduplicate by event ID.
6. Match assets by code and issuer (Solana mint, Monad ERC-20 contract). Use the package's network-specific asset constants: a `dv_` key with a mainnet issuer will not settle.
7. Amounts are decimal strings: at most 7 decimals on Stellar, the asset's own decimals elsewhere.
8. Express webhooks need the raw body; a JSON body parser breaks signature verification.

## Verify before finishing

- Exercise the flow on testnet with a `dv_` key.
- Confirm the rendered destination, asset, amount, memo and fee, and that the wallet and intent are on the same network.
- Reject a mismatched transaction during server-side validation.
- Verify webhook signatures and deduplicate stable event IDs.
- Run the consuming application's typecheck and tests.
