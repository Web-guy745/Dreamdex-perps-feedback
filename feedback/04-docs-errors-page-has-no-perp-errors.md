# The Errors page the perps dev doc calls "the full list of order errors" lists no perp/MarginBank errors (InsufficientMarginForOrder, InsufficientCollateral, MaxPositionSizeExceeded, ...)
Severity: normal
Area: docs

## Steps to reproduce
1. https://app.testnet.dreamdex.io/docs/developers/perps-testnet, "An IOC that fills nothing reverts", ends with: "The full list of order errors is on the [Errors](../contracts/errors.md#order-rejection) page."
2. Open https://app.testnet.dreamdex.io/docs/developers/contracts/errors.md. Its sections are "SpotPool errors" and "SpotStopOrderRegistry errors", plus a Preflight section. It has 49 selector rows.
3. Simulate an over-sized perp order (eth_call only) from an account that has a BTC-PERP position, and look up the revert selector on that page.

## Expected
The page linked as the full list covers the errors a perp order can revert with: at least the three the perps page itself names (`InsufficientCollateral()` 0x3a23d825, `MaxPositionSizeExceeded()` 0x714b36d6) and the one an over-margin perp order actually returns (`InsufficientMarginForOrder()` 0x7ea61260). The voucher, isolation and restriction reverts the perps page mentions (`VoucherMarketNotAllowed`, `IsolatedMarketBlocked`, `MarketRestricted`) should be there too.

## Actual
None of them is on the page. A bot developer who gets `0x7ea61260` back from a perp order finds no match in the docs. (I checked the 49 selectors the page does list with `cast sig`, and all of them are correct.)

## Evidence
```
curl -s https://app.testnet.dreamdex.io/docs/developers/contracts/errors.md | grep -c -E "7ea61260|3a23d825|714b36d6|InsufficientMarginForOrder|MaxPositionSizeExceeded|InsufficientCollateral|VoucherMarketNotAllowed|IsolatedMarketBlocked|MarketRestricted"
0
curl -s https://app.testnet.dreamdex.io/docs/developers/contracts/errors.md | grep "^##"
## Decoding a revert
## SpotPool errors
## SpotStopOrderRegistry errors
## Preflight (HTTP API) reverts

# Real revert from a perp pool (eth_call, BTC-PERP, from an account holding a position; the SDK's max for it was 2,473,000 raw)
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "placeOrder(bool,uint64,uint256,uint256,uint64,uint8,uint8,address,uint96)" true 0 80000000000000000000000 2480000 $EXP 0 0 0x0000000000000000000000000000000000000000 0 --from 0xd167376D0431131b4C2EC25e3581c6AF734794F3 --gas-limit 10000000 --rpc-url https://api.hideki2.infra.testnet.somnia.network
execution reverted, data: "0x7ea61260"
# decoded with the SDK's exported contractErrorsAbi:
0x7ea61260 InsufficientMarginForOrder()
cast sig "InsufficientCollateral()"   -> 0x3a23d825
cast sig "MaxPositionSizeExceeded()"  -> 0x714b36d6
```

## Environment
SDK @somnia-chain/markets-sdk 0.32.1, Node 22, Somnia Hideki testnet (chain 50383), read-only (no wallet; eth_call simulation only)
