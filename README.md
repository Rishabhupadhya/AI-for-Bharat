# Repurpose.ai

> **One Blog. Five Platforms. Zero Effort.**

AI-powered content repurposing platform that transforms blog posts into platform-optimized content for LinkedIn, Instagram, Twitter/X, newsletters, and SEO—all in under 60 seconds.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Solution](#solution)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Documentation](#documentation)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

---

## 🎯 Overview

**Repurpose.ai** is a SaaS platform designed for content creators, indie hackers, startup founders, and marketing teams who want to maximize their content ROI without spending hours on manual adaptation.

### The Problem

Content creators spend **5-10 hours per week** manually repurposing blog content for different social media platforms. Each platform requires unique formatting, tone, and structure, making content distribution time-consuming and inconsistent.

### Our Solution

Repurpose.ai automates the entire content adaptation process using AI, generating platform-specific content in under 60 seconds while maintaining brand voice and message consistency across all channels.

---

## 🚀 Key Features

### Core Features (MVP)

- **🔗 Smart Content Input**
  - Paste blog URL for automatic content extraction
  - Or paste raw text/markdown (up to 10,000 words)
  - Optional target audience specification

- **🤖 AI-Powered Multi-Platform Generation**
  - LinkedIn post (1,300-2,000 characters, story-driven)
  - Instagram carousel (5-8 slides with headlines)
  - Twitter/X thread (5-10 tweets, 280 chars each)
  - Newsletter version (email-optimized format)
  - SEO package (meta title, description, keywords)

- **💾 Generation History**
  - Access past generations anytime
  - Search and filter by date or title
  - Re-export previous content

- **📤 Export Options**
  - Copy to clipboard (one-click)
  - Export as .txt file
  - Export as .docx file

- **👤 User Management**
  - Email/password authentication
  - Google OAuth integration
  - Usage tracking and quota management

### Coming Soon (Phase 2)

- Advanced customization (tone presets, length control)
- Team collaboration features
- API access for developers
- Platform integrations (Buffer, Hootsuite)
- Multi-language support

---

## 🛠 Tech Stack

### Frontend
- **Framework:** Next.js 14+ (App Router)
- **Language:** TypeScript
- **Styling:** TailwindCSS
- **State Management:** Zustand
- **Data Fetching:** React Query

### Backend
- **Runtime:** Node.js 20+ LTS
- **Framework:** Express.js
- **Language:** TypeScript
- **Database:** PostgreSQL 15+
- **Cache:** Redis
- **Job Queue:** Bull

### AI Layer
- **Primary:** OpenAI GPT-4-turbo
- **Abstraction:** LangChain
- **Fallback:** Anthropic Claude

### Infrastructure
- **Frontend Hosting:** Vercel
- **Backend Hosting:** AWS ECS / GCP Cloud Run
- **Database:** AWS RDS / Supabase
- **Cache:** AWS ElastiCache / Upstash
- **Storage:** AWS S3
- **CDN:** Cloudflare

### DevOps
- **CI/CD:** GitHub Actions
- **Containerization:** Docker
- **IaC:** Terraform
- **Monitoring:** DataDog / New Relic
- **Error Tracking:** Sentry

---

## 🏗 Architecture

```
┌─────────────┐
│   Browser   │
│  (Next.js)  │
└──────┬──────┘
       │
┌──────▼──────┐
│ Cloudflare  │
│  CDN + WAF  │
└──────┬──────┘
       │
       ├──────────────┬──────────────┐
       │              │              │
┌──────▼──────┐ ┌────▼─────┐ ┌─────▼─────┐
│   Next.js   │ │ Express  │ │   Redis   │
│   Frontend  │ │ Backend  │ │   Cache   │
└─────────────┘ └────┬─────┘ └───────────┘
                     │
              ┌──────┼──────┐
              │      │      │
         ┌────▼──┐ ┌▼────┐ ┌▼──────┐
         │ Bull  │ │ DB  │ │OpenAI │
         │ Queue │ │ PG  │ │  API  │
         └───────┘ └─────┘ └───────┘
```

**Key Architecture Principles:**
- Microservices architecture for independent scaling
- Async job processing for AI operations
- Multi-layer caching for performance
- Defense-in-depth security
- Observability-first design

For detailed architecture, see [design.md](./design.md)

---

## 🚦 Getting Started

### Prerequisites

- Node.js 20+ LTS
- PostgreSQL 15+
- Redis 7+
- OpenAI API key
- Stripe account (for payments)

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/repurpose-ai.git
   cd repurpose-ai
   ```

2. **Install dependencies**
   ```bash
   # Install frontend dependencies
   cd frontend
   npm install

   # Install backend dependencies
   cd ../backend
   npm install
   ```

3. **Set up environment variables**

   **Frontend (.env.local):**
   ```env
   NEXT_PUBLIC_API_URL=http://localhost:3001
   NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
   ```

   **Backend (.env):**
   ```env
   # Server
   PORT=3001
   NODE_ENV=development

   # Database
   DATABASE_URL=postgresql://user:password@localhost:5432/repurpose_ai

   # Redis
   REDIS_URL=redis://localhost:6379

   # JWT
   JWT_SECRET=your-secret-key-here
   JWT_EXPIRES_IN=1h
   REFRESH_TOKEN_SECRET=your-refresh-secret-here
   REFRESH_TOKEN_EXPIRES_IN=7d

   # OpenAI
   OPENAI_API_KEY=sk-...

   # Stripe
   STRIPE_SECRET_KEY=sk_test_...
   STRIPE_WEBHOOK_SECRET=whsec_...

   # Email (SendGrid)
   SENDGRID_API_KEY=SG...
   FROM_EMAIL=noreply@repurpose.ai
   ```

4. **Set up the database**
   ```bash
   cd backend
   npm run db:migrate
   npm run db:seed # Optional: seed with sample data
   ```

5. **Start the development servers**

   **Terminal 1 - Backend:**
   ```bash
   cd backend
   npm run dev
   ```

   **Terminal 2 - Frontend:**
   ```bash
   cd frontend
   npm run dev
   ```

6. **Open your browser**
   ```
   http://localhost:3000
   ```

### Running Tests

```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run E2E tests
npm run test:e2e
```

---

## 📚 Documentation

This repository contains comprehensive documentation:

- **[requirements.md](./requirements.md)** - Product Requirements Document (PRD)
  - Product overview and goals
  - Target users and personas
  - Functional and non-functional requirements
  - User flows and success metrics
  - Monetization strategy

- **[design.md](./design.md)** - Technical Design Document
  - System architecture
  - Component breakdown
  - API design and database schema
  - Security and scaling strategies
  - Cost optimization

- **[pitch-deck.md](./pitch-deck.md)** - Investor Pitch Deck
  - Problem and market opportunity
  - Solution and product demo
  - Business model and traction
  - Roadmap and vision

---

## 🗺 Roadmap

### Q1 2026 (Current) - Foundation ✅
- [x] MVP launch
- [x] Core 5-platform generation
- [ ] Paid tier launch
- [ ] Stripe integration
- **Target:** 500 paid users, $15K MRR

### Q2 2026 - Enhancement
- [ ] Advanced customization (tone presets)
- [ ] Generation history improvements
- [ ] Mobile-responsive optimization
- [ ] Content quality improvements
- **Target:** 1,500 paid users, $45K MRR

### Q3 2026 - Scale
- [ ] Team collaboration features
- [ ] API access for developers
- [ ] Bulk generation capabilities
- [ ] Buffer/Hootsuite integration
- **Target:** 4,000 paid users, $120K MRR

### Q4 2026 - Expansion
- [ ] Multi-language support (8 languages)
- [ ] YouTube script generation
- [ ] Podcast show notes
- [ ] Enterprise features (SSO)
- **Target:** 8,000 paid users, $240K MRR

---

## 💰 Pricing

### Free Tier
- 5 generations per month
- All 5 platforms
- 30-day history
- Export to .txt and .docx

### Starter - $19/month
- 50 generations per month
- All 5 platforms
- Unlimited history
- Priority email support

### Pro - $49/month
- 200 generations per month
- All 5 platforms
- Unlimited history
- Custom tone (coming soon)
- API access (coming soon)
- Priority support

### Team - $149/month
- 1,000 generations per month
- Up to 5 team members
- Workspace features
- Priority support
- Custom integrations

---

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Code Style

- Follow the existing code style
- Run `npm run lint` before committing
- Write tests for new features
- Update documentation as needed

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📞 Contact

- **Website:** [repurpose.ai](https://repurpose.ai)
- **Email:** hello@repurpose.ai
- **Twitter:** [@RepurposeAI](https://twitter.com/RepurposeAI)
- **LinkedIn:** [Repurpose.ai](https://linkedin.com/company/repurpose-ai)

---

## 🙏 Acknowledgments

- OpenAI for GPT-4 API
- Vercel for hosting platform
- All our beta users for valuable feedback
- Open source community for amazing tools

---

## 📊 Project Status

- **Status:** Active Development
- **Version:** 1.0.0 (MVP)
- **Last Updated:** February 15, 2026
- **Maintainers:** [@yourusername](https://github.com/yourusername)

---

**Made with ❤️ by the Repurpose.ai team**

*Empowering creators to maximize their content reach without the manual work.*
