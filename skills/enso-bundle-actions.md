---
name: Compose ordered onchain actions with the Bundle API
description: Use Enso's Bundle API to compose a list of ordered Actions (deposit, swap, borrow, bridge, flashloan, etc.) into a single signer-ready transaction.
api: openapi/enso-openapi-original.json
operations: [ActionsController_findAll, StandardsController_standards, BundleController_bundleShortcutTransaction]
---

# Compose ordered onchain actions with the Bundle API

Use the Bundle API when your product controls the exact ordered sequence of onchain Actions (versus letting Route find the path). Enso packs them into one executable transaction.

## Auth
`Authorization: Bearer <API_KEY>` (or `apikey=` query param). See authentication/enso-authentication.yml.

## Steps
1. **Discover actions** — `ActionsController_findAll` (`GET /api/v1/actions`) lists the Actions available to bundle (e.g. `route`, `deposit`, `borrow`, `swap`, `bridge`, `flashloan`, `transfer`).
2. **Check protocol standards** — `StandardsController_standards` (`GET /api/v1/standards`) returns per-protocol standards with the exact supported actions and inputs, so you build valid action args.
3. **Bundle** — `BundleController_bundleShortcutTransaction` (`POST /api/v1/shortcuts/bundle`) with `chainId`, `fromAddress`, and an ordered `actions[]` array (each with `protocol`, `action`, and typed `args`). The response returns a single `tx`.
4. **Sign & broadcast** — the caller signs and submits the returned transaction.

## Rules
- Action `args` must match the schemas in the OpenAPI (e.g. `DepositActionDto`, `SwapActionDto`, `BridgeActionDto`); invalid args return 400.
- Outputs of one action can feed inputs of the next (use `MinAmountOut`, `Slippage`, `Fee` helper actions where needed).
- Respect rate limits (429) — see rate-limits/enso-rate-limits.yml.
