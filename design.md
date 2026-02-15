# Repurpose.ai - Technical Design Document

**Version:** 1.0  
**Last Updated:** February 15, 2026  
**Document Owner:** Engineering Architecture Team  
**Status:** Draft for Review

---

## 1. High-Level Architecture Overview

Repurpose.ai follows a modern, cloud-native microservices architecture with clear separation between frontend, backend, AI processing, and data layers. The system is designed for horizontal scalability, fault tolerance, and cost optimization.

### Architecture Principles
- **Separation of Concerns:** Frontend, API, AI services, and data layers are independently deployable
- **Stateless Services:** All backend services are stateless for easy horizontal scaling
- **Async Processing:** Long-running AI operations use job queues for non-blocking execution
- **Defense in Depth:** Multiple layers of security (authentication, authorization, rate limiting, encryption)
- **Observability First:** Comprehensive logging, metrics, and tracing from day one
- **Cost Optimization:** Caching, prompt optimization, and efficient resource utilization

### Technology Stack Summary

**Frontend:**
- Next.js 14+ (App Router)
- React 18+
- TypeScript
- TailwindCSS
- Zustand (state management)
- React Query (data fetching)

**Backend:**
- Node.js 20+ LTS
- Express.js
- TypeScript
- PostgreSQL 15+
- Redis (caching & sessions)
- Bull (job queue)

**AI Layer:**
- OpenAI API (GPT-4)
- LangChain (abstraction layer)
- Fallback: Anthropic Claude

**Infrastructure:**
- Vercel (frontend hosting)
- AWS/GCP (backend services)
- AWS RDS / Supabase (PostgreSQL)
- AWS ElastiCache / Upstash (Redis)
- AWS S3 (file storage)
- Cloudflare (CDN & DDoS protection)

**DevOps:**
- GitHub Actions (CI/CD)
- Docker (containerization)
- Terraform (IaC)
- DataDog / New Relic (monitoring)
- Sentry (error tracking)

---

## 2. System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Browser    │  │    Mobile    │  │   API Client │              │
│  │  (Next.js)   │  │   (Future)   │  │   (Future)   │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
└─────────┼──────────────────┼──────────────────┼────────────────────┘
          │                  │                  │
          └──────────────────┴──────────────────┘
                             │
                    ┌────────▼────────┐
                    │   Cloudflare    │
                    │   CDN + WAF     │
                    └────────┬────────┘
                             │
          ┌──────────────────┴──────────────────┐
          │                                     │
┌─────────▼──────────┐              ┌──────────▼──────────┐
│   FRONTEND LAYER   │              │   BACKEND LAYER     │
│                    │              │                     │
│  ┌──────────────┐  │              │  ┌──────────────┐  │
│  │   Next.js    │  │              │  │  API Gateway │  │
│  │  App Router  │  │              │  │   (Express)  │  │
│  │              │  │              │  └──────┬───────┘  │
│  │  - SSR/SSG   │  │              │         │          │
│  │  - Auth UI   │  │              │  ┌──────▼───────┐  │
│  │  - Dashboard │  │              │  │   Auth       │  │
│  └──────────────┘  │              │  │   Service    │  │
│                    │              │  └──────┬───────┘  │
│  Deployed on:      │              │         │          │
│  Vercel            │              │  ┌──────▼───────┐  │
└────────────────────┘              │  │   Content    │  │
                                    │  │   Service    │  │
                                    │  └──────┬───────┘  │
                                    │         │          │
                                    │  ┌──────▼───────┐  │
                                    │  │   History    │  │
                                    │  │   Service    │  │
                                    │  └──────┬───────┘  │
                                    │         │          │
                                    │  ┌──────▼───────┐  │
                                    │  │   Billing    │  │
                                    │  │   Service    │  │
                                    │  └──────────────┘  │
                                    │                     │
                                    │  Deployed on:       │
                                    │  AWS ECS/GCP Run    │
                                    └──────┬──────────────┘
                                           │
                    ┌──────────────────────┼──────────────────────┐
                    │                      │                      │
          ┌─────────▼─────────┐  ┌────────▼────────┐  ┌─────────▼─────────┐
          │   AI LAYER        │  │   DATA LAYER    │  │   CACHE LAYER     │
          │                   │  │                 │  │                   │
          │  ┌─────────────┐  │  │  ┌───────────┐ │  │  ┌─────────────┐  │
          │  │  Job Queue  │  │  │  │PostgreSQL │ │  │  │    Redis    │  │
          │  │   (Bull)    │  │  │  │           │ │  │  │             │  │
          │  └──────┬──────┘  │  │  │  - Users  │ │  │  │  - Sessions │  │
          │         │         │  │  │  - History│ │  │  │  - Rate     │  │
          │  ┌──────▼──────┐  │  │  │  - Billing│ │  │  │    Limits   │  │
          │  │ AI Worker   │  │  │  └───────────┘ │  │  │  - Content  │  │
          │  │             │  │  │                 │  │  │    Cache    │  │
          │  │ - Extractor │  │  │  AWS RDS /      │  │  └─────────────┘  │
          │  │ - Generator │  │  │  Supabase       │  │                   │
          │  │ - LangChain │  │  │                 │  │  AWS ElastiCache/ │
          │  └──────┬──────┘  │  └─────────────────┘  │  Upstash          │
          │         │         │                        └───────────────────┘
          │  ┌──────▼──────┐  │
          │  │  OpenAI API │  │
          │  │  (GPT-4)    │  │
          │  │             │  │
          │  │  Fallback:  │  │
          │  │  Claude API │  │
          │  └─────────────┘  │
          └───────────────────┘

          ┌───────────────────────────────────────┐
          │   EXTERNAL SERVICES                   │
          │                                       │
          │  - Stripe (Payments)                  │
          │  - SendGrid (Email)                   │
          │  - DataDog (Monitoring)               │
          │  - Sentry (Error Tracking)            │
          │  - AWS S3 (File Storage)              │
          └───────────────────────────────────────┘
