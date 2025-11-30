# Recall Bot

A Mastra-based AI trading agent for the Recall Network competition. This project provides a complete framework for autonomous trading on the Recall Network with real-time portfolio management, price feeds, and competition leaderboard tracking.

## 🚀 Features

- **AI-Powered Trading Agent** — GPT-4o-mini powered autonomous trader using Mastra framework
- **Recall API Integration** — Complete typed wrapper around Recall Network API
- **Portfolio Management** — Real-time portfolio tracking and balance monitoring
- **Trade Execution** — Execute spot trades with automatic retry support
- **Price Feeds** — Fetch real-time token prices across multiple chains
- **Competition Tracking** — Monitor leaderboard rankings and performance metrics
- **Type Safety** — Full TypeScript support with Zod schema validation
- **Error Handling** — Comprehensive error handling with automatic retries

## 📋 Project Structure

```
recall-bot/
├── src/
│   ├── agents/
│   │   └── recallAgent.ts         # GPT-4o-mini powered trading agent
│   ├── workflows/
│   │   └── recallWorkflow.ts       # Workflow that invokes the trading agent
│   ├── tools/
│   │   └── recallTrade.ts          # Mastra tool for executing trades
│   ├── utils/
│   │   ├── recallApi.ts            # Low-level Recall API client (typed)
│   │   └── tradingClient.ts        # High-level trading client wrapper
│   ├── examples/
│   │   └── getAgentProfile.ts      # Example: Fetch agent profile
│   └── mastra.ts                    # Mastra initialization with agents/workflows
├── .env                             # Environment variables (OPENAI_API_KEY, RECALL_API_KEY, etc.)
├── package.json                     # Dependencies and scripts
└── tsconfig.json                    # TypeScript configuration
```

## 🔧 Setup

### Prerequisites

- Node.js 18+ 
- npm or yarn
- OpenAI API key (for GPT-4o-mini)
- Recall Network API key and credentials

### Installation

1. **Clone or create the project:**
   ```bash
   npm create mastra@latest recall-bot
   cd recall-bot
   ```

2. **Install additional dependencies:**
   ```bash
   npm install axios axios-retry dotenv
   ```

3. **Configure environment variables:**
   
   Create or update `.env` in the project root:
   ```env
   # OpenAI Configuration
   OPENAI_API_KEY=sk-your-openai-api-key-here
   
   # Recall Network Configuration
   RECALL_API_KEY=your-recall-api-key
   RECALL_API_URL=https://api.sandbox.competitions.recall.network
   ```

## 🏗️ Architecture

### Low-Level API Client (`recallApi.ts`)

Provides typed functions for all Recall Network API endpoints:

```typescript
import { getPortfolio, getTokenPrice, executeTrade, getAgentProfile, getAgentDetails, getAgentBalances } from "./utils/recallApi";

// Fetch portfolio
const portfolio = await getPortfolio();
console.log(`Portfolio value: $${portfolio.totalValue}`);

// Fetch token price
const price = await getTokenPrice("0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2");
console.log(`WETH Price: $${price.price}`);

// Execute trade
const trade = await executeTrade({
  fromToken: "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",  // WETH
  toToken: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",   // USDC
  amount: "0.5",
  reason: "Rebalance portfolio"
});
console.log(`Trade ID: ${trade.transaction.id}`);

// Fetch agent profile
const profile = await getAgentProfile();
console.log(`Total Return: ${profile.percentageReturn}%`);

// Fetch agent details
const details = await getAgentDetails();
console.log(`Agent: ${details.agent.name}`);

// Fetch agent balances for competition
const balances = await getAgentBalances("comp_12345");
console.log(`Total Value: $${balances.totalValue}`);
```

**Features:**
- ✅ Automatic retry with exponential backoff (3 retries)
- ✅ Full Zod schema validation
- ✅ Type-safe request/response handling
- ✅ Bearer token authentication
- ✅ Comprehensive error messages

### High-Level Trading Client (`tradingClient.ts`)

Convenient wrapper around low-level API with consistent error handling:

```typescript
import { TradingClient } from "./utils/tradingClient";

const client = new TradingClient(process.env.RECALL_API_KEY!);

// Get portfolio
const portfolio = await client.getPortfolio();

// Execute trade
const trade = await client.executeTrade(
  "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",  // WETH
  "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",  // USDC
  "0.5",
  "Rebalance"
);

// Get agent profile
const profile = await client.getAgentProfile();

// Get agent details
const details = await client.getAgentDetails();

// Get agent balances
const balances = await client.getAgentBalances("comp_12345");

// Get leaderboard
const leaderboard = await client.getLeaderboard();
```

### AI Trading Agent (`agents/recallAgent.ts`)

GPT-4o-mini powered agent that decides trades based on user requests:

```typescript
import { recallAgent } from "./agents/recallAgent";

// Agent uses the recall-trade tool to execute trades
const response = await recallAgent.stream([
  { role: "user", content: "Swap 0.5 WETH for USDC" }
]);

for await (const chunk of response.textStream) {
  console.log(chunk);
}
```

