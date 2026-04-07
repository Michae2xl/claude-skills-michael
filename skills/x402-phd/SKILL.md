---
name: x402-phd
description: PhD-level reference for x402 v2 and MPP agentic payment protocols on Stellar — covers Spec v2 architecture, schemes (exact/upto), transports (HTTP/MCP/A2A), Soroban auth-entry signing, facilitators, MPP channels, Bazaar discovery, multi-chain (10+ networks), extensions, and hackathon blueprints
origin: research-session-2026-04-02
tools: Read, Write, Edit, Bash, Grep, Glob, WebFetch
---

# x402 & Agentic Payments — PhD Reference

Complete knowledge base for building payment-enabled AI agent systems on Stellar. Covers x402 (Spec v2), MPP, A2A commerce, transport protocols (HTTP/MCP/A2A), schemes (exact/upto), and infrastructure patterns.

## When to Activate

- Starting any project involving x402, MPP, or agentic payments on Stellar
- Adding pay-per-request monetization to an API or AI service
- Building agent-to-agent commerce or agent marketplaces
- Implementing MCP servers with payment capabilities
- Evaluating x402 vs MPP for a specific use case
- Adding x402 to non-HTTP transports (MCP tools, A2A agents)
- Implementing consumption-based pricing (scheme `upto`)

---

## 1. Protocol Overview

### x402

Open standard backed by Coinbase that revives HTTP `402 Payment Required` for machine-to-machine micropayments. No accounts, no subscriptions, no API keys — payment = access.

**Founded:** x402.org launched 2026-04-02. Maintained by Coinbase Developer Platform (CDP). Apache-2.0 license.

**Six Zeros:**

- Zero protocol fees (only nominal network fees)
- Zero latency (internet speed)
- Zero friction (no accounts or personal info)
- Zero centralization (neutral multi-chain standard)
- Zero restrictions (anyone can build)
- Zero wait (money moves at internet speed)

**Adopted by:** Stripe, AWS, Vercel, Cloudflare, Alchemy, Nansen, Messari, World (World ID), Stellar.

**Key metrics (as of 2026-04):** 75M+ transactions, $24M+ volume, 94K buyers, 22K sellers.

**GitHub:** `coinbase/x402` — 5,900+ stars, 1,400+ forks, 722 commits. TypeScript 43%, Python 33%, Go 23%.

### Spec v2 Architecture (Transport-Agnostic)

Spec v2 separates the protocol into three layers:

```
┌─────────────────────────────────────────┐
│  Representation (how data is sent)      │  ← HTTP headers, MCP _meta, A2A metadata
├─────────────────────────────────────────┤
│  Logic (payment formation/verification) │  ← scheme-specific (exact, upto)
├─────────────────────────────────────────┤
│  Types (core data structures)           │  ← PaymentRequirements, PaymentPayload, SettlementResponse
└─────────────────────────────────────────┘
```

- **Types**: Core structures independent of transport and scheme
- **Logic**: Scheme-dependent (exact, upto) and network-dependent (EVM, SVM, Stellar)
- **Representation**: Transport-dependent (HTTP headers, MCP `_meta`, A2A message metadata)

### MPP (Machine Payments Protocol)

Open standard co-developed by Tempo and Stripe. Extends HTTP 402 with two payment intents and supports MCP/JSON-RPC transport beyond pure HTTP.

---

## 2. x402 on Stellar — Technical Deep Dive

### Critical Stellar Difference

On EVM chains, x402 signs a **full transaction**. On Stellar, x402 signs a **Soroban auth-entry** — much lighter, no sequence number consumption, no account needed beyond the keypair.

### Complete Flow

```
1. Client  → GET /resource                           (no payment)
2. Server  → HTTP 402
             Header: PAYMENT-REQUIRED
             Body: { version, accepts: [{ scheme, price, network, payTo }] }
3. Client  → creates Soroban auth-entry for USDC transfer
             signs with Ed25519 keypair
4. Client  → GET /resource
             Header: PAYMENT-SIGNATURE: <base64-encoded payload>
5. Facilitator → POST /verify (validates auth-entry + amounts)
               → POST /settle (submits on-chain transaction)
6. Server  → HTTP 200 + resource
             Header: X-PAYMENT-RESPONSE: <settlement receipt>
```

### SDKs

**TypeScript/JavaScript:**

