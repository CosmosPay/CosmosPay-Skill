# Browser wallets

`WebClient` auto-detects supported Stellar wallets. The SDK ships `CosmosWalletAdapter` for the Cosmos Wallet extension and the hosted wallet, so prefer it over a hand-built adapter:

```ts
import { Wallets } from '@cosmosapp/pay_sdk';
import { WebClient } from '@cosmosapp/pay_sdk/web';

const webClient = new WebClient({ preferredWallets: [Wallets.COSMOS] });
const wallets = await webClient.getAvailableWallets();
const connection = await webClient.connect();
const payment = await webClient.pay(intentPayload);
```

The shipped adapter registers by default, covers the extension and the hosted wallet (`new WebClient({ cosmos: { walletUrl } })`), and reads its provider live in `isAvailable()`.

On the hosted Cosmos Wallet, load its provider script before constructing the client. When the browser extension is installed it supplies the same `window.cosmosWallet` interface and takes precedence.

Never sign an opaque XDR silently. Keep the wallet's approval screen in the flow and show the decoded destination, asset, amount, memo, fee, network, and validity window.
