# SDK Perps docs quick start uses symbol "BTC/USDSO:USDSO" and baseSymbol "WBTC"; on 0.32.1 every call throws "unknown symbol ... call loadMarkets() first" or returns []
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps. The Quick start is:
   `await exchange.loadMarkets(); await exchange.depositMargin("BTC/USDSO:USDSO", 1_000); await exchange.createOrder("BTC/USDSO:USDSO", "limit", "buy", 0.001, 62_000); ... await exchange.fetchFundingRate("BTC/USDSO:USDSO")`. The one-shot reads use `client.listPerpMarkets({ baseSymbol: "WBTC", limit: 20 })` and then `perps[0].id`. The page opens with "Perps are live on testnet: BTC/USDSO:USDSO and ETH/USDSO:USDSO".
2. Run against Hideki testnet:
```ts
const m = new SomniaMarkets({ chain: hidekiTestnet, indexerUrl: "https://hideki-dev.smk.somnia.host/v1/graphql", addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES })
await m.loadMarkets()
for (const s of ["BTC/USDso:USDso", "BTC/USDSO:USDSO"]) await m.fetchFundingRate(s)
await m.client.listPerpMarkets({ baseSymbol: "WBTC", limit: 20 })
```

## Expected
The documented symbols resolve, or symbol lookup ignores case, and the documented filter returns the BTC market.

## Actual
The real symbol is `BTC/USDso:USDso` and lookup is case-sensitive, so every quick-start call fails. The error tells you to "call loadMarkets() first" even though loadMarkets() was already called, which points developers at the wrong cause. `baseSymbol: "WBTC"` returns an empty array (the base symbol is `BTC`), so the next line, `perps[0].id`, throws a TypeError. The page also says only BTC and ETH perps are live, while `listPerpMarkets()` returns 21.

## Evidence
```
BTC/USDso:USDso OK BTC/USDso:USDso 0.00120115668112801
BTC/USDSO:USDSO InvalidInputError @somnia-chain/markets-sdk: unknown symbol BTC/USDSO:USDSO — call loadMarkets() first
btc/usdso:usdso InvalidInputError @somnia-chain/markets-sdk: unknown symbol btc/usdso:usdso — call loadMarkets() first
listPerpMarkets({baseSymbol:"WBTC"}).length = 0 ; listPerpMarkets({baseSymbol:"BTC"}).length = 1
listPerpMarkets().length = 21
```
(The page text contains "USDSO:USDSO" 5 times and never "USDso:USDso".) The app-side dev doc (app.testnet.dreamdex.io/docs/developers/perps-testnet) uses `baseSymbol === "BTC"` correctly. Only the SDK docs site is affected.

## Environment
SDK @somnia-chain/markets-sdk 0.32.1, Node 22, Somnia Hideki testnet (chain 50383), read-only (no wallet)