```bash
npm install @x402/core @x402/express @x402/fetch @x402/stellar @stellar/stellar-sdk
```

| Package            | Purpose                                                  |
| ------------------ | -------------------------------------------------------- |
| `@x402/core`       | HTTP facilitator client, payment payload types           |
| `@x402/express`    | `paymentMiddleware()` for Express/Hono servers           |
| `@x402/fetch`      | `wrapFetchWithPayment()` — intercepts 402, pays, retries |
| `@x402/stellar`    | `ExactStellarScheme` — Soroban auth-entry signing        |
| `@x402/evm`        | EVM scheme support (EIP-3009, EIP-2612)                  |
| `@x402/svm`        | Solana scheme support (SPL tokens)                       |
| `@x402/hono`       | Hono framework middleware                                |
| `@x402/next`       | Next.js middleware integration                           |
| `@x402/axios`      | Axios interceptor for auto-pay                           |
| `@x402/paywall`    | Turnkey paywall component                                |
| `@x402/extensions` | Extension system (Bazaar, gas sponsoring, etc.)          |

**Python:**

```bash
pip install x402
```

**Go:**

```bash
go get github.com/coinbase/x402/go
```

### Server (Express) — Minimal

```typescript
import express from "express";
import { paymentMiddlewareFromConfig } from "@x402/express";
import { HTTPFacilitatorClient } from "@x402/core/server";
import { ExactStellarScheme } from "@x402/stellar/exact/server";

const app = express();

app.use(
  paymentMiddlewareFromConfig(
    {
      "GET /my-service": {
        accepts: {
          scheme: "exact",
          price: "$0.01", // USDC, auto-converted
          network: "stellar:testnet",
          payTo: "G...", // recipient public key
        },
      },
    },
    new HTTPFacilitatorClient({ url: "https://www.x402.org/facilitator" }),
    [{ network: "stellar:testnet", server: new ExactStellarScheme() }],
  ),
);

app.get("/my-service", (req, res) => res.json({ secret: "valuable content" }));
app.listen(3001);
```

### Server (OpenZeppelin Facilitator — Production)

```typescript
import { paymentMiddleware, x402ResourceServer } from "@x402/express";
import { ExactStellarScheme } from "@x402/stellar/exact/server";
import { HTTPFacilitatorClient } from "@x402/core/server";

const facilitatorClient = new HTTPFacilitatorClient({
  url: "https://channels.openzeppelin.com/x402/testnet",
  createAuthHeaders: async () => ({
    verify: { Authorization: `Bearer ${OZ_API_KEY}` },
    settle: { Authorization: `Bearer ${OZ_API_KEY}` },
    supported: { Authorization: `Bearer ${OZ_API_KEY}` },
  }),
});

app.use(
  paymentMiddleware(
    {
      "GET /weather": {
        accepts: [
          {
            scheme: "exact",
            price: "$0.001",
            network: "stellar:testnet",
            payTo: "G...",
          },
        ],
        description: "Real-time weather data",
        mimeType: "application/json",
      },
    },
    new x402ResourceServer(facilitatorClient).register(
      "stellar:testnet",
      new ExactStellarScheme(),
    ),
  ),
);
```

### Client — Auto-pay

```typescript
import { wrapFetchWithPayment } from "@x402/fetch";
import { ExactStellarScheme } from "@x402/stellar/exact/client";
import { Keypair } from "@stellar/stellar-sdk";

const fetch402 = wrapFetchWithPayment(fetch, {
  network: "stellar:testnet",
  keypair: Keypair.fromSecret(process.env.STELLAR_PRIVATE_KEY!),
  scheme: new ExactStellarScheme(),
});

// Client pays automatically if server returns 402
const res = await fetch402("http://localhost:3001/my-service");
```

### Price Formats

```typescript
// String (auto-converted to USDC stroops)
price: "$0.001"

// Explicit (for non-USDC SEP-41 tokens)
price: {
  asset: "CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA",
  amount: "10000"  // base units (1 USDC = 10000000 stroops)
}
```

---

## 3. Facilitators

### Selection Guide

