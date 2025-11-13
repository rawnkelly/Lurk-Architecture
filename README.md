# Lurk - Real-time Trading Edge Discovery & Analysis Tool

## Overview

Lurk is a SaaS tool that automates online monitoring through intelligent agents called "Lurks". Enabling traders to enhance their signal discovery capabilities among other functionality.

## Architecture

### System Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│                 │     │                  │     │                 │
│   Next.js App   │────▶│  PostgreSQL DB   │◀────│  Admin Panel    │
│   (Vercel)      │     │  (Supabase)      │     │  (Manual Ops)   │
│                 │     │                  │     │                 │
└────────┬────────┘     └──────────────────┘     └─────────────────┘
         │                       ▲
         │                       │
         ▼                       │
┌─────────────────┐     ┌────────┴─────────┐
│                 │     │                  │
│  Clerk Auth     │     │  Stripe Billing  │
│                 │     │                  │
└─────────────────┘     └──────────────────┘
```

### Core Components

#### 1. **Lurk Engine**
The heart of the system - users create "Lurks" which are persistent monitoring agents:

- **Base Lurks**: Simple keyword/handle monitoring
- **Focused Lurks**: Enhanced with AI-powered narrowing for higher quality results
- **Multi-platform Support**: Currently Twitter/X, with Instagram, LinkedIn, TikTok planned

#### 2. **Narrowing System**
Our key differentiator - prevents information overload:

```typescript
interface NarrowedLurkConfig {
  baseSearch: string;          // "AI news"
  includeKeywords: string[];   // ["OpenAI", "Claude", "funding"]
  excludeKeywords: string[];   // ["memes", "art", "jokes"]
  followAccounts: string[];    // ["@sama", "@AnthropicAI"]
  minimumEngagement: number;   // 100+ likes
  postsPerSummary: number;     // 10 posts per digest
}
```

#### 3. **Manual Operation Pipeline**
Current MVP uses human-powered content discovery with AI enhancement:

1. Admin views pending lurks sorted by priority (Plus users first)
2. Manual content discovery on target platforms
3. Content pasted into admin panel
4. Claude API processes for sentiment, entities, and summaries
5. Results delivered to user dashboard

### Data Models

#### Core Tables
- **users**: Clerk-synced user accounts with admin flags
- **lurks**: User-defined monitoring configurations
- **lurk_results**: Discovered content with AI analysis
- **lurk_operations**: Tracking for usage/billing
- **teams**: Stripe subscription management

#### Enhanced Lurk Schema
```sql
lurks {
  id, userId, name,
  type: 'keyword' | 'handle' | 'topic',
  searchTerms: jsonb,      // ["@elonmusk", "SpaceX"]
  platforms: jsonb,        // ["twitter", "instagram"]
  frequency: 'daily' | 'weekly',
  lastLurkedAt: timestamp,
  metadata: jsonb          // Narrowing config stored here
}
```

### Technical Stack

- **Frontend**: Next.js 15 (App Router), React 19, Tailwind CSS
- **Authentication**: Clerk (handles all auth flows)
- **Database**: PostgreSQL with Drizzle ORM
- **Payments**: Stripe (subscriptions, webhooks)
- **AI Processing**: Anthropic Claude API
- **Deployment**: Vercel (frontend), Supabase (database)
- **Email**: Resend API

### Security & Privacy

- **Data Isolation**: Strict user-based queries, no cross-tenant access
- **Admin Access**: Hardcoded email whitelist + database flag
- **API Protection**: All endpoints require Clerk authentication
- **Webhook Security**: Stripe signature verification + idempotency
- **No Private Data**: Platform only monitors public social media content

### Performance Optimizations

1. **Database Indexes**
   - User lookups by email
   - Lurk queries by status and last processed time
   - Admin operations by timestamp

2. **Efficient Queries**
   - Batched lurk processing
   - Pagination on all list endpoints
   - Optimistic UI updates

3. **Smart Prioritization**
   - Plus subscribers processed first
   - Older lurks prioritized over recent ones
   - First-time lurks get priority boost

### Subscription Tiers

1. **Lurk Standard** ($4.99/mo)
   - 50 lurks/month ($0.09 per extra lurk)
   - 5 simultaneous lurks
   - Daily summary emails
   - Basic sentiment analysis
   - 7-day result history
   - Email support
   - Lurk one platform of your choice

2. **Lurk Plus** ($9.99/mo)
   - 200 lurks/month ($0.09 per extra lurk)
   - 20 simultaneous lurks
   - Real-time notifications
   - Advanced analytics
   - 30-day result history
   - Priority lurking & support
   - Early access to new platforms

3. **Lurk Business** ($99.99/mo)
   - Everything in Plus
   - Unlimited lurks/month
   - 100 simultaneous lurks
   - Premium analytics
   - 180-day result history
   - Premium support
   - All platforms simultaneously

### API Endpoints

#### Public Routes
- `GET /` - Landing page
- `GET /pricing` - Subscription tiers
- `POST /api/stripe/webhook` - Payment processing

#### Authenticated Routes
- `GET/POST /api/lurks` - CRUD for lurks
- `GET /api/lurk-results` - Fetch findings
- `POST /api/subscription/*` - Manage billing

#### Admin Routes
- `GET /api/admin/pending-lurks` - Processing queue
- `POST /api/admin/submit-finding` - Manual content entry
- `POST /api/admin/reconcile-subscriptions` - Stripe sync

### Deployment Architecture

```
Production Environment:
├── Vercel (Next.js App)
│   ├── Edge Functions
│   ├── Image Optimization
│   └── Analytics
│
├── Supabase (PostgreSQL)
│   ├── Connection Pooling
│   ├── Automated Backups
│   └── Row Level Security
│
└── External Services
    ├── Clerk (Auth)
    ├── Stripe (Payments)
    ├── Anthropic (AI)
    └── Resend (Email)
```

### Local Development

```bash
# Setup
npm install
npm run db:setup      # Interactive Postgres + Stripe setup
npm run db:migrate    # Run migrations
npm run db:seed       # Optional: Add test data

# Development
npm run dev           # Start Next.js
npm run db:studio     # Drizzle Studio GUI

# Admin Tools
npm run make-admin your-email@example.com
```

### Future Scaling Path

1. **Automated Content Discovery**
   - Official API integrations
   - Web scraping infrastructure
   - Real-time webhooks

2. **Advanced AI Features**
   - Trend detection
   - Competitor analysis
   - Predictive alerts

3. **Enterprise Features**
   - Team workspaces
   - API access
   - Custom integrations
   - White-label options

### Architecture Decisions

1. **Why Manual Operations?**
   - Avoid API rate limits and ToS issues during MVP
   - Maintain high content quality
   - Learn user patterns before automation

2. **Why Narrowing System?**
   - Differentiates from basic keyword alerts
   - Reduces noise, increases signal
   - Creates stickier user experience

3. **Why Clerk + Stripe + Supabase?**
   - Best-in-class for each function
   - Minimal custom auth/payment code
   - Focus on core product features

### Monitoring & Operations

- **Error Tracking**: Structured logging for debugging
- **Webhook Reliability**: Idempotency + retry logic
- **Usage Tracking**: Per-user operation limits
- **Admin Dashboard**: Real-time queue visibility

### Technical Challenges

#### 1. **Stripe Webhook Reliability & Edge Cases**

**Problem**: Stripe webhooks can arrive out of order, be duplicated, or fail mid-processing. A subscription could be created, updated, and canceled in rapid succession with webhooks arriving in any order.

**Solution**: Built a robust webhook processing system with:
- Idempotency checks using `stripe_event_id` as a unique key
- Database transaction wrapper ensuring all-or-nothing updates
- Event replay capability for failed webhooks
- Reconciliation job to sync database state with Stripe's truth

```typescript
// Store every webhook event with processing status
webhookEvents {
  stripeEventId: unique,
  processingStatus: 'success' | 'failed' | 'retrying',
  retryCount: number,
  processingError: text
}
```

#### 2. **The Narrowing System - Reducing Noise at Scale**

**Problem**: A lurk for "AI" returns 90% garbage (memes, "AI generated art", chatbot screenshots). Users get overwhelmed and churn.

**Solution**: Built a two-stage filtering system:
- **Pre-fetch filtering**: Suggestions engine that auto-populates likely relevant keywords and accounts based on the search term
- **Post-fetch scoring**: Engagement thresholds + exclude keywords to filter out noise
- **Smart defaults**: Platform-specific exclude lists (e.g., "moon", "lambos" for crypto searches)

The system reduced irrelevant content by ~75% in testing while maintaining high recall for actually important updates.

#### 3. **Fair Queue Processing with Limited Resources**

**Problem**: Manual operations mean limited throughput. How to fairly process lurks between free users, paying customers, and prevent gaming?

**Solution**: Multi-factor priority scoring:
```sql
-- Priority algorithm
1. Plan tier (Plus > Standard > Free)
2. Days since last lurk (older = higher priority)
3. First-time lurks get a boost
4. Total operations used this month (prevent abuse)
```

This ensures paying customers get value while free users still get reasonable service to encourage upgrades.

#### 4. **Dual User System (Clerk + Local DB)**

**Problem**: Clerk handles auth beautifully but we need local user records for relations (lurks, teams, usage tracking). Keeping them in sync is tricky.

**Solution**: 
- Lazy user creation: Only create local record when user performs first action
- Email as the canonical ID: Clerk user's email links to local database
- Middleware pattern that fetches both Clerk session and local user in one go
- Admin bypass: Hardcoded email list + database flag for double security

---

This architecture supports our current manual operations while providing clear paths to automation and scale. The modular design allows swapping components (e.g., different AI providers, payment processors) without major refactoring.
