---
name: kr-crypto-intelligence
description: Korean crypto market data API with x402 payments. Korean-to-English sentiment analysis (world's first), Kimchi Premium across 180+ tokens, Global vs Korea divergence with AI breakdown. For building agents that analyze Asian crypto markets, monitor regional regulations, or detect Korean retail sentiment shifts.
---

# KR Crypto Intelligence Integration Guide

A comprehensive guide for integrating Korean crypto market intelligence into AI agents. Access live Upbit/Bithumb data, Kimchi Premium across 180+ tokens, and AI-powered Korean sentiment analysis — all via x402 micropayments. No API keys, no accounts.

## Overview

Korean market data is the **blind spot** of every global trading agent. South Korea ranks **top 3 globally in crypto trading volume**, yet most agents have zero visibility into Upbit, Bithumb, or Korean-language news. Capital flows in/out of Korea move global prices — and they move first.

KR Crypto Intelligence closes that gap.

### Key Features

| Feature | Description |
|---------|-------------|
| **189+ Korean tokens** | Full Upbit + Bithumb coverage, 60s refresh |
| **Kimchi Premium** | Real-time price gap vs Binance for every supported token |
| **Korean Sentiment** | First-in-world Korean news → English sentiment via Claude AI |
| **Global vs Korea Divergence** | CoinGecko global price + Korean price + AI breakdown (light/deep tiers) |
| **Investment Warnings** | Live caution flags from Upbit (volume soaring, deposit soaring, listing changes) |
| **x402 Pay-per-Call** | Base, Polygon, Solana — no signup, no API keys |
| **MCP Server** | 13 tools for Claude Desktop, Cursor, ChatGPT |

### Numbers

- **11 paid endpoints** ($0.001 to $0.10 per call)
- **2 free endpoints** (`/health`, `/api/v1/symbols`)
- **3 networks** (Base mainnet, Polygon mainnet, Solana mainnet)
- **13 MCP tools** at `https://mcp.printmoneylab.com/mcp`
- **Live**: `https://api.printmoneylab.com`
- **Source**: [github.com/bakyang2/kr-crypto-intelligence](https://github.com/bakyang2/kr-crypto-intelligence)

---

## Quick Start

### Install x402 Client

**TypeScript:**
```bash
npm install x402 viem
```

**Python:**
```bash
pip install "x402[httpx,evm]"
```

### First Call — Kimchi Premium (Base, $0.001)

**TypeScript:**
```typescript
import { x402HttpClient } from 'x402';
import { exactEvm } from 'x402/mechanisms/evm/exact';
import { privateKeyToAccount } from 'viem/accounts';

const account = privateKeyToAccount(process.env.WALLET_PK as `0x${string}`);
const client = new x402HttpClient();
client.register('eip155:8453', exactEvm({ signer: account })); // Base

const res = await client.get(
  'https://api.printmoneylab.com/api/v1/kimchi-premium?symbol=BTC'
);
console.log(await res.json());
// { symbol: "BTC", premium_percent: 1.1, upbit_krw: 142000000,
//   binance_usdt: 95200.5, fx_rate: 1475.27, ... }
```

**Python:**
```python
import asyncio, getpass
from x402 import x402Client
from x402.mechanisms.evm.exact import ExactEvmScheme
from x402.http.clients.httpx import x402HttpxClient
from eth_account import Account

async def main():
    pk = getpass.getpass("Private key: ")
    signer = Account.from_key(pk)
    client = x402Client()
    client.register("eip155:8453", ExactEvmScheme(signer=signer))

    async with x402HttpxClient(client, timeout=30) as http:
        r = await http.get(
            "https://api.printmoneylab.com/api/v1/kimchi-premium?symbol=BTC"
        )
        print(r.json())

asyncio.run(main())
```

### MCP Server — Claude Desktop / Cursor

```json
{
  "mcpServers": {
    "kr-crypto": {
      "url": "https://mcp.printmoneylab.com/mcp",
      "transport": "streamable-http"
    }
  }
}
```

13 tools available: `get_kimchi_premium`, `get_kr_sentiment`, `get_global_vs_korea_divergence`, `get_global_vs_korea_divergence_deep`, `get_market_read`, `get_arbitrage_scanner`, `get_exchange_alerts`, `get_market_movers`, `get_kr_prices`, `get_stablecoin_premium`, `get_fx_rate`, `get_available_symbols`, `check_health`.

---

## Endpoints

### Korean Sentiment ($0.05) — Most Powerful

`GET /api/v1/kr-sentiment` — World's first Korean-to-English crypto sentiment. Combines exchange intelligence with Korean news (Coinness Telegram) for AI-powered insights. **1-hour cache.**

```typescript
const r = await client.get('https://api.printmoneylab.com/api/v1/kr-sentiment');
const data = await r.json();
// {
//   sentiment: "CAUTIOUS_FOMO",         // BULLISH | BEARISH | NEUTRAL | ...
//   score: 0.4,                          // -1 to +1
//   report_en: "Korean retail showing mixed signals...",
//   exchange_signals: { deposit_soaring: ["BIO","ARKM"], warnings: 2, ... },
//   news_context: { korean_count: 8, total_analyzed: 20 }
// }
```

### Global vs Korea Divergence — Light ($0.05) / Deep ($0.10)

Light tier: divergence + 1-2 sentence AI summary (60s cache).
Deep tier: + Korean news signals (Coinness 24h) + structured AI breakdown (drivers, action suggestion, confidence). 5-min cache.

```typescript
// Light
const r1 = await client.get(
  'https://api.printmoneylab.com/api/v1/global-vs-korea-divergence?symbol=BTC'
);

// Deep — uses 30s timeout for AI analysis
const r2 = await client.get(
  'https://api.printmoneylab.com/api/v1/global-vs-korea-divergence-deep?symbol=BTC'
);
const deep = await r2.json();
// {
//   divergence: { premium_pct: 0.92, direction: "positive", magnitude: "small" },
//   recent_news_signal: { korean_news_count_24h: 4, top_keywords: ["BTC","달러","선물"], sentiment_score: 0.6 },
//   ai_deep_analysis: {
//     summary: "BTC shows modest positive Korea premium...",
//     korean_market_drivers: ["...", "...", "..."],
//     implied_action_suggestion: "...",
//     confidence: "medium"
//   }
// }
```

25 supported symbols: BTC, ETH, XRP, SOL, ADA, DOGE, DOT, MATIC, LINK, AVAX, ATOM, UNI, LTC, NEAR, OP, ARB, APT, ALGO, FTM, SUI, TRX, BCH, ETC, HBAR, SHIB.

### Arbitrage Scanner ($0.01)

`GET /api/v1/arbitrage-scanner` — Token-by-token Kimchi Premium for 189+ tokens. Reverse premiums (Korean discount), Upbit-Bithumb price gaps, market share.

```typescript
const r = await client.get('https://api.printmoneylab.com/api/v1/arbitrage-scanner');
const { premiums, reverse_premiums, exchange_gaps, market_share } = await r.json();
const top = premiums.filter(p => p.premium_pct > 5);  // unusually high premium
```

### Exchange Alerts ($0.01)

`GET /api/v1/exchange-alerts` — Live caution flags from Upbit: `INVESTMENT_WARNING`, `VOLUME_SOARING`, `DEPOSIT_SOARING`, `GLOBAL_PRICE_DIFF`, `SMALL_ACCOUNTS_CONCENTRATION`. Detects new listings/delistings via market list diff every 60s.

### Market Movers ($0.01)

`GET /api/v1/market-movers` — 1-minute price surges/crashes (>1%), volume spikes (24h change), top 20 by volume on Upbit. Korean retail often leads global moves — **early signal**.

### Market Read ($0.10) — AI Synthesis

`GET /api/v1/market-read` — Combines 12+ data sources (Kimchi, stablecoin premium, Binance funding, OI, BTC dominance, Fear & Greed, Upbit/Bithumb top volume, exchange intelligence) → Claude Haiku → BULLISH/BEARISH/NEUTRAL signal with confidence score and token-level alerts.

### Stablecoin Premium ($0.001)

`GET /api/v1/stablecoin-premium` — USDT/USDC premium on Korean exchanges vs official USD/KRW. Positive = capital flowing **into** Korea. Negative = capital flowing **out**. Critical fund-flow indicator separate from Kimchi Premium.

### Basic Market Data ($0.001 each)

- `GET /api/v1/kimchi-premium?symbol=BTC` — single-token Kimchi Premium
- `GET /api/v1/kr-prices?symbol=BTC&exchange=all` — Upbit + Bithumb prices in KRW
- `GET /api/v1/fx-rate` — current USD/KRW

---

## Use Cases

### 1. Korean Market Entry Analysis (Global Funds)

Before allocating capital to a Korean position, check **stablecoin premium** + **Kimchi Premium** divergence:

```typescript
const [stable, kimchi] = await Promise.all([
  client.get('https://api.printmoneylab.com/api/v1/stablecoin-premium').then(r => r.json()),
  client.get('https://api.printmoneylab.com/api/v1/kimchi-premium?symbol=BTC').then(r => r.json()),
]);

if (stable.stablecoins.usdt.premium_percent > 0.5 && kimchi.premium_percent > 2) {
  // Strong inflow signal — Korean retail loading up
}
```

### 2. Kimchi Premium Arbitrage Bot

```typescript
const r = await client.get('https://api.printmoneylab.com/api/v1/arbitrage-scanner');
const { premiums } = await r.json();
const opportunities = premiums.filter(p =>
  Math.abs(p.premium_pct) > 3 && !p.warning && p.upbit_volume_krw > 5e9
);
```

### 3. Korean Policy / Regulation Monitoring

Poll `kr-sentiment` once per hour (built-in 1h cache means no double-charging) — when score swings >0.3 in either direction, regulatory news likely broke:

```typescript
setInterval(async () => {
  const r = await client.get('https://api.printmoneylab.com/api/v1/kr-sentiment');
  const { sentiment, score, news_context } = await r.json();
  if (Math.abs(score - lastScore) > 0.3) alert(sentiment, news_context.korean_related);
  lastScore = score;
}, 60 * 60 * 1000);
```

### 4. Global vs Korea Divergence Detection

```typescript
const r = await client.get(
  'https://api.printmoneylab.com/api/v1/global-vs-korea-divergence-deep?symbol=ETH'
);
const { divergence, ai_deep_analysis } = await r.json();
if (divergence.magnitude === 'large') {
  console.log(ai_deep_analysis.implied_action_suggestion);
}
```

### 5. Asian Signal for Trading Agents

Wire `market-read` into your agent's reasoning loop — its output is structured JSON with `signal`, `confidence`, `key_factors`, `token_alerts`, `risk_warning`. Treat as one input among many; weight by your agent's existing global signals.

---

## AWS Bedrock AgentCore Integration

AgentCore agents can call x402-protected endpoints by routing payments through a Coinbase AgentKit wallet.

### Setup (sketch)

```typescript
import { AgentKit } from '@coinbase/agentkit';
import { x402HttpClient } from 'x402';
import { exactEvm } from 'x402/mechanisms/evm/exact';

// AgentKit wallet manages the signer
const agentKit = await AgentKit.from({ cdpApiKeyName, cdpApiKeyPrivateKey });
const wallet = await agentKit.getWallet();

// Bridge to x402 client
const client = new x402HttpClient();
client.register('eip155:8453', exactEvm({ signer: wallet.toViemAccount() }));

// AgentCore tool — exposed to your Bedrock agent
export const krSentimentTool = {
  name: 'kr_sentiment',
  description: 'Korean crypto market sentiment in English. Costs $0.05 USDC.',
  invoke: async () => {
    const r = await client.get('https://api.printmoneylab.com/api/v1/kr-sentiment');
    return r.json();
  },
};
```

In your AgentCore action group, register the tool. The agent calls it, AgentKit signs the USDC transfer, x402 facilitator settles, KR Crypto returns the data — all in one round trip from the agent's perspective.

Reference: [Coinbase AgentKit docs](https://docs.cdp.coinbase.com/agentkit/welcome).

---

## Best Practices

- **Respect built-in caches.** Server-side caches: kimchi/kr-prices/fx 15s, intelligence endpoints 60s, kr-sentiment 1h, divergence-light 60s, divergence-deep 5min. Calling more often is wasted USDC.
- **Client-side cache 5min** for divergence/arbitrage/movers data. Hot symbols get nearly-free repeat reads.
- **Use 30s HTTP timeout** for `kr-sentiment`, `market-read`, `divergence-deep` — they trigger Claude Haiku which adds ~3s latency.
- **Warm Coinbase signer once.** The x402 SDK caches the signer; reuse one client across requests instead of recreating per call.
- **Handle 503 from intelligence endpoints** — happens for ~60s after server restart while the polling task collects first cycle.
- **Use Polygon for cheaper gas.** All paid endpoints accept `eip155:137` (Polygon mainnet) with native USDC at `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359`.
- **MCP for human-in-the-loop.** When a user (not agent) is driving — wire MCP and let Claude/Cursor auto-discover tools instead of hard-coding endpoint URLs.

---

## Pricing

| Endpoint | Price | Cache | Purpose |
|---|---|---|---|
| `/api/v1/kimchi-premium` | $0.001 | 15s | Single-token Kimchi |
| `/api/v1/kr-prices` | $0.001 | 15s | Upbit/Bithumb price |
| `/api/v1/fx-rate` | $0.001 | 15s | USD/KRW |
| `/api/v1/stablecoin-premium` | $0.001 | 15s | Fund flow indicator |
| `/api/v1/arbitrage-scanner` | $0.01 | 60s | 189+ tokens Kimchi |
| `/api/v1/exchange-alerts` | $0.01 | 60s | Caution flags + listings |
| `/api/v1/market-movers` | $0.01 | 60s | 1-min movers + volume |
| `/api/v1/global-vs-korea-divergence` | $0.05 | 60s | Light AI divergence |
| `/api/v1/kr-sentiment` | $0.05 | 1 hour | Korean news + AI sentiment |
| `/api/v1/global-vs-korea-divergence-deep` | $0.10 | 5 min | Deep AI + news signal |
| `/api/v1/market-read` | $0.10 | n/a | Full market AI synthesis |
| `/api/v1/symbols` | free | 5 min | Symbol list |
| `/health` | free | n/a | Service status |

---

## Live URLs

- **API**: `https://api.printmoneylab.com`
- **MCP**: `https://mcp.printmoneylab.com/mcp` (streamable-http)
- **Manifest**: `https://api.printmoneylab.com/.well-known/x402`
- **llms.txt**: `https://api.printmoneylab.com/llms.txt`
- **Source**: [github.com/bakyang2/kr-crypto-intelligence](https://github.com/bakyang2/kr-crypto-intelligence)
- **Networks**: Base (`eip155:8453`), Polygon (`eip155:137`), Solana mainnet
- **Bazaar**: indexed via CDP x402 discovery (auto-listed on Agentic.market)