```

---

## 3. Component Breakdown

### 3.1 Frontend Layer (Next.js)

**Responsibilities:**
- Server-side rendering for SEO and performance
- Client-side interactivity and state management
- Authentication UI and session management
- Real-time generation progress updates
- Responsive design for desktop and tablet

**Key Components:**

**Pages:**
- `/` - Landing page (SSG)
- `/auth/login` - Login page
- `/auth/register` - Registration page
- `/dashboard` - Main generation interface (SSR)
- `/history` - Generation history (SSR)
- `/settings` - User settings and billing (SSR)

**Components:**
- `ContentInput` - URL/text input with validation
- `GenerationProgress` - Real-time progress indicator
- `OutputDisplay` - Platform-specific content cards
- `CopyButton` - One-click clipboard functionality
- `ExportModal` - Export options (.txt, .docx)
- `HistoryList` - Paginated generation history
- `UsageIndicator` - Credit usage display

**State Management:**
- Zustand stores for user, generation, and UI state
- React Query for server state and caching
- Local storage for draft content

**API Integration:**
- REST API calls via fetch with error handling
- WebSocket connection for real-time progress updates
- Automatic retry logic with exponential backoff

### 3.2 Backend Layer (Node.js + Express)

**Responsibilities:**
- API gateway and request routing
- Business logic orchestration
- Authentication and authorization
- Rate limiting and quota management
- Job queue management

**Services Architecture:**

#### Auth Service
**Endpoints:**
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - Session termination
- `POST /api/auth/refresh` - Token refresh
- `POST /api/auth/reset-password` - Password reset
- `POST /api/auth/verify-email` - Email verification

**Responsibilities:**
- JWT token generation and validation
- Password hashing (bcrypt, cost factor 12)
- OAuth integration (Google)
- Session management via Redis
- Email verification workflow

#### Content Service
**Endpoints:**
- `POST /api/content/extract` - Extract content from URL
- `POST /api/content/generate` - Generate multi-platform content
- `GET /api/content/status/:jobId` - Check generation status
- `POST /api/content/regenerate/:platform` - Regenerate single platform

**Responsibilities:**
- URL content extraction using Cheerio/Puppeteer
- Job creation and queue management
- Result aggregation from AI workers
- Content validation and sanitization

#### History Service
**Endpoints:**
- `GET /api/history` - List user's generations
- `GET /api/history/:id` - Get specific generation
- `DELETE /api/history/:id` - Delete generation
- `POST /api/history/:id/export` - Export generation

**Responsibilities:**
- CRUD operations for generation history
- Search and filtering
- Pagination
- Export file generation

#### Billing Service
**Endpoints:**
- `GET /api/billing/usage` - Get current usage
- `POST /api/billing/checkout` - Create Stripe checkout
- `POST /api/billing/portal` - Access billing portal
- `POST /api/billing/webhook` - Stripe webhook handler

**Responsibilities:**
- Usage tracking and quota enforcement
- Stripe integration for payments
- Subscription lifecycle management
- Invoice generation

### 3.3 AI Service Layer

**Responsibilities:**
- Content extraction from URLs
- AI-powered content generation
- Prompt management and optimization
- Multi-provider fallback handling

**Components:**

#### Content Extractor
**Technology:** Cheerio (static sites), Puppeteer (dynamic sites)

**Process:**
1. Fetch URL with user-agent headers
2. Parse HTML and identify main content
3. Remove navigation, ads, footers
4. Extract title, headings, paragraphs, lists
5. Preserve formatting (bold, italic, code blocks)
6. Return structured content object

**Fallback Strategy:**
- Try Cheerio first (fast, low cost)
- Fall back to Puppeteer for JavaScript-heavy sites
- Return error if extraction fails (user can paste raw text)

#### AI Generator (LangChain)
**Technology:** LangChain + OpenAI GPT-4

**Process:**
1. Receive extracted content and target audience
2. Create 5 parallel generation tasks
3. Use platform-specific prompt templates
4. Call OpenAI API with optimized parameters
5. Parse and validate outputs
6. Return structured results

**Prompt Templates:**

**LinkedIn Prompt:**
```
You are a LinkedIn content expert. Transform this blog post into an engaging LinkedIn post.
Requirements:
- 1,300-2,000 characters
- Professional yet conversational tone
- Story-driven format with personal insights
- Include 2-3 relevant hashtags
- Strong hook in first 2 lines
- Call-to-action at the end
Target Audience: {audience}
Blog Content: {content}
```

**Instagram Prompt:**
```
You are an Instagram content strategist. Create a carousel post from this blog.
Requirements:
- 5-8 slides with clear slide breaks
- Each slide: headline (5-7 words) + 2-3 bullet points
- Casual, engaging tone
- Visual-friendly formatting
- 5-10 relevant hashtags at the end
Target Audience: {audience}
Blog Content: {content}
```

**Twitter Prompt:**
```
You are a Twitter engagement expert. Create a thread from this blog post.
Requirements:
- 5-10 tweets, each under 280 characters
- Strong hook in first tweet
- Numbered format (1/10, 2/10, etc.)
- 2-3 hashtags total (not per tweet)
- Conversational, punchy tone
Target Audience: {audience}
Blog Content: {content}
```

**Newsletter Prompt:**
```
You are an email marketing expert. Adapt this blog for a newsletter.
Requirements:
- Email-friendly formatting with clear sections
- Engaging subject line suggestion
- Personal greeting tone
- 3-5 key takeaways highlighted
- Clear call-to-action
- 500-800 words
Target Audience: {audience}
Blog Content: {content}
```

**SEO Prompt:**
```
You are an SEO specialist. Create SEO metadata for this blog post.
Requirements:
- Meta title: 50-60 characters, include primary keyword
- Meta description: 150-160 characters, compelling and keyword-rich
- 3-5 focus keywords
- Optimized for click-through rate
Blog Content: {content}
```

---

## 4. Data Flow (Step-by-Step)

### 4.1 User Registration Flow
1. User submits email/password or clicks Google OAuth
2. Frontend sends POST to `/api/auth/register`
3. Backend validates input and checks for existing user
4. Password hashed with bcrypt (cost factor 12)
5. User record created in PostgreSQL
6. Verification email sent via SendGrid
7. JWT tokens generated (access + refresh)
8. Session stored in Redis (7-day TTL)
9. Response sent to frontend with tokens
10. Frontend stores tokens and redirects to dashboard

### 4.2 Content Generation Flow
1. User pastes URL or text in dashboard
2. Frontend validates input (length, format)
3. POST request to `/api/content/generate` with auth token
4. Backend validates JWT and checks user quota
5. If URL: Content extraction job queued
6. Extractor fetches and parses URL content
7. Extracted content passed to AI generator
8. Generator creates 5 parallel AI tasks
9. Each task uses platform-specific prompt template
10. OpenAI API called with optimized parameters
11. Responses parsed and validated
12. Results stored in PostgreSQL (generations table)
13. User quota decremented in database
14. WebSocket notification sent to frontend
15. Frontend displays all 5 platform outputs
16. User can copy, export, or regenerate

### 4.3 History Retrieval Flow
1. User navigates to history page
2. Frontend sends GET to `/api/history?page=1&limit=20`
3. Backend validates JWT token
4. Query PostgreSQL for user's generations
5. Apply pagination and sorting
6. Check Redis cache for recent queries
7. Return generation list with metadata
8. Frontend renders history cards
9. User clicks specific generation
10. Frontend fetches full details via `/api/history/:id`
11. Backend retrieves from database or cache
12. Full generation data returned and displayed

---

## 5. API Design (Endpoints with Request/Response Examples)

### 5.1 Authentication Endpoints

#### POST /api/auth/register
**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "name": "John Doe"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "usr_abc123",
      "email": "user@example.com",
      "name": "John Doe",
      "plan": "free",
      "creditsRemaining": 5
    },
    "tokens": {
      "accessToken": "eyJhbGc...",
      "refreshToken": "eyJhbGc...",
      "expiresIn": 3600
    }
  }
}
```

