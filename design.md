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
