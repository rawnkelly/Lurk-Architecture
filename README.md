Lurk Terminal - Technical Architecture
Overview
Lurk Terminal is an AI-powered trading terminal for prediction market arbitrage and whale tracking. The platform provides real-time price spreads between Polymarket and Kalshi, whale wallet monitoring, AI sentiment analysis, and performance tracking with public leaderboards.
Core Value Proposition: Users pay $29-99/month to spot arbitrage opportunities instantly, copy whale trades, get AI-powered market sentiment, and build verifiable trading history that creates network lock-in effects.

System Architecture
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (Next.js 15)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Terminal   │  │  Leaderboard │  │ Performance  │      │
│  │   Dashboard  │  │    Page      │  │   Tracking   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Next.js API Routes                         │
│  /api/markets | /api/whales | /api/performance | /api/lurks │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
┌──────────────────────┐    ┌──────────────────────┐
│  PostgreSQL/Supabase │    │   Redis Cache Layer  │
│  - Market Data       │    │   - Rate Limiting    │
│  - Performance       │    │   - 3sec Cache       │
│  - User Wallets      │    │   - Queue Jobs       │
│  - Subscriptions     │    └──────────────────────┘
└──────────────────────┘
                ▲
                │
┌───────────────┴────────────────┐
│   Python Data Pipeline (24/7)  │
│  ┌──────────────────────────┐  │
│  │  polymarket_fetch.py     │  │
│  │  - Alchemy Polygon RPC   │  │
│  │  - Top 100 markets       │  │
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │  kalshi_fetch.py         │  │
│  │  - WebSocket + REST API  │  │
│  │  - Matching markets      │  │
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │  arb_calculator.py       │  │
│  │  - Spread calculation    │  │
│  │  - Write to DB every 1-3s│  │
│  └──────────────────────────┘  │
└─────────────────────────────────┘
                ▲
                │
┌───────────────┴────────────────┐
│    External Data Sources        │
│  - Polymarket (Polygon)        │
│  - Kalshi API                  │
│  - Social Media (Lurk infra)   │
└─────────────────────────────────┘

Core Features
1. Arbitrage Scanner
Real-time price spread detection between Polymarket and Kalshi.
How it works:

Python script fetches prices from both platforms every 1-3 seconds
Calculates spread percentage for matched markets
Pushes clean data to PostgreSQL
Frontend polls /api/markets every 3 seconds
Displays in Bloomberg-style terminal grid

Terminal Columns:
TICKER | EVENT_CONTRACT | POLY (ODDS) | KALSHI (ODDS) | ARB % | 24H VOL
Paywall: Starter users see full data with no delays. Pro users get API access for automated execution.
2. Whale Watching
Track known whale wallets and user performance leaderboard.
Whale Wallet System:

Curated list of 50 known whale wallets (Polymarket power traders)
Background job scans wallet history via Polygon RPC
Calculates P&L metrics (volume if P&L too complex initially)
Displays last 5-10 trades for Pro users

User Wallet Connection:

RainbowKit integration for wallet linking
Automatic position history import
Performance metrics: Win rate %, total P&L, accuracy by category
Leaderboard ranking based on verified performance

3. AI Sentiment Analysis (Lurk Infrastructure)
Existing lurk system becomes the intelligence layer for prediction markets.
Integration:

User creates lurk for market topic (e.g., "Trump 2024")
Scraper fetches social media mentions (manual → automated over time)
AI processes sentiment via Anthropic API at /api/ai/process
Sentiment score displays in Terminal alongside arb data
Example: "Polymarket 52% | Kalshi 48% | Spread 4% | Sentiment -15% (bearish)"

The Alpha: Market prices lag sentiment. If Twitter crashes but prices haven't moved, that's the edge.
4. Performance Tracking & Network Lock-In
Users build verifiable prediction history that creates switching costs.
Metrics Tracked:

Win rate %
Total P&L
Accuracy by market category
Position history with entry/exit prices
Time-weighted performance

Lock-In Mechanism:

Starter: 3-month history
Pro: 2-year history
Losing 12-18 months of verified performance = can't leave without losing credibility
Export/share functionality (screenshot-friendly for flexing)

5. Copy Trading (Pro Feature)
Auto-follow whale positions in real-time.
Workflow:

Pro user selects whale to follow from leaderboard
System monitors whale wallet via Polygon RPC
When whale enters position, triggers notification/webhook
User can manually mirror or (future) auto-execute via API

6. Staked Buying Power Program (Pro Feature)
Prop trading model where Lurk provides capital, user trades, profits split.
How it works:

