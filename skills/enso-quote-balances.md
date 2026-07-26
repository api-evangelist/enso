---
name: Quote a swap and read wallet balances
description: Read a wallet's balances across chains and get an expected-output quote for a swap before building an executable route.
api: openapi/enso-openapi-original.json
operations: [WalletController_walletBalances, QuoteController_quote, RouterController_routeShortcutTransaction]
---

# Quote a swap and read wallet balances

Read balances and preview a swap's expected output without building calldata, then build the executable route when ready.

## Auth
`Authorization: Bearer <API_KEY>` (or `apikey=` query param).

## Steps
1. **Read balances** — `WalletController_walletBalances` (`GET /api/v1/wallet/balances`) returns token balances for the Enso Wallet tied to an address across supported chains; set `useEoa=true` to read the raw EOA balances instead.
2. **Quote** — `QuoteController_quote` (`GET /api/v1/shortcuts/quote`) returns the expected `amountOut` and selected route for `tokenIn -> tokenOut` on a `chainId`, without executable calldata. Use it to preview price/impact.
3. **Build the route** — when the quote is acceptable, call `RouterController_routeShortcutTransaction` (`GET /api/v1/shortcuts/route`) with the same params to get the signer-ready `tx` (route quotes internally, so this is the single execution call).
4. **Sign & broadcast** — caller signs and submits the route `tx`.

## Rules
- Quote is a preview; the authoritative `amountOut` comes from the route response at execution time.
- Handle 400/429 per errors/enso-problem-types.yml; back off on 429.
