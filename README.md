# Cosmos Pay agent skill

An agent skill that teaches AI coding agents to integrate [Cosmos Pay](https://cosmospay.lat): a stablecoin payments API with hosted checkout, payment links, webhooks, swaps and a non-custodial wallet, with payments on Stellar, Solana and Monad.

Once installed, an agent loads it when you ask for things like accepting USDC payments, creating payment links or a checkout, verifying signed webhooks, or swapping assets on Stellar, Solana or Monad.

## Contents

| Path | What it covers |
| --- | --- |
| `SKILL.md` | Overview, per-chain support, auth and the key-prefix network rule, payment flow, webhooks, security rules |
| `references/payments.md` | Payment intents on Stellar, Solana and Monad with the SDK or the REST API, hosted checkout and embeds |
| `references/wallets.md` | Browser wallets and the Cosmos Pay Wallet adapter (Stellar) |
| `references/operations.md` | Swaps (same-chain and cross-chain), USDT0, private payments, liquidity, ramps, KYC and other managers |

It matches `@cosmosapp/pay_sdk` 2.3.

## Install

Copy this folder into your agent's skills directory, for example `.claude/skills/cosmospay/` (project) or `~/.claude/skills/cosmospay/` (user).

## Resources

- Docs: https://dev.cosmospay.lat/docs
- Docs index for LLMs: https://dev.cosmospay.lat/docs/llms.txt
- OpenAPI spec: https://dev.cosmospay.lat/docs/openapi.json
- SDK: [`@cosmosapp/pay_sdk`](https://www.npmjs.com/package/@cosmosapp/pay_sdk) ([source](https://github.com/CosmosPay/CosmosJS_SDK))
