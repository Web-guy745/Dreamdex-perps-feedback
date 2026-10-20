# getMaxPerpOrderSize / previewPerpOrderMargin report an off-tick price or off-lot quantity as placeable/sufficient, and the pool then reverts InvalidPrice / InvalidQuantity
Severity: normal
Area: SDK

## Steps to reproduce
BTC-PERP pool 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0: tickSize 1e17, lotSize 1000, minQuantity 13000. The account is 0xd167376D0431131b4C2EC25e3581c6AF734794F3, which holds a long.
```ts
const pool="0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0", marginBank="0x478769e1177c0773c43a593dccc4ad15f15f1394", account="0xd167376D0431131b4C2EC25e3581c6AF734794F3"
const offTick = 80000n*10n**18n + 1n
await c.getMaxPerpOrderSize({ pool, marginBank, account, isBid: true, price: offTick })
await c.previewPerpOrderMargin({ pool, marginBank, account, isBid: true, price: offTick, quantity: 13000n })
await c.previewPerpOrderMargin({ pool, marginBank, account, isBid: true, price: 80000n*10n**18n, quantity: 13001n }) // off-lot
```
Then simulate the same orders against the pool with eth_call.

## Expected
The perps docs tell bots to call getMaxPerpOrderSize "before sending rather than reading a revert afterwards", and `sufficient` is documented as "Every gate passes". So an off-grid price or quantity should come back `placeable: false` / `sufficient: false`, or throw `InvalidInputError` the way `price <= 0` already does. Snapping to the grid would also do.

## Actual
Both methods say the order is fine. The pool rejects both.

## Evidence
```
getMaxPerpOrderSize price=80000000000000000000000 -> { priceable: true, placeable: true, maxQuantity: '2473000', limitedBy: 'initialMargin' }
getMaxPerpOrderSize price=80000000000000000000001 -> { priceable: true, placeable: true, maxQuantity: '2473000', limitedBy: 'initialMargin' }
previewPerpOrderMargin price=...001 qty=13000     -> { sufficient: true, meetsInitialMargin: true }
previewPerpOrderMargin price=on-tick qty=13001    -> { sufficient: true, meetsInitialMargin: true }

R=https://api.hideki2.infra.testnet.somnia.network; EXP=$(( ($(date +%s)+3600)*1000000000 ))
SIG="placeOrder(bool,uint64,uint256,uint256,uint64,uint8,uint8,address,uint96)"; H=0xd167376D0431131b4C2EC25e3581c6AF734794F3; Z=0x0000000000000000000000000000000000000000
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "$SIG" true 0 80000000000000000000000 13000 $EXP 0 0 $Z 0 --from $H --gas-limit 10000000 --rpc-url $R
0x...01...  (success)
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "$SIG" true 0 80000000000000000000001 13000 $EXP 0 0 $Z 0 --from $H --gas-limit 10000000 --rpc-url $R
execution reverted, data: "0xaf608abb...10f0cf064dd592000001...016345785d8a0000"  = InvalidPrice(80000000000000000000001, 100000000000000000)
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "$SIG" true 0 80000000000000000000000 13001 $EXP 0 0 $Z 0 --from $H --gas-limit 10000000 --rpc-url $R
execution reverted, data: "0x4f174b29...32c9..."  = InvalidQuantity(13001, ...)
```
The SDK's own sizing is otherwise right: on-tick, 2,473,000 simulates successfully and 2,480,000 reverts `InsufficientMarginForOrder()`, which matches `maxQuantity`. Only the grid check is missing. The SDK clearly knows the grid matters (unified/exchange.js: "The pool rejects an off-lot quantity (`InvalidQuantity`) or an off-tick price (`InvalidPrice`)"), but the low-level pre-trade checks don't apply it.

## Environment
SDK @somnia-chain/markets-sdk 0.32.1, Node 22, Somnia Hideki testnet (chain 50383), read-only (no wallet; eth_call simulation only)