| Environment        | URL                                              | API Key | Best For                |
| ------------------ | ------------------------------------------------ | ------- | ----------------------- |
| Testnet (Coinbase) | `https://www.x402.org/facilitator`               | No      | Prototyping, hackathons |
| Testnet (OZ)       | `https://channels.openzeppelin.com/x402/testnet` | Yes     | Pre-prod testing        |
| Mainnet (OZ)       | `https://channels.openzeppelin.com/x402`         | Yes     | Production              |
| Self-hosted        | OpenZeppelin Relayer + plugin                    | Own key | Full control            |

### Facilitator Endpoints

```
POST /verify    — validates auth-entry + amounts + signatures
POST /settle    — submits transaction on-chain via relayer
GET  /supported — lists supported networks and schemes
GET  /health    — service health check
```

### High-Throughput Setup (Production)

To handle parallel transactions without sequence number conflicts:

- 1 fee-bump signer account (pays transaction fees)
- Up to 19 channel accounts (each has own sequence number)
- Enables 19 concurrent transactions

```bash
# stellar/x402-stellar repo provides setup script
pnpm generate-channel-accounts
```

Environment:

```env
FACILITATOR_STELLAR_FEE_BUMP_SECRET=S...
FACILITATOR_STELLAR_CHANNEL_SECRETS=S1,S2,...,S19
```

---

## 4. MPP (Machine Payments Protocol)

### Two Intents

#### Charge Intent (per-request, on-chain)

Equivalent to x402 but with Stripe/Tempo SDK. One on-chain transaction per request.

**When to use:** Low-frequency calls, external users, one-off payments.

```typescript
import { Mppx } from "mppx/server";
import { stellar } from "@stellar/mpp/charge/server";
import { USDC_SAC_TESTNET } from "@stellar/mpp";

const mppx = Mppx.create({
  secretKey: MPP_SECRET_KEY,
  methods: [
    stellar.charge({
      recipient: RECIPIENT_ADDRESS,
      currency: USDC_SAC_TESTNET,
      network: "stellar:testnet",
    }),
  ],
});

// In route handler:
const result = await mppx.charge({ amount: "0.01", description: "API call" })(
  req,
);
if (result.status === 402)
  return res.status(402).send(await result.challenge.text());
```

#### Session Intent (payment channel, off-chain)

Uses Soroban's `one-way-channel` contract. Client deposits USDC once; subsequent payments are cumulative ed25519 signatures verified locally — **zero on-chain transactions per call**.

**When to use:** High-frequency AI agent calls, streaming services, agent swarms.

**Critical properties:**

- Commitments are **cumulative** (each new signature covers the running total)
- Server tracks the **highest commitment seen** — replay protection built-in
- Final `close()` settles the cumulative amount in a single on-chain transaction

```typescript
// Server setup
import { stellar } from "@stellar/mpp/channel/server";
import { Mppx, Store } from "mppx/server";

const mppx = Mppx.create({
  secretKey: MPP_SECRET_KEY,
  methods: [
    stellar.channel({
      channel: CHANNEL_CONTRACT_ADDRESS, // C... (Soroban contract)
      commitmentKey: COMMITMENT_PUBLIC_KEY,
      store: Store.database(db), // NEVER memory in production
      network: "stellar:testnet",
    }),
  ],
});

// Client setup
import { stellar } from "@stellar/mpp/channel/client";
import { Mppx } from "mppx/client";
import { Keypair } from "@stellar/stellar-sdk";

const commitmentKey = Keypair.fromRawEd25519Seed(
  Buffer.from(SECRET_HEX, "hex"),
);

Mppx.create({
  methods: [
    stellar.channel({
      commitmentKey,
      onProgress(event) {
        if (event.type === "signed")
          console.log(`Cumulative: ${event.cumulativeAmount}`);
      },
    }),
  ],
});

// Automatic off-chain payment on each fetch
const res1 = await fetch("http://server/service");
const res2 = await fetch("http://server/service"); // cumulative: 2x cost

// Close channel — single on-chain settlement
import { close } from "@stellar/mpp/channel/server";
await close({
  channel: CHANNEL_CONTRACT_ADDRESS,
  amount: 2000000n, // bigint, base units (0.1 USDC = 1000000n for 7 decimals)
  signature: lastCommitmentSig,
  feePayer: { envelopeSigner: Keypair.fromSecret(SIGNER_SECRET) },
  network: "stellar:testnet",
});
```

### MPP vs x402 Decision Matrix

