# Other CosmosPay operations

Inspect the installed package's version-matched guides before using an operation:

```text
node_modules/@cosmosapp/pay_sdk/llms/llms.txt
node_modules/@cosmosapp/pay_sdk/llms/llms-full.txt
```

Live docs: https://dev.cosmospay.lat/docs/llms.txt (index) and https://dev.cosmospay.lat/docs/openapi.json (REST spec).

The server client exposes focused managers for payment intents, webhooks, products, customers, analytics, swaps, crossChainSwaps, usdt0, privatePayments, liquidity pools, fiat ramps, KYC, activity, assets, aliases, plugins, defindex, publicKey, and health. `Wallets` and `addresses` are exported helpers, not Client managers. Prefer their typed methods over hand-written HTTP requests.

Customers keep wallets on any chain: `client.customers.addWallet(customerId, { chain: 'monad', address: '0x...' })` and `removeWallet(customerId, walletId)`. `client.analytics.series({ period })` (`'day'`, `'week'`, `'month'` or `'year'`) buckets payments, swaps, liquidity, webhook deliveries and private payments over time.

Swaps:

- `client.swaps.create({ ... })` with no `chain` (or `chain: 'stellar'`) is a Stellar path payment and returns a `Swap`.
- `chain: 'solana'` (Jupiter) or `chain: 'monad'` (Kuru Flow) returns a `ChainSwap`. Both are mainnet only and non-custodial: the server builds the transaction, the wallet signs it.
- `client.crossChainSwaps` swaps between Stellar, Solana and Monad through NEAR Intents: quote, pay the returned deposit address, follow the swap to its outcome. Mainnet only; a `dev` key can list assets and quote but not create.

USDT0 (`client.usdt0`) sends Tether's omnichain USDT from Stellar to an EVM network through LayerZero: `routes()`, `quote({ to, amount })`, `create({ source, to, recipient, amount })`, sign `transfer.xdr` with the holder's wallet, then `transfer.submit({ signedXdr })`. Delivery takes about 27 minutes; follow `transfer.status` to `DELIVERED`. Mainnet only. Fees are in XLM; a 7th decimal of `amount` is dust that stays in the account. Guide: https://dev.cosmospay.lat/docs/sdk/server/usdt0.md

Private payments (`client.privatePayments`) hide the amount on-chain with confidential transfers, priced in XLM. Stellar testnet only for now, 0.18% commission per payment, and the merchant account must already be registered on the confidential token. The payer completes it in the private checkout, not with `WebClient`. Guide: https://dev.cosmospay.lat/docs/sdk/server/private-payments.md

Apply these rules to value-moving operations:

1. Request quotes on the server where provider credentials remain secret.
2. Render the quoted input, minimum output, fee, expiry, network, and counterparty.
3. Pass the unsigned XDR to the wallet only after the user confirms those values.
4. Decode and constrain the XDR against the confirmed quote before signing.
5. Submit once with a stable idempotency key when the manager supports it.
6. Reconcile the returned hash or operation ID from the server.

For fiat ramps, check country, currency, rail, KYC, and provider availability at runtime. Do not show a provider as usable merely because it exists in the catalog.
