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
