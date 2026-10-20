# Perps market table: open interest caps are wrong for 20 of 21 markets (live OI already exceeds the documented cap on BTC and XRP)
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://app.testnet.dreamdex.io/docs/trading/perps-testnet (append /README.md for the markdown) and read the "Open interest cap" column of the Markets table. The note under it says "These figures are read from the live markets".
2. For each pool, read `getPerpPoolParameters()` (the 4th field is `maxOpenInterest`) and `getOpenInterest()`, then scale by the market's `baseDecimals`.

## Expected
The documented caps match the on-chain `maxOpenInterest`, and no market's live open interest is above its documented cap.

## Actual
Only TAO matches. The on-chain cap is exactly 2x the documented one on BTC, ETH, SOL and HYPE, and exactly 4x on the other 16 markets. On BTC and XRP the live open interest is already above the documented cap (BTC 286.35 BTC open vs a documented cap of 246.81545 BTC; XRP 828,079 XRP vs 344,632 XRP). Anyone sizing against the documented cap gets the market's headroom wrong by 2x to 4x.

## Evidence
```
R=https://api.hideki2.infra.testnet.somnia.network
# BTC-PERP pool (baseDecimals 8)
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "getPerpPoolParameters()((uint256,uint256,uint256,uint256,uint256,uint256,int256,uint256))" --rpc-url $R
(500, 250, 125, 49363090000 [4.936e10], 10000000000 [1e10], 0, 0, 0)      -> maxOpenInterest 493.6309 BTC (docs: 246.81545)
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "getOpenInterest()(uint256)" --rpc-url $R
28635419000 [2.863e10]                                                     -> 286.35 BTC open, above the documented cap
# XRP-PERP pool (baseDecimals 6)
cast call 0x7058579f3248559963f79adb937c6de9f953e060 "getPerpPoolParameters()(...same tuple...)" --rpc-url $R
(500, 250, 125, 1378528000000 [1.378e12], 68926400000 [6.892e10], 0, 0, 0)  -> 1,378,528 XRP (docs: 344,632)
cast call 0x7058579f3248559963f79adb937c6de9f953e060 "getOpenInterest()(uint256)" --rpc-url $R
828079400000 [8.28e11]                                                     -> 828,079 XRP open
# TAO-PERP (the only match)
cast call 0xd9b6a347a3ae9fd6ba21179d3343f2c8a922f697 "getPerpPoolParameters()(...)" --rpc-url $R
(2000, 1000, 500, 2185491000000 [2.185e12], 218549000000 [2.185e11], 0, 0, 0) -> 2,185.491 TAO (docs: 2,185.491)
```
All 21 markets (pool addresses from `client.listPerpMarkets()`; chain values read with `getPerpPoolParameters()`, open interest from the SDK row):

| Market | Docs OI cap | Chain maxOpenInterest | Chain/Docs | Chain maxPositionSize | posCap/OIcap | indexer openInterest now |
|---|---|---|---|---|---|---|
| ZRO | 250600 | 1.0024e+06 | 4.00x | 25059.9 | 1/40.0 | 7711 |
| LIT | 138897 | 555586 | 4.00x | 13889.6 | 1/40.0 | 892.3 |
| VVV | 21786.1 | 87144.6 | 4.00x | 2178.61 | 1/40.0 | 155.77 |
| 1000PEPE | 1.30286e+08 | 5.21142e+08 | 4.00x | 1.30285e+07 | 1/40.0 | 3.4109e+06 |
| PUMP | 8.79937e+07 | 3.51975e+08 | 4.00x | 8.7993e+06 | 1/40.0 | 4.6459e+06 |
| ZEC | 420 | 1680 | 4.00x | 420 | 1/4.0 | 40.475 |
| TAO | 2185.49 | 2185.49 | 1.00x | 218.549 | 1/10.0 | 102.748 |
| WLD | 1.264e+06 | 5.056e+06 | 4.00x | 126400 | 1/40.0 | 78064 |
| NEAR | 250410 | 1.00164e+06 | 4.00x | 50000 | 1/20.0 | 238994 |
| XLM | 2.67778e+06 | 1.07111e+07 | 4.00x | 535556 | 1/20.0 | 257945 |
| AVAX | 66672.2 | 266689 | 4.00x | 13334.4 | 1/20.0 | 11262 |
| LINK | 42643.6 | 170574 | 4.00x | 8528.6 | 1/20.0 | 3429.1 |
| SUI | 635009 | 2.54004e+06 | 4.00x | 127000 | 1/20.0 | 185905 |
| ADA | 2.25468e+06 | 9.01871e+06 | 4.00x | 450934 | 1/20.0 | 130411 |
| BNB | 693.975 | 2775.9 | 4.00x | 140 | 1/19.8 | 196.878 |
| DOGE | 5.61359e+06 | 2.24544e+07 | 4.00x | 1.1227e+06 | 1/20.0 | 1.95723e+06 |
| XRP | 344632 | 1.37853e+06 | 4.00x | 68926.4 | 1/20.0 | 828079 |
| HYPE | 23812.7 | 47625.4 | 2.00x | 2381.27 | 1/20.0 | 18247.3 |
| SOL | 47654.4 | 95308.9 | 2.00x | 4765.44 | 1/20.0 | 18480.1 |
| ETH | 3995.61 | 7991.23 | 2.00x | 399.561 | 1/20.0 | 845.692 |
| BTC | 246.815 | 493.631 | 2.00x | 100 | 1/4.9 | 286.13 |

Every other column of the table (tick, min size, initial and maintenance margin, max leverage) matched the chain on all 21 markets.

## Environment
SDK @somnia-chain/markets-sdk 0.32.1, Node 22, Somnia Hideki testnet (chain 50383), read-only (no wallet). Read 2026-10-20 ~17:50 UTC.