| Criteria                  | Use x402            | Use MPP Session        |
| ------------------------- | ------------------- | ---------------------- |
| Request frequency         | Low (< 10/session)  | High (> 10/session)    |
| On-chain cost sensitivity | OK with per-tx cost | Want minimal on-chain  |
| User type                 | External users      | Trusted agents/clients |
| Settlement preference     | Immediate           | Batched                |
| SDK complexity            | Simple              | Medium                 |
| Transport                 | HTTP only           | HTTP + MCP/JSON-RPC    |

---

## 5. Agentic Patterns

### Pattern 1: Wrap-and-Monetize

Any existing API → x402-protected in ~10 lines. Agents pay autonomously.

```typescript
// Before: free API
app.get("/analysis", handler);

// After: paid API
app.use(paymentMiddlewareFromConfig({ "GET /analysis": { accepts: { price: "$0.05", ... } } }, ...));
app.get("/analysis", handler);  // unchanged
```

### Pattern 2: MCP + x402 (Agent with Wallet)

MCP server holds a `STELLAR_SECRET_KEY`. When the agent calls a tool, the MCP server pays x402 automatically — transparent to the AI model.

```typescript
// In MCP tool handler:
const fetch402 = wrapFetchWithPayment(fetch, { keypair, scheme, network });
const data = await fetch402("https://paid-service.com/endpoint");
// Agent gets data, payment happened invisibly
```

**Reference:** `jamesbachini/x402-mcp-stellar` — minimal MCP server with 3 tools:

- `x402_wallet_info` — check wallet balance and config
- `x402_facilitator_supported` — query facilitator capabilities
- `fetch_paid_resource` — fetch any x402-protected URL

### Pattern 3: Sponsored Agent Account

New agents need XLM to create a Stellar account, but have no XLM. Solution: operator sponsors account creation using Stellar's native sponsorship protocol.

Atomic 4-operation transaction:

1. `beginSponsoring`
2. `createAccount`
3. `changeTrust` (USDC trustline)
4. `endSponsoring`

Cost to operator: ~1.5 XLM one-time.

```typescript
// POST /create → returns unsigned XDR (agent inspects + signs locally)
// POST /submit → agent sends signed XDR, operator co-signs and submits
```

**Reference:** `oceans404/stellar-sponsored-agent-account`

### Pattern 4: Economic Load Balancing

Intercept 402 response, analyze `accepts[]` array (multiple networks supported), calculate real cost (amount + gas) per chain, select cheapest.

```typescript
interface GasEstimator {
  getNetworkCost(network: Network, amount: bigint): Promise<CostEstimate>;
}
// Cache estimates for 60s, health-check each chain
// Ranking criteria: Price | Soft-Finality | Irrevocable-Finality
```

**Reference:** `marcelosalloum/x402` (x402-hackathon branch)

### Pattern 5: Agent-to-Agent (A2A) Commerce

Each agent has its own keypair. When orchestrator calls a specialized agent, it pays via x402. Agent verifies payment before processing.

```typescript
// Orchestrator pays specialist agent
const agentEndpoint = `${AGENT_BASE_URL}/agents/${agentWallet}/analyze`;
const fetch402 = wrapFetchWithPayment(fetch, { keypair: orchestratorKeypair, ... });
const analysis = await fetch402(agentEndpoint, { method: "POST", body: JSON.stringify(project) });

// Specialist agent verifies
app.use("/analyze", paid(PRICING.singleAgentAnalysis));
app.post("/analyze", async (req, res) => {
  // payment already verified by middleware
  const result = await this.performAnalysis(req.body);
  res.json(result);
});
```

### Pattern 6: Bazaar Discovery (`/.well-known/x402`)

Expose a machine-readable catalog of all paid services. Any agent or client can discover and consume.

```json
{
  "version": "2",
  "services": [
    {
      "id": "agent:G123...",
      "name": "Shark Agent",
      "type": "investment-analyst",
      "capabilities": ["analyze_project", "score_tokenomics"],
      "pricing": { "perCall": 0.05, "currency": "USDC" },
      "endpoint": "https://platform.com/api/agents/G123.../analyze",
      "x402_endpoint": "https://platform.com/api/agents/G123.../payment",
      "mcp_endpoint": "https://platform.com/mcp"
    }
  ],
  "facilitator": "https://www.x402.org/facilitator"
}
```

---

## 6. Schemes (Payment Types)

### Scheme `exact` (Original)