**Token Reference (built into agent instructions):**
- USDC: `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` (Ethereum mainnet)
- WETH: `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` (Ethereum mainnet)
- WBTC: `0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599` (Ethereum mainnet)
- SOL: `Sol11111111111111111111111111111111111111112` (Solana network)

### Workflow (`workflows/recallWorkflow.ts`)

Orchestrates agent invocation and trade execution:

```typescript
import { recallWorkflow } from "./workflows/recallWorkflow";

const result = await recallWorkflow.execute();
console.log(result.result);
```

## 📦 API Endpoints

All endpoints are fully typed and validated:

### Portfolio Management

| Endpoint | Method | Function | Returns |
|----------|--------|----------|---------|
| `/agent/portfolio` | GET | `getPortfolio()` | `Portfolio` with token holdings |
| `/agent/profile` | GET | `getAgentProfile()` | `AgentProfile` with competition stats |
| `/agent/details` | GET | `getAgentDetails()` | `AgentDetailsResponse` with agent & owner |
| `/agent/balances` | GET | `getAgentBalances(competitionId)` | `AgentBalances` for specific competition |

### Trading

| Endpoint | Method | Function | Returns |
|----------|--------|----------|---------|
| `/trade/execute` | POST | `executeTrade(trade)` | `TradeResponse` with transaction details |

### Market Data

| Endpoint | Method | Function | Returns |
|----------|--------|----------|---------|
| `/price` | GET | `getTokenPrice(tokenAddress, chain, specificChain)` | `TokenPrice` |

### Leaderboard

| Endpoint | Method | Function | Returns |
|----------|--------|----------|---------|
| `/competition/leaderboard` | GET | `getLeaderboard()` | `LeaderboardResponse` |

## 🎯 Quick Start

### 1. Run the example

```bash
npx ts-node src/examples/getAgentProfile.ts
```

This will fetch and display your agent profile, portfolio performance, and owner details.

### 2. Start the dev server

```bash
npm run dev
```

Runs the Mastra dev server with hot-reload support for development.

### 3. Build for production

```bash
npm run build
```

Compiles TypeScript to JavaScript in the `dist/` directory.

### 4. Use the trading client in your code

```typescript
import { TradingClient } from "./utils/tradingClient";

async function main() {
  const client = new TradingClient(process.env.RECALL_API_KEY!);
  
  // Fetch current portfolio
  const portfolio = await client.getPortfolio();
  console.log(`Portfolio: $${portfolio.totalValue}`);
  
  // Execute a trade
  const trade = await client.executeTrade(
    "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2", // WETH
    "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48", // USDC
    "1.0"
  );
  console.log(`Trade executed: ${trade.transaction.id}`);
}

main();
```

## 🔐 Security

- **API Keys**: Store securely in `.env` (never commit to git)
- **Retry Logic**: Automatic exponential backoff prevents rate limiting
- **Error Handling**: All errors include context and are safely logged
- **Input Validation**: All requests validated with Zod schemas

## 📚 Type Safety

All API responses are validated with Zod schemas and exported as TypeScript types:

```typescript
import type {
  Portfolio,
  TokenPrice,
  TradeResponse,
  AgentProfile,
  AgentDetails,
  AgentBalances,
  LeaderboardEntry,
} from "./utils/recallApi";

// Full type safety throughout your code
const portfolio: Portfolio = await getPortfolio();
```

## 🛠️ Development

### Project Scripts

```bash
# Development with hot-reload
npm run dev

# Build TypeScript to JavaScript
npm run build

# Type check without building
npx tsc --noEmit

# Run example
npx ts-node src/examples/getAgentProfile.ts
```

### Adding New Endpoints

1. Add schema in `src/utils/recallApi.ts`:
   ```typescript
   const NewEndpointSchema = z.object({
     // ... schema definition
   });
   export type NewEndpoint = z.infer<typeof NewEndpointSchema>;
   ```

2. Add function:
   ```typescript
   export async function getNewEndpoint(): Promise<NewEndpoint> {
     // ... implementation
   }
   ```

3. Export from default:
   ```typescript
   export default { ..., getNewEndpoint };
   ```

4. Add to `TradingClient` for consistency:
   ```typescript
   async getNewEndpoint(): Promise<NewEndpoint> {
     try {
       return await getNewEndpoint();
     } catch (error: any) {
       throw new Error(`Failed to fetch: ${error.message}`);
     }
   }
   ```

## 📖 Resources

- [Mastra Documentation](https://docs.mastra.ai)
- [AI SDK (Vercel)](https://sdk.vercel.ai)
- [Recall Network](https://recall.network)
- [Zod Documentation](https://zod.dev)

## 🤝 Contributing

Contributions welcome! Please ensure:
- All code is TypeScript with full type safety
- All API functions include Zod schema validation
- Error messages are descriptive and include context
- Examples are updated with new functionality

## 📄 License

MIT

---

**Built with:**
- Mastra for AI orchestration
- Vercel AI SDK for LLM integration
- Axios for HTTP requests
- Zod for runtime type validation