#### POST /api/auth/login
**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "usr_abc123",
      "email": "user@example.com",
      "name": "John Doe",
      "plan": "pro",
      "creditsRemaining": 150
    },
    "tokens": {
      "accessToken": "eyJhbGc...",
      "refreshToken": "eyJhbGc...",
      "expiresIn": 3600
    }
  }
}
```

### 5.2 Content Generation Endpoints

#### POST /api/content/generate
**Request:**
```json
{
  "input": {
    "type": "url",
    "content": "https://example.com/blog/my-post"
  },
  "options": {
    "targetAudience": "developers"
  }
}
```

**Response (202):**
```json
{
  "success": true,
  "data": {
    "jobId": "job_xyz789",
    "status": "processing",
    "estimatedTime": 45
  }
}
```

#### GET /api/content/status/:jobId
**Response (200):**
```json
{
  "success": true,
  "data": {
    "jobId": "job_xyz789",
    "status": "completed",
    "progress": 100,
    "result": {
      "generationId": "gen_def456",
      "platforms": {
        "linkedin": {
          "content": "Here's what I learned building...",
          "characterCount": 1450,
          "hashtags": ["#SaaS", "#ProductDevelopment"]
        },
        "instagram": {
          "slides": [
            {
              "slideNumber": 1,
              "headline": "5 Lessons from Building a SaaS",
              "content": "• Validate before you build\n• Talk to users daily\n• Ship fast, iterate faster"
            }
          ],
          "hashtags": ["#SaaS", "#Startup", "#Entrepreneurship"]
        },
        "twitter": {
          "thread": [
            {
              "tweetNumber": 1,
              "content": "I just spent 6 months building a SaaS. Here's what I learned 🧵 (1/8)"
            }
          ]
        },
        "newsletter": {
          "subject": "5 Hard-Earned Lessons from Building My First SaaS",
          "content": "Hey there,\n\nI wanted to share..."
        },
        "seo": {
          "title": "5 Lessons Learned Building a SaaS Product in 2026",
          "metaDescription": "Discover 5 critical lessons from building a SaaS product...",
          "keywords": ["saas development", "product building", "startup lessons"]
        }
      }
    }
  }
}
```

### 5.3 History Endpoints

#### GET /api/history
**Query Parameters:**
- `page` (default: 1)
- `limit` (default: 20, max: 100)
- `search` (optional)
- `sortBy` (default: createdAt)
- `order` (default: desc)

**Response (200):**
```json
{
  "success": true,
  "data": {
    "generations": [
      {
        "id": "gen_def456",
        "title": "5 Lessons from Building a SaaS",
        "sourceUrl": "https://example.com/blog/my-post",
        "createdAt": "2026-02-15T10:30:00Z",
        "platforms": ["linkedin", "instagram", "twitter", "newsletter", "seo"]
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 5,
      "totalItems": 98,
      "itemsPerPage": 20
    }
  }
}
```

#### GET /api/history/:id
**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "gen_def456",
    "title": "5 Lessons from Building a SaaS",
    "sourceUrl": "https://example.com/blog/my-post",
    "sourceType": "url",
    "targetAudience": "developers",
    "createdAt": "2026-02-15T10:30:00Z",
    "platforms": {
      "linkedin": { "content": "..." },
      "instagram": { "slides": [...] },
      "twitter": { "thread": [...] },
      "newsletter": { "subject": "...", "content": "..." },
      "seo": { "title": "...", "metaDescription": "...", "keywords": [...] }
    }
  }
}
```

#### DELETE /api/history/:id
**Response (200):**
```json
{
  "success": true,
  "message": "Generation deleted successfully"
}
```

### 5.4 Billing Endpoints

#### GET /api/billing/usage
**Response (200):**
```json
{
  "success": true,
  "data": {
    "plan": "pro",
    "billingPeriod": {
      "start": "2026-02-01T00:00:00Z",
      "end": "2026-03-01T00:00:00Z"
    },
    "usage": {
      "generationsUsed": 87,
      "generationsLimit": 200,
      "percentageUsed": 43.5
    },
    "overage": {
      "count": 0,
      "cost": 0
    }
  }
}
```

#### POST /api/billing/checkout
**Request:**
```json
{
  "plan": "pro",
  "interval": "monthly"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "checkoutUrl": "https://checkout.stripe.com/c/pay/cs_test_...",
    "sessionId": "cs_test_abc123"
  }
}
```

---

## 6. Database Schema (Tables with Fields)

### 6.1 Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255),
  name VARCHAR(255),
  oauth_provider VARCHAR(50),
  oauth_id VARCHAR(255),
  email_verified BOOLEAN DEFAULT FALSE,
  plan VARCHAR(50) DEFAULT 'free',
  stripe_customer_id VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  last_login_at TIMESTAMP,
  INDEX idx_email (email),
  INDEX idx_stripe_customer (stripe_customer_id)
);
```

### 6.2 Generations Table
```sql
CREATE TABLE generations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(500),
  source_type VARCHAR(20) NOT NULL,
  source_url TEXT,
  source_content TEXT,
  target_audience VARCHAR(100),
  platforms JSONB NOT NULL,
  status VARCHAR(50) DEFAULT 'completed',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_user_created (user_id, created_at DESC),
  INDEX idx_status (status)
);
```

**platforms JSONB structure:**
```json
{
  "linkedin": { "content": "...", "characterCount": 1450, "hashtags": [...] },
  "instagram": { "slides": [...], "hashtags": [...] },
  "twitter": { "thread": [...] },
  "newsletter": { "subject": "...", "content": "..." },
  "seo": { "title": "...", "metaDescription": "...", "keywords": [...] }
}
```

### 6.3 Subscriptions Table
```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  stripe_subscription_id VARCHAR(255) UNIQUE,
  plan VARCHAR(50) NOT NULL,
  status VARCHAR(50) NOT NULL,
  current_period_start TIMESTAMP NOT NULL,
  current_period_end TIMESTAMP NOT NULL,
  cancel_at_period_end BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_user (user_id),
  INDEX idx_stripe_subscription (stripe_subscription_id),
  INDEX idx_status (status)
);
```

### 6.4 Usage Table
```sql
CREATE TABLE usage (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  generation_id UUID REFERENCES generations(id) ON DELETE SET NULL,
  action_type VARCHAR(50) NOT NULL,
  credits_used INTEGER DEFAULT 1,
  created_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_user_created (user_id, created_at DESC),
  INDEX idx_action_type (action_type)
);
```

### 6.5 Sessions Table (Redis)
```
Key: session:{userId}
Value: {
  "userId": "usr_abc123",
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "expiresAt": 1708012800,
  "userAgent": "Mozilla/5.0...",
  "ipAddress": "192.168.1.1"
}
TTL: 604800 (7 days)
```

---

## 7. Prompt Engineering Strategy

### 7.1 Prompt Optimization Principles
- **Specificity:** Clear requirements for length, tone, format
- **Context:** Include target audience for tone adaptation
- **Examples:** Few-shot learning with high-quality examples
- **Constraints:** Explicit character/word limits per platform
- **Structure:** Consistent prompt template structure
- **Iteration:** A/B test prompts and track quality metrics

### 7.2 Token Optimization
**Goal:** Minimize API costs while maintaining quality

**Strategies:**
- Use GPT-4-turbo for cost efficiency (50% cheaper than GPT-4)
- Limit input content to 3,000 tokens (truncate if longer)
- Parallel generation instead of sequential (5 calls vs 1 large call)
- Cache common prompt templates
- Use lower temperature (0.7) for consistency
- Set max_tokens per platform to prevent over-generation

**Cost Calculation:**
- Input: ~1,500 tokens (prompt + content) × $0.01/1K = $0.015
- Output: ~2,000 tokens (all platforms) × $0.03/1K = $0.06
- Total per generation: ~$0.075
- With 200 generations/month (Pro plan): $15 cost vs $49 revenue = 69% margin

### 7.3 Quality Assurance
- Validate output format (JSON structure)
- Check character limits per platform
- Detect and filter inappropriate content
- Verify hashtag relevance
- Ensure factual consistency with source
- Track regeneration rate as quality metric

---

## 8. Rate Limiting Strategy

### 8.1 Rate Limit Tiers

**Anonymous Users (IP-based):**
- 10 requests/minute to public endpoints
- 100 requests/hour
- Prevents abuse and DDoS

**Authenticated Users (User-based):**
- Free tier: 5 generations/month, 20 API calls/minute
- Starter tier: 50 generations/month, 30 API calls/minute
- Pro tier: 200 generations/month, 60 API calls/minute
- Team tier: 1,000 generations/month, 100 API calls/minute

**Implementation:**
- Redis-based rate limiting with sliding window
- Return 429 status with Retry-After header
- Display remaining quota in response headers

### 8.2 Redis Rate Limit Keys
```
Key: ratelimit:{userId}:{endpoint}:{window}
Value: request_count
TTL: window_duration