Fixed-amount payment. Client signs authorization for the exact price specified by the server.

- **EVM**: Uses EIP-3009 (Transfer with Authorization) for gasless ERC-20 transfers
- **Stellar**: Signs Soroban auth-entry for USDC transfer (lighter than full tx)
- **SVM**: SPL token transfer authorization

**Verification includes**: signature validation, balance check, amount validation, time window check, parameter matching, transaction simulation.

**Supported networks**: Base, Ethereum, Avalanche, IoTeX, Polygon (EVM) | Solana (SVM) | Stellar | Aptos | Sui | Hedera | Algorand | HyperEVM

### Scheme `upto` (New in v2)

Authorizes transfer of **up to a maximum amount**. Actual charge determined at settlement based on real consumption.

```typescript
// Server returns upto scheme in 402:
{
  scheme: "upto",
  maxPrice: "$1.00",  // maximum the client can be charged
  network: "stellar:testnet",
  payTo: "G..."
}

// Settlement charges actual usage:
// e.g., 847 tokens generated × $0.001/token = $0.847 (under $1.00 max)
```

**Use cases**: LLM token billing, bandwidth metering, dynamic compute pricing, streaming services.

**Properties**: single-use authorization, explicit time limits, cryptographic recipient binding, maximum amount enforcement.

---

## 7. Transport Protocols (v2)

### 7a. HTTP (Standard)

Headers: `PAYMENT-REQUIRED` (402 response), `PAYMENT-SIGNATURE` (client request), `PAYMENT-RESPONSE` (200 response).

This is the original transport — see Section 2 for full flow.

### 7b. MCP (Model Context Protocol)

For AI agents calling MCP tools. Payment travels inside `_meta` of the tool call:

```typescript
// MCP tool call with x402 payment
{
  "method": "tools/call",
  "params": {
    "name": "analyze_project",
    "arguments": { "project": "AquaSwap" },
    "_meta": {
      "x402/payment": {
        // PaymentPayload (same structure as HTTP)
        "x402Version": 2,
        "scheme": "exact",
        "network": "stellar:testnet",
        "payload": "<base64-encoded-payload>"
      }
    }
  }
}

// MCP tool result with payment response
{
  "result": {
    "content": [...],
    "_meta": {
      "x402/payment-response": {
        "success": true,
        "network": "stellar:testnet",
        "txHash": "abc123..."
      }
    }
  }
}
```

### 7c. A2A (Agent-to-Agent Protocol)

For direct agent-to-agent payments via Google's A2A protocol. Uses message metadata:

```typescript
// Agent declares x402 support in AgentCard
{
  "capabilities": {
    "extensions": [{
      "uri": "https://github.com/google-a2a/a2a-x402/v0.1",
      "required": true
    }]
  }
}

// A2A message metadata keys:
// x402.payment.status    — "required" | "completed" | "failed"
// x402.payment.required  — PaymentRequirements object
// x402.payment.payload   — PaymentPayload from client
// x402.payment.receipts  — Settlement receipts
```

Integrates with the A2A task lifecycle — payment status transitions align with task state changes.

---

## 8. Extensions (v2)

### Bazaar (Discovery API)

Machine-readable catalog of x402-protected resources. Servers declare endpoint specs for facilitator-powered discovery:

```
GET /discovery/resources?type=api&page=1&limit=20
```

Returns: method, parameters, output format, pricing, and network info for each endpoint.

### Other Extensions

| Extension                     | Purpose                                              |
| ----------------------------- | ---------------------------------------------------- |
| **EIP-2612 Gas Sponsoring**   | Sponsor gas fees for users via EIP-2612 permits      |
| **ERC-20 Gas Sponsoring**     | Sponsor gas for ERC-20 token transfers               |
| **Payment Identifier**        | Unique identifiers for tracking/reconciling payments |
| **Sign-in with X**            | Auth integration with discounted pricing             |
| **Extension Offer & Receipt** | Structured offers and receipts for complex flows     |

---

## 9. Supported Networks

| Category    | Networks                                                                          |
| ----------- | --------------------------------------------------------------------------------- |
| **EVM**     | Base, Base Sepolia, Ethereum, Avalanche, Avalanche Fuji, IoTeX, Polygon, HyperEVM |
| **SVM**     | Solana                                                                            |
| **Stellar** | Stellar mainnet, Stellar testnet                                                  |
| **Other**   | Aptos, Sui, Hedera, Algorand                                                      |

