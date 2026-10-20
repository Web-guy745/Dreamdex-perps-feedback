# Unified createOrder's minimum-size guard assumes lot == minQuantity, which is false on all 21 perp markets, so sub-minimum orders get past it and revert QuantityBelowMinimum
Severity: normal
Area: SDK

## Steps to reproduce
```ts
const m = new SomniaMarkets({ chain: hidekiTestnet, indexerUrl: "https://hideki-dev.smk.somnia.host/v1/graphql", addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES })
const mk = await m.loadMarkets()
mk["BTC/USDso:USDso"].limits          // { amount: { min: 0.00013 } }, lot 0.00001
for (const amt of [0.000001, 0.0001, 0.00013]) await m.createOrder("BTC/USDso:USDso", "limit", "buy", amt, 80000)
```
(No signer, so every call that gets past the SDK's input validation stops at `SignerRequiredError`. That shows which amounts the guard lets through.)

## Expected
0.0001 BTC is below the market's own `limits.amount.min` (0.00013), so it should be rejected with `InvalidInputError` like 0.000001 is.

## Actual
Only amounts below one lot are rejected. 0.0001 BTC (10 lots, under the 13-lot minimum) gets past validation. With a signer it would be sent and revert `QuantityBelowMinimum`. The guard's code comment says this band can't exist: "One lot is also the venue's `minQuantity`: every deployed venue sets tick = lot = min ... so ... there is no band where an order passes here and still reverts as below-minimum". On perps, minQuantity is 9 to 69 lots on every market (BTC 13, ETH 40, SOL 10, XRP 69, ZRO 51, ...).

## Evidence
```
symbol BTC/USDso:USDso limits {"amount":{"min":0.00013}} precision {"price":1,"amount":5}
0.000001 InvalidInputError  amount 0.000001 is below one lot on BTC/USDso:USDso (lot 0.00001) — it would place a zero-quantity order
0.0001   SignerRequiredError createTrader is authenticated — construct SomniaMarkets with a privateKey / account / walletClient
0.00013  SignerRequiredError createTrader is authenticated — ...

# What the pool does with 0.00012 BTC (12000 raw, on the lot grid, below min), eth_call only:
cast call 0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0 "placeOrder(bool,uint64,uint256,uint256,uint64,uint8,uint8,address,uint96)" true 0 80000000000000000000000 12000 $EXP 0 0 0x0000000000000000000000000000000000000000 0 --from 0xd167376D0431131b4C2EC25e3581c6AF734794F3 --gas-limit 10000000 --rpc-url https://api.hideki2.infra.testnet.somnia.network
execution reverted, data: "0xeaa68ceb...2ee0...32c8" = QuantityBelowMinimum(12000, 13000)

# minQuantity / lotSize from listPerpMarkets (all 21 > 1):
ZRO 51; LIT 28; VVV 44; 1000PEPE 27; PUMP 18; ZEC 9; TAO 44; WLD 26; NEAR 51; XLM 54; AVAX 14; LINK 9; SUI 13; ADA 46; BNB 14; DOGE 12; XRP 69; HYPE 12; SOL 10; ETH 40; BTC 13
```
Source: node_modules/@somnia-chain/markets-sdk/dist/unified/exchange.js, `createOrder`, around line 4370 (the `quantity <= 0n` check and its comment). Fix: compare against the market's `minQuantity` (it is already on the perp row and in `limits.amount.min`).

## Environment
SDK @somnia-chain/markets-sdk 0.32.1, Node 22, Somnia Hideki testnet (chain 50383), read-only (no wallet)