Example:
Key: ratelimit:usr_abc123:generate:60
Value: 15
TTL: 60 seconds
```

### 8.3 Quota Management
**Monthly Generation Quota:**
- Tracked in usage table
- Reset on billing cycle start date
- Soft limit: Warning at 80% usage
- Hard limit: Block at 100% with upgrade prompt
- Overage: Allow with $0.50/generation charge (if enabled)

**Implementation:**
```javascript
async function checkQuota(userId) {
  const subscription = await getSubscription(userId);
  const usage = await getMonthlyUsage(userId);
  
  if (usage >= subscription.limit) {
    if (subscription.allowOverage) {
      await chargeOverage(userId);
      return { allowed: true, overage: true };
    }
    return { allowed: false, reason: 'quota_exceeded' };
  }
  
  return { allowed: true, remaining: subscription.limit - usage };
}
```

---

## 9. Caching Strategy

### 9.1 Cache Layers

**CDN Caching (Cloudflare):**
- Static assets: 1 year TTL
- Landing page: 1 hour TTL
- API responses: No cache (dynamic)

**Application Caching (Redis):**
- User sessions: 7 days TTL
- Generation results: 30 days TTL
- User profiles: 1 hour TTL
- Usage statistics: 5 minutes TTL

**Database Query Caching:**
- Frequently accessed generations
- User subscription details
- Usage summaries

### 9.2 Cache Keys Structure
```
user:{userId}:profile
user:{userId}:usage:monthly
generation:{generationId}
history:{userId}:page:{pageNum}
```

### 9.3 Cache Invalidation
**Strategies:**
- Time-based expiration (TTL)
- Event-based invalidation (on update/delete)
- Cache-aside pattern for reads
- Write-through for critical data

**Example:**
```javascript
async function getGeneration(generationId) {
  // Try cache first
  const cached = await redis.get(`generation:${generationId}`);
  if (cached) return JSON.parse(cached);
  
  // Cache miss: fetch from database
  const generation = await db.generations.findById(generationId);
  
  // Store in cache
  await redis.setex(
    `generation:${generationId}`,
    2592000, // 30 days
    JSON.stringify(generation)
  );
  
  return generation;
}
```

### 9.4 Content Extraction Caching
**Strategy:** Cache extracted content by URL hash
- Reduces redundant URL fetching
- 7-day TTL (content may change)
- Saves API costs and improves speed

```
Key: extract:{urlHash}
Value: { title, content, extractedAt }
TTL: 604800 (7 days)
```

---

## 10. Cost Optimization Strategy

### 10.1 AI API Cost Management

**Current Costs (GPT-4-turbo):**
- Input: $0.01 per 1K tokens
- Output: $0.03 per 1K tokens
- Average per generation: $0.075

**Optimization Tactics:**
1. **Prompt Compression:** Remove unnecessary words, use abbreviations
2. **Content Truncation:** Limit input to 3,000 tokens (keep key sections)
3. **Parallel Processing:** 5 separate calls vs 1 large call (better control)
4. **Model Selection:** Use GPT-4-turbo instead of GPT-4 (50% savings)
5. **Caching:** Cache extracted content to avoid re-processing
6. **Batch Processing:** Queue multiple requests during off-peak hours

**Fallback Strategy:**
- Primary: OpenAI GPT-4-turbo
- Fallback: Anthropic Claude (if OpenAI fails)
- Emergency: Queue for later processing

### 10.2 Infrastructure Cost Management

**Vercel (Frontend):**
- Free tier: 100GB bandwidth/month
- Pro tier: $20/month for production needs
- Estimated: $20-50/month

**AWS/GCP (Backend):**
- Container hosting: $50-150/month (2-4 instances)
- RDS PostgreSQL: $50-100/month (db.t3.medium)
- ElastiCache Redis: $30-60/month (cache.t3.micro)
- S3 Storage: $5-10/month
- Estimated: $135-320/month

**Total Infrastructure:** $155-370/month

### 10.3 Cost Per User Economics
**Pro Plan User ($49/month, 200 generations):**
- AI costs: 200 × $0.075 = $15
- Infrastructure (allocated): $2
- Payment processing (Stripe 2.9% + $0.30): $1.72
- Total cost: $18.72
- Gross profit: $30.28
- Gross margin: 62%

**Scaling Efficiency:**
- At 1,000 users: $49K revenue, $18.7K costs = 62% margin
- At 10,000 users: $490K revenue, $187K costs = 62% margin
- Infrastructure scales sub-linearly (economies of scale)

---

## 11. Security Architecture

### 11.1 Authentication & Authorization

**JWT Token Strategy:**
- Access token: 1-hour expiration, stored in memory
- Refresh token: 7-day expiration, stored in httpOnly cookie
- Token rotation on refresh
- Blacklist for revoked tokens (Redis)

**Password Security:**
- bcrypt hashing with cost factor 12
- Minimum requirements: 8 chars, 1 number, 1 special char
- Password reset via time-limited tokens (1-hour expiration)
- Rate limiting on auth endpoints (5 attempts/15 minutes)

**OAuth Integration:**
- Google OAuth 2.0
- State parameter for CSRF protection
- Secure token exchange
- Link OAuth accounts to existing users

### 11.2 API Security

**Request Validation:**
- Input sanitization (XSS prevention)
- SQL injection prevention (parameterized queries)
- CORS policy (whitelist allowed origins)
- Content Security Policy headers

**Rate Limiting:**
- IP-based for anonymous users
- User-based for authenticated users
- Exponential backoff for repeated violations
- Temporary IP bans for abuse

**Encryption:**
- TLS 1.3 for all data in transit
- Database encryption at rest (AWS RDS encryption)
- API keys encrypted in database
- Sensitive logs redacted

### 11.3 Data Privacy

**GDPR Compliance:**
- User consent for data processing
- Right to access (export user data)
- Right to deletion (account deletion with data purge)
- Data retention policies (90 days for free users)
- Privacy policy and terms of service

**Data Minimization:**
- Collect only necessary user information
- No tracking of generated content usage
- Anonymized analytics
- Secure deletion of expired data

### 11.4 Security Monitoring

**Threat Detection:**
- Failed login attempt monitoring
- Unusual API usage patterns
- Suspicious IP addresses
- SQL injection attempt detection
- XSS attempt detection

**Incident Response:**
- Automated alerts for security events
- IP blocking for confirmed threats
- User notification for account compromise
- Security audit logs (90-day retention)

---

## 12. Error Handling Strategy

### 12.1 Error Categories

**Client Errors (4xx):**
- 400 Bad Request: Invalid input format
- 401 Unauthorized: Missing or invalid token
- 403 Forbidden: Insufficient permissions or quota exceeded
- 404 Not Found: Resource doesn't exist
- 429 Too Many Requests: Rate limit exceeded

**Server Errors (5xx):**
- 500 Internal Server Error: Unexpected server error
- 502 Bad Gateway: Upstream service failure
- 503 Service Unavailable: Maintenance or overload
- 504 Gateway Timeout: Request timeout

### 12.2 Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "QUOTA_EXCEEDED",
    "message": "You've reached your monthly generation limit",
    "details": {
      "used": 50,
      "limit": 50,
      "resetDate": "2026-03-01T00:00:00Z"
    },
    "action": {
      "type": "upgrade",
      "url": "/settings/billing"
    }
  }
}
```

