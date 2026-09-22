# Other CosmosPay operations

Inspect the installed package's version-matched guides before using an operation:

```text
node_modules/@cosmosapp/pay_sdk/llms/llms.txt
node_modules/@cosmosapp/pay_sdk/llms/llms-full.txt
```

The server client exposes focused managers for payment intents, webhooks, products, customers, analytics, assets, wallets, addresses, swaps, liquidity pools, fiat ramps, and KYC. Prefer their typed methods over hand-written HTTP requests.

Apply these rules to value-moving operations:

1. Request quotes on the server where provider credentials remain secret.
2. Render the quoted input, minimum output, fee, expiry, network, and counterparty.
3. Pass the unsigned XDR to the wallet only after the user confirms those values.
4. Decode and constrain the XDR against the confirmed quote before signing.
5. Submit once with a stable idempotency key when the manager supports it.
6. Reconcile the returned hash or operation ID from the server.

For fiat ramps, check country, currency, rail, KYC, and provider availability at runtime. Do not show a provider as usable merely because it exists in the catalog.
