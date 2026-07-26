---
name: Swap or zap between two tokens with the Route API
description: Use Enso's Route API to find the optimal path between any two tokens or DeFi positions and return signer-ready calldata, then approve and execute.
api: openapi/enso-openapi-original.json
operations: [NetworksController_networks, TokensController_tokens, WalletController_createApproveTransaction, RouterController_routeShortcutTransaction]
---

# Swap or zap between two tokens with the Route API

Enso's Route API finds the optimal path (direct swaps, zaps into DeFi positions) between `tokenIn` and `tokenOut` across 250+ protocols and multiple chains, and returns executable calldata plus `amountOut` and `gas` in a single call — there is no separate quote step.

## Auth
Send `Authorization: Bearer <API_KEY>` on every request (or `apikey=<API_KEY>` query param). Get a key at https://developers.enso.build/. Default limit 10 rps / 600 rpm; a 429 means back off. The demo key `1e02632d-6feb-4a75-a157-documentation` (1 rps) works for testing.

## Steps
1. **Confirm the chain** — `NetworksController_networks` (`GET /api/v1/networks`) to resolve the `chainId` you will route on.
2. **Resolve tokens** — `TokensController_tokens` (`GET /api/v1/tokens`) to confirm `tokenIn`/`tokenOut` addresses and decimals on that chain (cursor-paginated, 1000/page).
3. **Build the route** — `RouterController_routeShortcutTransaction` (`GET /api/v1/shortcuts/route`, or `POST` for large payloads) with `chainId`, `fromAddress`, `tokenIn`, `tokenOut`, `amountIn`, and `slippage`. The response contains `tx` (to/data/value), `amountOut`, and `gas`.
4. **Approve if needed** — for ERC-20 `tokenIn`, call `WalletController_createApproveTransaction` (`GET /api/v1/wallet/approve`) and broadcast the returned approval tx before the route tx.
5. **Sign & broadcast** — the caller signs `tx` with their wallet and submits it onchain. Nothing settles server-side until you broadcast.

## Rules
- Routes are non-destructive to build/quote; state only changes when you sign and broadcast.
- Respect `slippage`; the returned `amountOut` is a quote at request time.
- Handle 400 (bad params) and 429 (rate limit) per errors/enso-problem-types.yml.
