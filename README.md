# dreamDEX Perps Builder Bounty — Feedback

Feedback-track submission for the [dreamDEX Perps Builder Bounty](https://dorahacks.io/hackathon/dreamdex-perps-bounty/detail).

Each issue is one file in [`feedback/`](feedback/), in the bounty's template (title, severity, area, steps, expected, actual, evidence, environment).

Environment: `@somnia-chain/markets-sdk` 0.32.1, Node 22, Somnia Tokyo/Hideki testnet (chain 50383).
Test wallet: `0x3F95EB2D1C076f53381396935bDfF690763DaC4C`.

| # | Title | Severity | Area |
|---|---|---|---|
| [01](feedback/01-docs-oi-cap-table-stale.md) | Perps market table: open interest caps are wrong for 20 of 21 markets (live OI already exceeds the documented cap on BTC and XRP) | normal | docs |
| [02](feedback/02-docs-per-account-cap-not-one-tenth.md) | Docs say the per-account position cap is "a tenth of the market's open interest cap on most markets"; it is 1/10 on only 1 of 21 markets | normal | docs |
| [03](feedback/03-docs-paper-usdso-allowlist-wrong.md) | Docs say Paper USDso can't open positions in 6 markets (ZRO, LIT, VVV, 1000PEPE, PUMP, ZEC), but the chain allows all 21 | normal | docs |
| [04](feedback/04-docs-errors-page-has-no-perp-errors.md) | The Errors page the perps dev doc calls "the full list of order errors" lists no perp/MarginBank errors (InsufficientMarginForOrder, InsufficientCollateral, MaxPositionSizeExceeded, ...) | normal | docs |
| [05](feedback/05-sdk-max-and-preview-ignore-tick-lot-grid.md) | getMaxPerpOrderSize / previewPerpOrderMargin report an off-tick price or off-lot quantity as placeable/sufficient, and the pool then reverts InvalidPrice / InvalidQuantity | normal | SDK |
| [06](feedback/06-sdk-unified-createorder-min-quantity-guard.md) | Unified createOrder's minimum-size guard assumes lot == minQuantity, which is false on all 21 perp markets, so sub-minimum orders get past it and revert QuantityBelowMinimum | normal | SDK |
| [07](feedback/07-sdk-docs-perps-quickstart-wrong-symbols.md) | SDK Perps docs quick start uses symbol "BTC/USDSO:USDSO" and baseSymbol "WBTC"; on 0.32.1 every call throws "unknown symbol ... call loadMarkets() first" or returns [] | normal | docs |