### 12.3 Error Handling Patterns

**Graceful Degradation:**
- If OpenAI fails, queue for retry
- If extraction fails, prompt for raw text input
- If Redis fails, continue without caching
- If email fails, log and retry later

**Retry Logic:**
- Exponential backoff for transient failures
- Maximum 3 retry attempts
- Circuit breaker for repeated failures
- Dead letter queue for permanent failures

**User-Facing Errors:**
- Clear, actionable error messages
- Avoid technical jargon
- Provide next steps or solutions
- Include support contact for critical errors

**Example:**
```
❌ Bad: "OpenAI API returned 429 status code"
✅ Good: "We're experiencing high demand. Your content is queued and will be ready in 2-3 minutes."
```

---

## 13. Logging & Monitoring

### 13.1 Logging Strategy

**Log Levels:**
- ERROR: System failures, exceptions
- WARN: Degraded performance, retry attempts
- INFO: Important business events (user registration, generation completed)
- DEBUG: Detailed diagnostic information (development only)

**Structured Logging:**
```json
{
  "timestamp": "2026-02-15T10:30:00Z",
  "level": "INFO",
  "service": "content-service",
  "event": "generation_completed",
  "userId": "usr_abc123",
  "generationId": "gen_def456",
  "duration": 42000,
  "platforms": ["linkedin", "instagram", "twitter", "newsletter", "seo"],
  "metadata": {
    "sourceType": "url",
    "targetAudience": "developers"
  }
}
```

