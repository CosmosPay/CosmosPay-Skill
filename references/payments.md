# Payments

Intents are created on the server with your secret API key. The hosted checkout for every intent is `https://payments.cosmospay.lat/<id>`.

## With the SDK

Stellar is the default chain:

```ts
import { Assets, Client, TestnetAssets } from '@cosmosapp/pay_sdk';

const apiKey = process.env.COSMOS_PAY_API_KEY;
if (!apiKey) throw new Error('Missing COSMOS_PAY_API_KEY');

const client = new Client({ apiKey });
const usdc = apiKey.startsWith('prod_') ? Assets.USDC : TestnetAssets.USDC;

export async function createCheckout(orderId: string, destination: string, amount: string) {
  const intent = await client.paymentIntents.createPay({
    destination,
    amount,
    asset: usdc,
    msg: `Order ${orderId}`,
  });
  await intent.edit({ reference: orderId });
  return intent.toJSON();
}
```

Complete it in the browser (Stellar wallets):

```ts
import { WebClient } from '@cosmosapp/pay_sdk/web';

const result = await new WebClient().pay(intentPayload);
await fetch('/api/confirm', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ id: intentPayload.id, txHash: result.txHash }),
});
```

Validate on the server:

```ts
const outcome = await client.paymentIntents.validate(intentId, { txHash });
if (!outcome.valid) throw new Error(outcome.reason ?? 'Payment validation failed');
```

Use `createPay` when the payer chooses the source account. Use `createTx` (Stellar only) when the source is already known and the API should return an unsigned XDR. Omitting the asset creates a payment in the chain's coin (XLM, SOL or MON). Omitting the amount creates an open-amount intent (not on Monad); pass the chosen amount to `WebClient.pay`.

On Solana and Monad, name the chain and give the token by code and issuer (the SPL mint or the ERC-20 contract for your key's network):

```ts
const intent = await client.paymentIntents.createPay({
  chain: 'monad',                  // or 'solana'
  destination: '0x...MERCHANT',    // Solana: a base58 address
  amount: '25',                    // required on Monad
  assetCode: 'USDC',
  assetIssuer: '0x...USDC_CONTRACT',
});
intent.chain; // 'monad'
intent.uri;   // EIP-681 link (Solana: a Solana Pay link)
```

The payer opens `intent.uri` or `intent.qr` in their own wallet, or the hosted checkout. `WebClient` signing is Stellar only.

## With the REST API

`POST https://api.cosmospay.lat/cosmos-api/v1/payment-intents/pay` with `Authorization: Bearer <apiKey>`. The body takes `chain` (`stellar` by default, `solana` or `monad`), `destination`, `amount`, `assetCode`, `assetIssuer`, `memo`, `msg`, `callback`, `checkoutTheme` and `merchantFee`.

```bash
curl -X POST https://api.cosmospay.lat/cosmos-api/v1/payment-intents/pay \
  -H "Authorization: Bearer $COSMOS_PAY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "solana",
    "destination": "<merchant Solana address, base58>",
    "amount": "10",
    "assetCode": "USDC",
    "assetIssuer": "<USDC SPL mint>"
  }'
```

Chain rules:

- `destination` is a Stellar `G...` account, a Solana base58 address or a Monad `0x...` address. The network tier (mainnet or testnet) is the one your API key is for.
- Omit `assetCode` and `assetIssuer` for the chain's coin (XLM, SOL, MON). For a token, `assetIssuer` is the Stellar issuer, the Solana SPL mint or the Monad ERC-20 contract.
- On Monad the amount is required (it is how the payment is recognised) and `msg` is not supported.
- The response carries `id`, `chain`, `network`, `status`, `uri` (SEP-7, Solana Pay or EIP-681) and `qr`. Show the QR, open the `uri` in the payer's wallet, or send the payer to the hosted checkout.
- Validate with `POST /v1/payment-intents/{id}/validate` and body `{ "txHash": "..." }`, where `txHash` is in the chain's own form: Stellar hash (64 hex), Solana signature (base58) or Monad hash (`0x` + 64 hex). The Payments API also detects the payment on-chain and fires the webhooks.

## Hosted checkout, pay button and embedded checkout

Every intent is a payment link. Share `https://payments.cosmospay.lat/<id>`, or embed it without an API key:

```html
<script src="https://payments.cosmospay.lat/embed/cosmos-pay.js" defer></script>
<cosmos-pay-button link="<intent id>"></cosmos-pay-button>
<cosmos-pay-checkout link="<intent id>" lang="en" theme="auto"></cosmos-pay-checkout>
```

Show one embedded checkout per payment. See https://dev.cosmospay.lat/docs/embed.

## Fulfilment

Register a signed webhook and act on `PAYMENT_INTENT_SUCCEEDED` (`WebhookEventType.PaymentIntentSucceeded`). Store each webhook event ID and ignore duplicates. Details: https://dev.cosmospay.lat/docs/sdk/server/webhooks.
