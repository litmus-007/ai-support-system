# AI-Powered Customer Support System

A fullstack multi-agent customer support system built with modern technologies. Features a router agent that analyzes incoming queries and delegates to specialized sub-agents (Support, Order, Billing), each with access to relevant tools.

![Architecture](docs/architecture.png)

## 🏗️ Architecture

### Multi-Agent System

```
┌─────────────────────────────────────────────────────────────┐
│                      Router Agent                            │
│  • Analyzes incoming customer queries                        │
│  • Classifies intent and delegates to appropriate sub-agent  │
│  • Handles fallback for unclassified queries                 │
└─────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  Support Agent  │ │   Order Agent   │ │  Billing Agent  │
│                 │ │                 │ │                 │
│ Tools:          │ │ Tools:          │ │ Tools:          │
│ • searchFAQs    │ │ • getOrderBy... │ │ • getInvoice... │
│ • getConv...    │ │ • getUserOrders │ │ • getUserPay... │
│ • getUserInfo   │ │ • checkDeliv... │ │ • checkRefund.. │
│                 │ │ • cancelOrder   │ │ • requestRefund │
│                 │ │ • modifyOrder   │ │ • getSubscr...  │
│                 │ │                 │ │ • cancelSub...  │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React + Vite + Tailwind CSS |
| Backend | Hono.js (Node.js) |
| Database | PostgreSQL |
| ORM | Prisma |
| AI | Vercel AI SDK + OpenAI |
| Monorepo | Turborepo |
| Type Safety | Hono RPC + Zod |

## 📁 Project Structure

```
ai-support-system/
├── apps/
│   ├── api/                    # Backend API
│   │   ├── src/
│   │   │   ├── agents/         # AI agents (Router, Support, Order, Billing)
│   │   │   ├── controllers/    # Request handlers
│   │   │   ├── services/       # Business logic
│   │   │   ├── tools/          # Agent tools
│   │   │   ├── middleware/     # Error handling, rate limiting
│   │   │   ├── routes/         # API routes with Hono RPC
│   │   │   └── db/             # Database client & seed
│   │   └── prisma/             # Database schema
│   └── web/                    # Frontend
│       └── src/
│           ├── components/     # React components
│           ├── hooks/          # Custom hooks
│           └── lib/            # API client
├── packages/
│   └── shared/                 # Shared types & schemas
├── turbo.json                  # Turborepo config
└── package.json                # Root package.json
```

## 🚀 Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL 14+
- OpenAI API Key

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/ai-support-system.git
   cd ai-support-system
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp apps/api/.env.example apps/api/.env
   ```
   
   Edit `apps/api/.env`:
   ```env
   DATABASE_URL="postgresql://postgres:postgres@localhost:5432/ai_support"
   OPENAI_API_KEY="sk-your-api-key-here"
   PORT=3001
   CORS_ORIGIN="http://localhost:5173"
   ```

4. **Set up the database**
   ```bash
   # Generate Prisma client
   npm run db:generate
   
   # Push schema to database
   npm run db:push
   
   # Seed with sample data
   npm run db:seed
   ```

5. **Start development servers**
   ```bash
   npm run dev
   ```

   This starts:
   - API server at `http://localhost:3001`
   - Frontend at `http://localhost:5173`

## 📡 API Endpoints

### Chat

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/chat/messages` | Send message (streaming SSE) |
| POST | `/api/chat/messages/sync` | Send message (non-streaming) |
| GET | `/api/chat/conversations` | List user conversations |
| GET | `/api/chat/conversations/:id` | Get conversation with messages |
| DELETE | `/api/chat/conversations/:id` | Delete conversation |

### Agents

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/agents` | List available agents |
| GET | `/api/agents/:type/capabilities` | Get agent capabilities |

### Health

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |

## 🤖 Agents & Tools

### Support Agent
Handles general support inquiries, FAQs, and troubleshooting.

**Tools:**
- `searchFAQs` - Search FAQ database
- `getConversationHistory` - Get previous conversations for context
- `getUserInfo` - Get user information

### Order Agent
Handles order status, tracking, modifications, and cancellations.

**Tools:**
- `getOrderByNumber` - Get order by order number
- `getUserOrders` - Get all orders for a user
- `checkDeliveryStatus` - Get delivery/tracking info
- `cancelOrder` - Cancel an order
- `modifyOrder` - Check modification options

### Billing Agent
Handles payment issues, refunds, invoices, and subscriptions.

**Tools:**
- `getInvoiceDetails` - Get invoice by number
- `getUserPayments` - Get payment history
- `checkRefundStatus` - Check refund status
- `requestRefund` - Initiate refund
- `getSubscription` - Get subscription details
- `cancelSubscription` - Cancel subscription

## 🔄 Streaming Response Flow

```
Client                    Server                    AI
  │                         │                        │
  ├──POST /messages────────►│                        │
  │                         ├──Route to agent───────►│
  │◄──SSE: thinking─────────┤                        │
  │◄──SSE: routing──────────┤◄──Agent selected──────┤
  │◄──SSE: tool_call────────┤◄──Tool execution──────┤
  │◄──SSE: tool_result──────┤                        │
  │◄──SSE: text_delta───────┤◄──Stream response─────┤
  │◄──SSE: text_delta───────┤◄─────────────────────┤
  │◄──SSE: done─────────────┤◄──Complete───────────┤
  │                         │                        │
```

## ✨ Features

- [x] **Multi-Agent Architecture** - Router delegates to specialized agents
- [x] **Controller-Service Pattern** - Clean separation of concerns
- [x] **Streaming Responses** - Real-time SSE streaming
- [x] **Tool Execution** - Agents use tools to query database
- [x] **Conversation Context** - Maintains history across messages
- [x] **Hono RPC** - End-to-end type safety
- [x] **Monorepo** - Turborepo for efficient builds
- [x] **Rate Limiting** - Prevent abuse
- [x] **Error Handling** - Global error middleware
- [x] **Typing Indicators** - Real-time "AI is typing" feedback

## 🎯 Bonus Features

- [x] Hono RPC + Monorepo Setup (+30 points)
- [x] Rate limiting implementation
- [x] Show reasoning/thinking indicators
- [ ] Unit/integration tests
- [ ] Context compaction
- [ ] Deployed live demo
- [ ] useworkflow.dev integration

## 🧪 Testing

```bash
# Run all tests
npm run test

# Run API tests
cd apps/api && npm run test
```

## 📦 Building for Production

```bash
# Build all packages
npm run build

# Start production server
cd apps/api && npm run start
```

## 🔧 Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | - |
| `OPENAI_API_KEY` | OpenAI API key | - |
| `OPENAI_BASE_URL` | Custom OpenAI-compatible API base URL | - |
| `PORT` | API server port | 3001 |
| `CORS_ORIGIN` | Allowed CORS origin | * |

## 📝 License

MIT

## 👤 Author

Your Name

---

Built with ❤️ using Hono, React, Prisma, and Vercel AI SDK
