# Docs say the per-account position cap is "a tenth of the market's open interest cap on most markets"; it is 1/10 on only 1 of 21 markets
Severity: normal
Area: docs

## Steps to reproduce
1. Read the claim in two places:
   - https://app.testnet.dreamdex.io/docs/developers/perps-testnet, Common pitfalls: "Each account has a position cap per market, a tenth of the market's open interest cap on most markets."
   - https://app.testnet.dreamdex.io/docs/trading/perps-testnet, the note under Markets: "A single account is capped at a tenth of it on most markets."
2. For each pool, read `getPerpPoolParameters()` and compare `maxPositionSize` (5th field) with `maxOpenInterest` (4th field).

## Expected
maxPositionSize / maxOpenInterest is about 0.1 on most markets.

## Actual
The ratio is 1/10 only on TAO. It is 1/20 on all 10x and 20x markets except BTC (BNB 1/19.8), 1/40 on six of the eight 5x markets (ZRO, LIT, VVV, 1000PEPE, PUMP, WLD), 1/4 on ZEC and about 1/4.9 on BTC. A bot that derives its per-account cap from this rule overestimates it by 2x to 4x on most markets (so its orders revert with `MaxPositionSizeExceeded()`), and underestimates it on BTC and ZEC.

## Evidence
```
R=https://api.hideki2.infra.testnet.somnia.network
SIG="getPerpPoolParameters()((uint256,uint256,uint256,uint256,uint256,uint256,int256,uint256))"
cast call 0xd9b6a347a3ae9fd6ba21179d3343f2c8a922f697 "$SIG" --rpc-url $R   # TAO
(2000, 1000, 500, 2185491000000, 218549000000, 0, 0, 0)   -> 1/10.0
cast call 0xd6cb1dd58580b84d76068ac21594219ce03aeddf "$SIG" --rpc-url $R   # ETH
(500, 250, 125, 7991225000000000000000 [7.991e21], 399561200000000000000 [3.995e20], 0, 0, 0)   -> 7991.225 vs 399.5612 ETH -> 1/20.0
cast call 0x2a1822b02039ddf0e293f3f7243673fc03eaf16c "$SIG" --rpc-url $R   # ZRO
(2000, 1000, 500, 1002398000000000000000000 [1.002e24], 25059900000000000000000 [2.505e22], 0, 0, 0) -> 1/40.0
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "$SIG" --rpc-url $R   # BTC
(500, 250, 125, 49363090000, 10000000000, 0, 0, 0)        -> 493.6309 vs 100 BTC -> 1/4.94
cast call 0x2a73129f26917124bd2a47a9b355c395c537f050 "$SIG" --rpc-url $R   # ZEC
(2000, 1000, 500, 168000000000 [1.68e11], 42000000000 [4.2e10], 0, 0, 0)   -> 1680 vs 420 ZEC -> 1/4.0
```
Ratios across all 21 markets: TAO 1/10; NEAR, XLM, AVAX, LINK, SUI, ADA, DOGE, XRP, HYPE, SOL, ETH 1/20; BNB 1/19.8; ZRO, LIT, VVV, 1000PEPE, PUMP, WLD 1/40; ZEC 1/4; BTC 1/4.94.
`client.getMaxPerpOrderSize(...)` reports the real cap correctly as `maxPositionSize` (BTC: "10000000000" = 100 BTC). The fix is in the docs: drop the ratio and point to `maxPositionSize` / `getPerpPoolParameters()`.

## Environment
SDK @somnia-chain/markets-sdk 0.32.1, Node 22, Somnia Hideki testnet (chain 50383), read-only (no wallet)