**Primary token**: USDC (EIP-3009 on EVM, SPL on Solana, SEP-41 on Stellar). Any EIP-3009-compatible ERC-20 can be supported.

---

## 10. Ecosystem (250+ Projects)

| Category          | Count | Highlights                                                 |
| ----------------- | ----- | ---------------------------------------------------------- |
| **Clients**       | 25+   | 1Pay.ing, AgentCash, Primer, Zerion, thirdweb, Nuwa AI     |
| **Services**      | 100+  | AskClaude ($0.01-$0.10), BlockRun.AI, Firecrawl, Bitrefill |
| **Infra/Tooling** | 90+   | Foldset, tollbooth, x402-go, Mogami (Java), Atomic Rail    |
| **Facilitators**  | 28    | CDP (free, USDC on Base), OpenZeppelin, Heurist, PayAI     |
| **Learning**      | Many  | x402 Example Gallery, API Paywall Cookbook                 |

---

## 11. Standardized Error Codes

| Code                                        | Meaning                       |
| ------------------------------------------- | ----------------------------- |
| `insufficient_funds`                        | Wallet balance too low        |
| `invalid_exact_evm_payload_signature`       | Bad EVM signature             |
| `invalid_exact_evm_payload_authorization_*` | Auth timing or value mismatch |
| `invalid_network`                           | Unsupported network           |
| `invalid_scheme` / `unsupported_scheme`     | Scheme not available          |
| `invalid_x402_version`                      | Wrong protocol version        |
| `invalid_transaction_state`                 | Tx in unexpected state        |
| `unexpected_verify_error`                   | Facilitator verify failure    |
| `unexpected_settle_error`                   | Facilitator settle failure    |

---

## 12. Pricing Strategy

### Granularity Examples (from stellar-observatory)

| Endpoint            | Price   | Rationale                  |
| ------------------- | ------- | -------------------------- |
| Simple status check | $0.0001 | Near-free, encourage usage |
| Standard query      | $0.001  | Core product               |
| Enriched data       | $0.005  | More compute               |
| Full dataset        | $0.01   | Max value                  |
| AI inference        | $0.05+  | GPU compute                |
| Premium/rare data   | $0.025  | Scarcity pricing           |

### Pricing by Agent Tier (shark-stellar pattern)

| Tier | Creature  | Per Analysis | Per Copy |
| ---- | --------- | ------------ | -------- |
| 1    | Plankton  | $0.02        | $1.00    |
| 2    | Shrimp    | $0.03        | $2.00    |
| 3    | Fish      | $0.05        | $3.00    |
| 4    | Shark     | $0.10        | $5.00    |
| 5    | Whale     | $0.20        | $10.00   |
| 6    | Leviathan | $0.50        | $25.00   |

---

## 13. Testnet Setup

### Required Accounts

```bash
# 1. Generate keypair: https://lab.stellar.org/account/create
# 2. Fund with XLM: https://lab.stellar.org/account/fund (Friendbot)
# 3. Create USDC trustline: same lab page
# 4. Get USDC: https://faucet.circle.com (select Stellar Testnet)
```

### Environment Variables

```env
# Server
STELLAR_SECRET_KEY=S...              # server/agent signing key
STELLAR_PAYMENT_ADDRESS=G...         # recipient public key
STELLAR_NETWORK=testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
FACILITATOR_URL=https://www.x402.org/facilitator  # free testnet

# Client/Agent
STELLAR_PRIVATE_KEY=S...             # agent wallet key (has USDC)

# Production only
OZ_FACILITATOR_API_KEY=...           # OpenZeppelin API key
X402_ENABLED=true
```

---

## 14. Reference Implementations

| Repo                                        | What It Shows                              | Key Files                           |
| ------------------------------------------- | ------------------------------------------ | ----------------------------------- |
| `stellar/x402-stellar`                      | Official SDK + simple paywall + CLI client | `packages/paywall/`, `examples/`    |
| `jamesbachini/x402-mcp-stellar`             | MCP server with x402 auto-pay              | `src/tools/`, `.env.example`        |
| `jamesbachini/x402-Stellar-Demo`            | Progressive demos basic→advanced           | `server-basic/`, `client-advanced/` |
| `elliotfriend/stellar-observatory`          | Full MCP + x402 + granular pricing         | `src/routes/`, `mcp-server/`        |
| `oceans404/stellar-sponsored-agent-account` | Agent onboarding without XLM               | `src/routes/create.ts`              |
| `marcelosalloum/x402` (x402-hackathon)      | Multi-chain economic load balancer         | `src/router/`, `src/estimator/`     |
| `xlm402.com`                                | Live marketplace of 44 x402 endpoints      | `/.well-known/x402`, `/api/catalog` |

