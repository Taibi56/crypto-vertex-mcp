# Crypto Vertex MCP

Crypto Vertex is a full crypto analysis platform at https://www.cryptovertex.net: screener, order book, liquidation map, backtesting, trading challenge and forex signals. This MCP server exposes the screener and market-data part, read-only.

<!-- mcp-name: io.github.Taibi56/crypto-vertex -->

**Crypto Vertex: real-time crypto screener for Binance. Filter and rank 400+ spot and 500+ futures coins by price, volume, RSI, MACD, ADX, funding, open interest, long/short ratio and 100+ technical signals; get market-structure signals (CHoCH, BOS, CRT), market breadth, Fear & Greed and BTC dominance. Read-only. Unavailable values are null, never estimated.**

This repository documents the remote [Model Context Protocol](https://modelcontextprotocol.io) server of [Crypto Vertex](https://www.cryptovertex.net). It contains documentation only; the server is hosted by Crypto Vertex.

- **Endpoint:** `https://www.cryptovertex.net/mcp` (Streamable HTTP, stateless; use the `www` address exactly)
- **Access:** read-only. No tool can place orders, change settings or write anything.
- **Docs:** https://www.cryptovertex.net/api-docs

## Authentication

A free API key is required for `tools/call`. The discovery handshake (`initialize`, `tools/list`, `ping`, `notifications/initialized`) is open and needs no key. Sign up at https://www.cryptovertex.net, open **API keys** (https://www.cryptovertex.net/api-keys) and create a key (it starts with `cv_live_`). Send it as a header on every request:

```
Authorization: Bearer cv_live_YOUR_KEY
```

The MCP server and the REST API share the same key and the same daily limit (the counter resets at 00:00 UTC). Only `tools/call` counts as a request; the handshake methods are open without a key, are not counted, and are limited per IP. Keep your key secret and never commit it.

## Connect

**Claude Code**

```bash
claude mcp add --transport http crypto-vertex https://www.cryptovertex.net/mcp \
  --header "Authorization: Bearer cv_live_YOUR_KEY"
```

**Claude app (custom connector):** Server URL `https://www.cryptovertex.net/mcp`, authentication "No connection", then add a header `Authorization` with value `Bearer cv_live_YOUR_KEY`.

**Claude Desktop and other JSON-config clients** (via the `mcp-remote` bridge):

```json
{
  "mcpServers": {
    "crypto-vertex": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://www.cryptovertex.net/mcp",
               "--header", "Authorization:${CV_AUTH}"],
      "env": { "CV_AUTH": "Bearer cv_live_YOUR_KEY" }
    }
  }
}
```

Connectors that only offer OAuth sign-in and no custom header cannot use a key yet.

## Tools

All five tools are read-only.

| Tool | What it does |
| --- | --- |
| `get_screener` | Screen every listed USDT spot or perpetual-futures pair. Filter with `min_<field>` / `max_<field>` (price, change_24h, rsi14, adx14, macd_hist, funding_rate, market_cap…), signal ids and chart patterns, then sort and paginate (up to 200 rows per page). Filtering and sorting run over all coins before pagination. |
| `get_coin` | Everything published for one coin: price, performance, volume, trend, funding / open interest (futures), market cap, and per-timeframe indicators, patterns, divergence, support/resistance, CRT, SMC signals and a sparkline. |
| `get_signals` | Current market-structure signals: CHoCH, BOS and CRT per coin and timeframe (spot, futures, and forex CRT). Order blocks and fair value gaps are not published. |
| `get_market` | Market overview: advancers/decliners, average and median 24h change, volume, top gainers and losers, futures funding and long/short aggregates, BTC and ETH snapshots, Fear & Greed and BTC dominance. |
| `get_meta` | Describes the data: markets, timeframes, every filter and sort field, every valid signal id, coin counts and last-update times. **Call this first.** |

Markets: `spot`, `futures` (and `forex` for signals). Timeframes: `1h`, `4h`, `1d`, `1w`.

`get_screener`, `get_coin` and `get_signals` accept a `fields` argument (for example `"symbol,price,rsi14"`) so you can get short, fast answers instead of the full row.

## Example requests

Ask your assistant in plain language, or call the tools directly.

> "Which futures coins have RSI above 70 on 4h with a bullish CHoCH?"

```json
{
  "name": "get_screener",
  "arguments": {
    "market": "futures",
    "timeframe": "4h",
    "min_rsi14": 70,
    "signals": "smc-choch-bull-4h",
    "sort": "rsi14",
    "order": "desc",
    "limit": 10,
    "fields": "symbol,price,rsi14"
  }
}
```

> "The five most oversold spot coins on 4h."

```json
{ "name": "get_screener", "arguments": { "market": "spot", "timeframe": "4h", "sort": "rsi14", "order": "asc", "limit": 5, "fields": "symbol,price,rsi14" } }
```

> "Show BTC's price, RSI and MACD histogram on 4h."

```json
{ "name": "get_coin", "arguments": { "symbol": "BTC", "market": "futures", "timeframe": "4h", "fields": "price,rsi14,macd_hist" } }
```

> "Latest bearish BOS signals on futures."

```json
{ "name": "get_signals", "arguments": { "market": "futures", "type": "bos", "direction": "bearish", "limit": 20 } }
```

> "How is the market doing right now?"

```json
{ "name": "get_market", "arguments": {} }
```

## Raw protocol example

```bash
curl -s https://www.cryptovertex.net/mcp \
  -H "Authorization: Bearer cv_live_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Data notes

- **Unavailable values are returned as `null`.** Nothing is estimated, interpolated or back-filled. A coin with no data for a filter or signal never matches it.
- Responses include `meta.updated_at` so you can see how fresh the data is.
- Binance spot and perpetual-futures USDT pairs only.

## Disclaimer

Data is provided for information only and is **not financial advice**. Markets are risky; do your own research.

## License

The documentation in this repository is released under the [MIT License](LICENSE). The hosted service is operated by Crypto Vertex and is subject to its own terms on https://www.cryptovertex.net.