**Log Storage:**
- CloudWatch Logs (AWS) or Cloud Logging (GCP)
- 30-day retention for standard logs
- 90-day retention for security logs
- Searchable and filterable

### 13.2 Monitoring Metrics

**Application Metrics:**
- Request rate (requests/second)
- Response time (p50, p95, p99)
- Error rate (errors/total requests)
- Generation success rate
- AI API latency
- Database query time

**Business Metrics:**
- New user registrations
- Daily/weekly/monthly active users
- Generations per user
- Free-to-paid conversion rate
- Churn rate
- Revenue (MRR, ARR)

**Infrastructure Metrics:**
- CPU utilization
- Memory usage
- Disk I/O
- Network throughput
- Database connections
- Redis cache hit rate

### 13.3 Alerting

**Critical Alerts (PagerDuty/Slack):**
- Error rate >5% for 5 minutes
- API response time >2 seconds (p95)
- Database connection failures
- Payment processing failures
- Security incidents

**Warning Alerts (Email):**
- Error rate >2% for 10 minutes
- Disk usage >80%
- Cache hit rate <70%
- High AI API costs (>$100/day)

**Monitoring Tools:**
- DataDog or New Relic (APM)
- Sentry (error tracking)
- Stripe Dashboard (payment monitoring)
- Custom dashboards (Grafana)

---

## 14. CI/CD Pipeline

### 14.1 Development Workflow

**Branching Strategy:**
- `main` - Production-ready code
- `develop` - Integration branch
- `feature/*` - Feature branches
- `hotfix/*` - Emergency fixes

**Pull Request Process:**
1. Create feature branch from `develop`
2. Write code with tests
3. Run local tests and linting
4. Create PR with description
5. Automated checks run (CI)
6. Code review by team member
7. Merge to `develop` after approval
8. Deploy to staging automatically
9. QA testing on staging
10. Merge to `main` for production

### 14.2 CI Pipeline (GitHub Actions)

