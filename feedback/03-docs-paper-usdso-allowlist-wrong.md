# Docs say Paper USDso can't open positions in 6 markets (ZRO, LIT, VVV, 1000PEPE, PUMP, ZEC), but the chain allows all 21
Severity: normal
Area: docs

## Steps to reproduce
1. Read https://app.testnet.dreamdex.io/docs/trading/perps-testnet. The "Paper USDso" column says **No** for 1000PEPE, LIT, PUMP, VVV, ZEC and ZRO. https://app.testnet.dreamdex.io/docs/trading/perps-testnet/margin-and-liquidations.md adds: "You can open or increase positions only in the markets marked Yes ... The six newest markets are not on that list." The developer page also says an increase outside the allowlist "reverts, and getMaxPerpOrderSize reports it as limitedBy: \"voucherBlocked\"".
2. Read `MarginBank.isVoucherMarketAllowed(pool)` for each pool.
3. Take an account that holds only the voucher (credit floor 1000 USDso, no positions), for example `0x371881d93c3f4aef17ac079b071faace549396d8`, which was credited at 1792519030. Simulate (eth_call only) a position-opening limit order on ZRO-PERP and ZEC-PERP, and call `getMaxPerpOrderSize` for it.

## Expected
`isVoucherMarketAllowed` is false for the six "No" markets. The simulated order reverts (`VoucherMarketNotAllowed`), and `getMaxPerpOrderSize` returns `limitedBy: "voucherBlocked"`.

## Actual
`isVoucherMarketAllowed` returns true for all 21 pools. A voucher-only account can open on ZRO and ZEC: the eth_call succeeds, and the SDK reports the order as placeable, limited only by collateral. The docs, not the SDK, are what's out of date. Bots and Arena users are steered away from six markets they can actually trade.

## Evidence
```
R=https://api.hideki2.infra.testnet.somnia.network; MB=0x478769E1177C0773c43A593DCCc4Ad15F15F1394
cast call $MB "isVoucherMarketAllowed(address)(bool)" 0x2a1822b02039ddf0e293f3f7243673fc03eaf16c --rpc-url $R   # ZRO
true
cast call $MB "isVoucherMarketAllowed(address)(bool)" 0x000000000000000000000000000000000000dEaD --rpc-url $R   # control: non-pool
false
# (the same read returns true for all 21 pools from listPerpMarkets: ZRO LIT VVV 1000PEPE PUMP ZEC TAO WLD NEAR XLM AVAX LINK SUI ADA BNB DOGE XRP HYPE SOL ETH BTC)

A=0x371881d93c3f4aef17ac079b071faace549396d8
cast call $MB "getCreditFloor(address)(uint256)" $A --rpc-url $R
1000000000000000000000 [1e21]
cast call $MB "getAccountState(address)((int256,uint256,address[]))" $A --rpc-url $R
(1000000000000000000000 [1e21], 0, [])

SIG="placeOrder(bool,uint64,uint256,uint256,uint64,uint8,uint8,address,uint96)"; EXP=$(( ($(date +%s)+3600)*1000000000 ))
# ZRO-PERP: GTC bid 5.1 ZRO @ 1.8 (on tick, min size)
cast call 0x2a1822b02039ddf0e293f3f7243673fc03eaf16c "$SIG" true 0 1800000000000000000 5100000000000000000 $EXP 0 0 0x0000000000000000000000000000000000000000 0 --from $A --gas-limit 10000000 --rpc-url $R
0x000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000700000000000488e9   (success=true, orderId returned)
# ZEC-PERP: GTC bid 0.009 ZEC @ 1100
cast call 0x2a73129f26917124bd2a47a9b355c395c537f050 "$SIG" true 0 1100000000000000000000 900000 $EXP 0 0 0x0000000000000000000000000000000000000000 0 --from $A --gas-limit 10000000 --rpc-url $R
0x000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000001100000000001b0a38   (success=true, orderId returned)

# SDK: client.getMaxPerpOrderSize({pool, marginBank, account: A, isBid: true, price: <mark on tick>})
ZRO { placeable: true, limitedBy: 'collateral', maxQuantity: '2531600000000000000000' }
ZEC { placeable: true, limitedBy: 'collateral', maxQuantity: '410400000' }
```

## Environment
SDK @somnia-chain/markets-sdk 0.32.1, Node 22, Somnia Hideki testnet (chain 50383), read-only (no wallet; eth_call simulation only, nothing sent)