Pro users qualify based on performance metrics
Lurk provides contractual buying power (no actual capital fronted)
User makes predictions with "house money"
Winners get 70-80% of profits, Lurk gets 20-30%
Losing traders (Starter tier) subsidize winning traders (Pro tier)
Insane retention: winners never leave because they're trading free capital

Legal Structure:

Profit-sharing agreement (not investment)
"Educational capital allocation" framing
Winner gets majority split, platform gets minority
Risk parameters enforced


Database Schema
Existing Tables (Already Built)
sqlusers (
  id, email, clerk_id,
  is_admin, is_pro,
  created_at, updated_at
)

subscriptions (
  id, user_id, stripe_subscription_id,
  plan_tier, status, current_period_end,
  created_at, updated_at
)

lurks (
  id, user_id, name, type,
  search_terms, platforms, frequency,
  last_lurked_at, metadata
)

lurk_results (
  id, lurk_id, content,
  sentiment_score, entities,
  engagement_metrics, created_at
)

user_preferences (
  id, user_id, preferences_json
)
New Tables (Need to Build)
sqlCREATE TABLE market_data (
  id UUID PRIMARY KEY,
  polymarket_id TEXT,
  kalshi_id TEXT,
  event_name TEXT,
  poly_price DECIMAL,
  kalshi_price DECIMAL,
  spread_percent DECIMAL,
  volume_24h BIGINT,
  last_updated TIMESTAMP
);

CREATE TABLE market_mappings (
  id UUID PRIMARY KEY,
  polymarket_id TEXT UNIQUE,
  kalshi_id TEXT UNIQUE,
  event_name TEXT,
  bet_type TEXT,
  created_at TIMESTAMP
);

CREATE TABLE performance_history (
  id UUID PRIMARY KEY,
  user_id TEXT,
  market_id UUID REFERENCES market_data(id),
  prediction TEXT,
  entry_price DECIMAL,
  exit_price DECIMAL,
  outcome TEXT,
  pnl DECIMAL,
  created_at TIMESTAMP
);

CREATE TABLE whale_wallets (
  id UUID PRIMARY KEY,
  wallet_address TEXT UNIQUE,
  nickname TEXT,
  total_volume BIGINT,
  win_rate DECIMAL,
  last_trade_at TIMESTAMP,
  is_verified BOOLEAN DEFAULT false
);

CREATE TABLE user_wallets (
  id UUID PRIMARY KEY,
  user_id TEXT,
  wallet_address TEXT UNIQUE,
  connected_at TIMESTAMP,
  last_synced_at TIMESTAMP
);

CREATE TABLE leaderboard_entries (
  id UUID PRIMARY KEY,
  user_id TEXT,
  rank INTEGER,
  total_pnl DECIMAL,
  win_rate DECIMAL,
  total_trades INTEGER,
  last_updated TIMESTAMP
);

CREATE TABLE copy_trade_follows (
  id UUID PRIMARY KEY,
  follower_user_id TEXT,
  whale_wallet_id UUID REFERENCES whale_wallets(id),
  auto_execute BOOLEAN DEFAULT false,
  created_at TIMESTAMP
);

Technical Stack
Frontend

Next.js 15 (App Router)
React 19
Tailwind CSS + shadcn/ui (Terminal aesthetic)
TanStack Table for arb scanner grid
RainbowKit for wallet connections
Recharts for performance visualizations

Backend

Python FastAPI (or scripts) for blockchain data ingestion
PostgreSQL via Supabase for all data storage
Redis for caching (3-second cache on market data) and rate limiting
Bull/BullMQ for job queue processing
Anthropic Claude API for AI summaries/sentiment (already integrated)

Data Sources

Polymarket: Polygon RPC via Alchemy (listening to Gnosis CTF contract events)
Kalshi: WebSocket + REST API (official)
Social Media: Existing lurk scraping infrastructure for sentiment

Infrastructure

Vercel (Next.js deployment)
Railway/DigitalOcean (Python script 24/7 runner)
Supabase (PostgreSQL + connection pooling)
Clerk Auth (already set up)
Stripe (subscriptions, webhooks - already integrated)
Resend (email delivery - already in codebase)


What's Already Built
✅ Database tables: users, subscriptions, lurks, lurk_results, user_preferences
✅ Lurk CRUD API routes at /api/lurks
✅ Dashboard connected to lurk APIs
✅ Admin panel UI at /admin
✅ AI processing endpoint at /api/ai/process (Anthropic API)
✅ Lurk results saving endpoint at /api/lurk-results
✅ Clerk authentication system
✅ Stripe payment integration
✅ Resend email service in codebase