**On Pull Request:**
```yaml
- Checkout code
- Install dependencies
- Run linter (ESLint)
- Run type checker (TypeScript)
- Run unit tests (Jest)
- Run integration tests
- Build application
- Security scan (Snyk)
- Code coverage report
```

**On Merge to Main:**
```yaml
- All CI checks
- Build Docker images
- Push to container registry
- Deploy to production (Vercel + AWS)
- Run smoke tests
- Notify team (Slack)
```

### 14.3 Deployment Strategy

**Frontend (Vercel):**
- Automatic deployment on push to `main`
- Preview deployments for PRs
- Instant rollback capability
- Edge caching and CDN
- Zero-downtime deployments

**Backend (AWS ECS / GCP Cloud Run):**
- Blue-green deployment strategy
- Health checks before traffic switch
- Automatic rollback on failure
- Gradual traffic shifting (10% → 50% → 100%)
- Database migrations run before deployment

**Database Migrations:**
- Versioned migration files (Knex.js or Prisma)
- Run migrations in separate job
- Backward-compatible changes only
- Rollback plan for each migration

### 14.4 Testing Strategy

**Unit Tests:**
- Test individual functions and components
- Mock external dependencies
- Target: >80% code coverage
- Run on every commit

**Integration Tests:**
- Test API endpoints end-to-end
- Use test database
- Test authentication flows
- Test generation workflows

**E2E Tests (Playwright):**
- Test critical user journeys
- Run on staging before production
- User registration and login
- Content generation flow
- Payment flow

**Load Tests (k6):**
- Simulate 1,000 concurrent users
- Test generation throughput
- Identify bottlenecks
- Run before major releases

---

## 15. Scaling Strategy

### 15.1 Horizontal Scaling

**Frontend:**
- Vercel handles auto-scaling
- Edge caching reduces origin load
- Static assets on CDN

**Backend:**
- Stateless services enable easy scaling
- Auto-scaling based on CPU (>70%) or request rate
- Load balancer distributes traffic
- Target: 2-10 instances based on load

**AI Workers:**
- Separate worker pool for AI processing
- Scale independently from API servers
- Queue-based architecture (Bull)
- Target: 5-20 workers based on queue depth

**Database:**
- Vertical scaling initially (larger instance)
- Read replicas for read-heavy queries
- Connection pooling (PgBouncer)
- Horizontal sharding if needed (future)

### 15.2 Performance Optimization

**Database Optimization:**
- Indexes on frequently queried columns
- Query optimization (EXPLAIN ANALYZE)
- Pagination for large result sets
- Materialized views for complex queries
- Regular VACUUM and ANALYZE

**API Optimization:**
- Response compression (gzip)
- Pagination for list endpoints
- Field selection (return only requested fields)
- Batch endpoints for multiple operations
- GraphQL for flexible queries (future)

**Caching Strategy:**
- Redis for session and frequently accessed data
- CDN for static assets
- Browser caching headers
- Cache warming for popular content

### 15.3 Capacity Planning

**Current Capacity (MVP):**
- 10,000 concurrent users
- 1,000 generations/hour
- 99.5% uptime

**6-Month Target:**
- 50,000 concurrent users
- 5,000 generations/hour
- 99.9% uptime

**12-Month Target:**
- 100,000 concurrent users
- 10,000 generations/hour
- 99.9% uptime

**Scaling Triggers:**
- CPU >70% for 10 minutes → Add instance
- Queue depth >100 jobs → Add AI worker
- Database connections >80% → Add read replica
- Response time >1 second (p95) → Investigate and optimize

---

## 16. Trade-offs & Design Decisions

### 16.1 Technology Choices

**Decision: Next.js over React SPA**
- ✅ Pros: Better SEO, faster initial load, server-side rendering
- ❌ Cons: More complex deployment, higher hosting costs
- Rationale: SEO critical for organic growth, worth the complexity

**Decision: PostgreSQL over MongoDB**
- ✅ Pros: ACID compliance, relational data, mature ecosystem
- ❌ Cons: Less flexible schema, harder to scale horizontally
- Rationale: Structured data (users, subscriptions) fits relational model

**Decision: OpenAI API over Self-Hosted Model**
- ✅ Pros: No infrastructure management, latest models, reliable
- ❌ Cons: Higher per-request cost, vendor lock-in, API limits
- Rationale: Faster time-to-market, focus on product not ML ops

**Decision: Microservices over Monolith**
- ✅ Pros: Independent scaling, clear boundaries, easier to maintain
- ❌ Cons: More complex deployment, network overhead, debugging harder
- Rationale: Enables independent scaling of AI workers and API servers

### 16.2 Architecture Trade-offs

**Decision: Async Job Queue for AI Processing**
- ✅ Pros: Non-blocking, better UX, handles spikes, retry logic
- ❌ Cons: More complex, requires WebSocket for real-time updates
- Rationale: AI generation takes 30-60 seconds, can't block HTTP request

**Decision: JWT over Session Cookies**
- ✅ Pros: Stateless, scales horizontally, works with mobile apps
- ❌ Cons: Can't revoke easily, larger payload, security concerns
- Rationale: Stateless auth enables easy scaling, use short expiration

**Decision: Redis for Caching over In-Memory**
- ✅ Pros: Shared across instances, persistent, rich data structures
- ❌ Cons: Network latency, additional cost, single point of failure
- Rationale: Necessary for multi-instance deployment and session sharing

### 16.3 Feature Trade-offs

**Decision: 5 Platforms (Not 10+)**
- ✅ Pros: Focused product, easier to maintain quality, faster MVP
- ❌ Cons: Limited appeal, competitors may offer more platforms
- Rationale: Focus on quality over quantity, expand based on demand

