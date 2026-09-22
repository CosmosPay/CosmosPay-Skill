# Payments

Create the intent on the server:

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

Complete it in the browser:

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

Use `createPay` when the payer chooses the source account. Use `createTx` when the source is already known and the API should return an unsigned XDR. Omitting the asset creates an XLM payment. Omitting the amount creates an open-amount intent; pass the chosen amount to `WebClient.pay`.

For fulfilment, register a signed webhook and act on `paymentIntentSucceeded`. Store each webhook event ID and ignore duplicates.
