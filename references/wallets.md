# Browser wallets

`WebClient` auto-detects supported Stellar wallets and can accept a custom adapter:

```ts
import { WebClient } from '@cosmosapp/pay_sdk/web';

const webClient = new WebClient();
const wallets = await webClient.getAvailableWallets();
const connection = await webClient.connect();
const payment = await webClient.pay(intentPayload);
```

Register Cosmos Wallet through its SEP-43-style provider:

```ts
import { WebClient } from '@cosmosapp/pay_sdk/web';

const cosmos = globalThis.cosmosWallet;
const cosmosAdapter = {
  id: 'cosmos',
  name: 'Cosmos Wallet',
  async isAvailable() {
    return Boolean(globalThis.cosmosWallet);
  },
  async getPublicKey() {
    const result = await cosmos.getAddress();
    return typeof result === 'string' ? result : result.address;
  },
  async signTransaction(xdr: string, options: { networkPassphrase: string }) {
    const result = await cosmos.signTransaction(xdr, options);
    return typeof result === 'string' ? result : result.signedTxXdr;
  },
};

const webClient = new WebClient();
webClient.registerWallet(cosmosAdapter, true);
```

On the hosted Cosmos Wallet, load its provider script before constructing the adapter. When the browser extension is installed it supplies the same `window.cosmosWallet` interface and takes precedence.

Never sign an opaque XDR silently. Keep the wallet's approval screen in the flow and show the decoded destination, asset, amount, memo, fee, network, and validity window.