**Decision: No Direct Publishing (Phase 1)**
- ✅ Pros: Simpler MVP, avoid OAuth complexity, faster launch
- ❌ Cons: Extra step for users, less "magical" experience
- Rationale: Copy-paste is acceptable for MVP, add publishing later

**Decision: English Only (Phase 1)**
- ✅ Pros: Simpler prompts, easier QA, faster iteration
- ❌ Cons: Limits market size, excludes non-English creators
- Rationale: English market is large enough for validation, expand later

### 16.4 Cost vs. Quality Trade-offs

**Decision: GPT-4-turbo over GPT-3.5**
- ✅ Pros: Better quality, fewer regenerations, higher user satisfaction
- ❌ Cons: 3x more expensive, slower response time
- Rationale: Quality is competitive advantage, worth the cost

**Decision: Parallel Generation over Sequential**
- ✅ Pros: Faster (30s vs 150s), better UX, lower timeout risk
- ❌ Cons: 5x API calls, slightly higher cost, more complex error handling
- Rationale: Speed is critical for UX, cost difference is minimal

**Decision: 30-Day History for Free Users**
- ✅ Pros: Encourages upgrades, reduces storage costs
- ❌ Cons: May frustrate free users, potential churn
- Rationale: Incentivizes paid conversion, storage costs add up

---

## 17. Future Enhancements & Roadmap

### 17.1 Phase 2 (Months 4-6)

**Advanced Customization:**
- Custom tone presets (Professional, Casual, Technical, Inspirational)
- Length preferences per platform
- Brand voice training from user's historical content
- Custom prompt templates

**Team Features:**
- Workspace creation with team members
- Role-based access control (Admin, Editor, Viewer)
- Shared generation history
- Team usage analytics

**Integrations:**
- Buffer/Hootsuite scheduling
- Zapier integration
- Webhook support for custom workflows

### 17.2 Phase 3 (Months 7-12)

**Content Expansion:**
- YouTube video script generation
- Podcast show notes and timestamps
- Medium article reformatting
- Reddit post optimization

**Multi-Language Support:**
- Spanish, French, German, Portuguese
- Language detection and translation
- Localized tone adaptation

**Enterprise Features:**
- SSO (SAML)
- Custom AI model fine-tuning
- Dedicated account manager
- SLA guarantees
- On-premise deployment option

### 17.3 Phase 4 (Year 2+)

**AI Enhancements:**
- Predictive content performance scoring
- A/B test variant generation
- Automated hashtag research
- Trending topic integration
- Image suggestion and alt-text generation

**Platform Expansion:**
- TikTok script generation
- Pinterest pin descriptions
- Quora answer formatting
- Slack community posts

**Mobile Applications:**
- iOS native app
- Android native app
- Offline mode with sync
- Mobile-optimized generation interface

**White-Label Solution:**
- Rebrandable platform for agencies
- Custom domain support
- API-first architecture
- Revenue sharing model

---

## Appendix

### A. Technology Alternatives Considered

**Frontend Alternatives:**
- Remix: Great DX, but smaller ecosystem than Next.js
- SvelteKit: Faster, but less mature and smaller talent pool
- Vue/Nuxt: Good option, but React has larger ecosystem

**Backend Alternatives:**
- Python/FastAPI: Great for ML, but Node.js better for real-time features
- Go: Excellent performance, but smaller ecosystem and slower development
- Ruby on Rails: Rapid development, but slower runtime performance

**Database Alternatives:**
- MongoDB: Flexible schema, but ACID compliance important for billing
- MySQL: Mature and reliable, but PostgreSQL has better JSON support
- Supabase: Managed PostgreSQL with auth, good option for MVP

**AI Provider Alternatives:**
- Anthropic Claude: Excellent quality, good fallback option
- Google PaLM: Competitive pricing, but less proven for content generation
- Self-hosted (Llama 2): Lower cost at scale, but requires ML ops expertise

### B. Glossary

- **Generation:** A complete set of platform-optimized content outputs from a single blog input
- **Credit:** Unit of usage; 1 credit = 1 generation
- **Platform Output:** Content formatted for a specific social media platform
- **Job Queue:** Asynchronous task processing system for long-running operations
- **Rate Limiting:** Restricting the number of requests a user can make in a time window
- **Horizontal Scaling:** Adding more instances to handle increased load
- **Vertical Scaling:** Increasing resources (CPU, RAM) of existing instances

### C. References

**Technical Documentation:**
- Next.js Documentation: https://nextjs.org/docs
- OpenAI API Reference: https://platform.openai.com/docs
- LangChain Documentation: https://docs.langchain.com
- PostgreSQL Documentation: https://www.postgresql.org/docs
- Redis Documentation: https://redis.io/docs
- Stripe API Reference: https://stripe.com/docs/api

**Best Practices:**
- The Twelve-Factor App: https://12factor.net
- API Security Best Practices: OWASP API Security Top 10
- Database Design Patterns: Martin Fowler's Patterns of Enterprise Application Architecture
- Microservices Patterns: Chris Richardson's Microservices Patterns

**Industry Standards:**
- GDPR Compliance: https://gdpr.eu
- SOC 2 Compliance: AICPA SOC 2 Framework
- PCI DSS (for payment processing): https://www.pcisecuritystandards.org

### D. Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 15, 2026 | Engineering Team | Initial draft for review |

---

**Approval Sign-off:**

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Senior Architect | [Name] | [Date] | [Signature] |
| Engineering Lead | [Name] | [Date] | [Signature] |
| CTO | [Name] | [Date] | [Signature] |
| Product Manager | [Name] | [Date] | [Signature] |

---

**End of Technical Design Document**

*This document is confidential and intended for internal use only. Distribution outside the organization requires approval from the CTO.*