---

## 15. Live Services for Testing

### xlm402.com — 44 Ready Endpoints

```bash
# Discover all services (machine-readable)
curl https://xlm402.com/.well-known/x402
curl https://xlm402.com/api/catalog

# Categories available:
# - Weather (8 endpoints, $0.01+)
# - News (26 endpoints, $0.01+)
# - Crypto market data (4 endpoints, $0.01+)
# - AI inference/GPT (mainnet, $0.05+)
# - Image generation (mainnet, $0.10+)
# - Web extraction (mainnet, $0.03+)
```

### MPP Demo

```bash
# Live MPP server charging $0.01/request
# https://mpp.stellar.buzz (Stellar Testnet)
```

---

## 16. Security Checklist

- [ ] `STELLAR_PRIVATE_KEY` / `STELLAR_SECRET_KEY` only in `.env`, never committed
- [ ] Facilitator URL validated — only use known facilitators in production
- [ ] Price amounts validated server-side (don't trust client-provided amounts)
- [ ] Receipt verification: check `X-PAYMENT-RESPONSE` header on 200 responses
- [ ] MPP Session store: use persistent DB, never in-memory in production
- [ ] Channel commitment tracking: always persist the highest cumulative amount seen
- [ ] Rate limiting on payment endpoints to prevent facilitator abuse
- [ ] Auth-entry expiry: Soroban auth entries have TTL — validate timing windows

---

## 17. Common Errors & Fixes

| Error                              | Cause                                               | Fix                                         |
| ---------------------------------- | --------------------------------------------------- | ------------------------------------------- |
| `402` never resolves               | Client missing `@x402/fetch` wrapper                | Wrap fetch with `wrapFetchWithPayment()`    |
| `verify failed: amount mismatch`   | Price format wrong                                  | Use string `"$0.01"` not number `0.01`      |
| `settle failed: sequence conflict` | Single account, parallel txs                        | Use 19 channel accounts for high throughput |
| `MPP store error`                  | In-memory store on restart                          | Switch to `Store.database(db)`              |
| `auth entry expired`               | Clock skew or slow client                           | Increase TTL window in facilitator config   |
| `trustline not found`              | Agent wallet missing USDC trustline                 | Add `changeTrust` to onboarding flow        |
| Freighter mobile 402 fails         | Mobile Freighter doesn't support auth-entry signing | Use desktop Freighter, Albedo, or Hana      |

---

## 18. Hackathon Quick-Start (30 minutes to working demo)

```bash
# 1. Clone reference
git clone https://github.com/jamesbachini/x402-Stellar-Demo
cd x402-Stellar-Demo/server-basic

# 2. Install
npm install

# 3. Configure
cp .env.example .env
# Set PAY_TO=G... (your Stellar testnet public key)
# PRICE=$0.01, NETWORK=stellar:testnet
# FACILITATOR_URL=https://www.x402.org/facilitator

# 4. Run server
node server.js

# 5. In another terminal — client
cd ../client-basic
# Set STELLAR_PRIVATE_KEY=S... (testnet key with USDC)
node client.js
# → Sees 402 → pays → gets resource → logs Stellar TX hash
```

**Total time: ~30 minutes including testnet account setup.**

---

## Key Insight Summary

> x402 v2 makes any HTTP endpoint, MCP tool, or A2A agent machine-payable in ~10 lines. The `exact` scheme handles fixed pricing; `upto` handles consumption-based billing. MPP Session makes high-frequency agent calls virtually free on-chain. Together they form the payment layer of the agentic economy: agents discover services via Bazaar/`/.well-known/x402`, pay autonomously across 10+ networks (Stellar, EVM, Solana, Aptos, Sui, Hedera, Algorand), coordinate via MCP or A2A, and settle efficiently. With 250+ ecosystem projects, 75M+ transactions, and $24M+ monthly volume, the protocol is production-ready today.
