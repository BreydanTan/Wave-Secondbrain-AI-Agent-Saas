# Wave AI Project Technical Specification (English Version)

> Complete Technical Specification for Second-Brain AI Agent SaaS Platform

**Generated**: 2025-11-16
**Version**: v1.0
**Audience**: Developers, Product Managers, Technical Users

---

## 📖 Table of Contents

1. [Project Overview](#1-project-overview)
2. [Technical Architecture](#2-technical-architecture)
3. [Database Design](#3-database-design)
4. [Core Business Flows](#4-core-business-flows)
5. [Frontend-Backend Interaction](#5-frontend-backend-interaction)
6. [API Documentation](#6-api-documentation)
7. [Environment Configuration](#7-environment-configuration)
8. [Development Guide](#8-development-guide)
9. [Deployment Guide](#9-deployment-guide)
10. [Appendix](#10-appendix)

---

## 1. Project Overview

### 1.1 Introduction

Wave AI is a **"Second Brain" AI Assistant Platform** that provides:

- **Intelligent Chat**: Conversation with multiple top AI models (Claude, GPT-4, Gemini)
- **Note Management**: AI-powered note creation, search, and organization
- **Web Search**: Real-time information retrieval beyond AI training data
- **Content Extraction**: Extract and summarize content from any webpage

### 1.2 Key Features

| Feature | Description | Use Case |
|---------|-------------|----------|
| **AI Chat** | Multi-model conversation with streaming | Daily consultation, learning |
| **Note System** | Create, edit, search notes | Knowledge management |
| **AI Tool Chain** | AI proactively calls tools | Task automation |
| **Web Search** | Real-time information | News, research |
| **Subscription** | Free/Plus/Premium plans | Pay-as-you-go |

### 1.3 Tech Highlights

1. **Unified Multi-Model Interface**: Seamless switching between Claude, Grok, GPT-4, Gemini via AI Gateway
2. **AI Tool Calling**: AI can proactively call notes, search tools
3. **Streaming Response**: Real-time display of AI-generated content
4. **Stripe Integration**: Complete payment flow with webhook sync
5. **Modern Stack**: Next.js 15 + React 19 + TypeScript 5

### 1.4 Use Cases

- **Personal Knowledge Management**: Researchers, students, creators
- **Enterprise Tools**: Team collaboration, knowledge base
- **SaaS Learning**: Full-stack development, AI integration, payment systems

---

## 2. Technical Architecture

### 2.1 Architecture Diagram

```
                    ┌─────────────────────────────────┐
                    │      User Browser               │
                    │  ┌───────────┐  ┌────────────┐ │
                    │  │ React 19  │  │ Tailwind  │ │
                    │  │ Frontend  │  │    CSS     │ │
                    │  └───────────┘  └────────────┘ │
                    └───────────────┬─────────────────┘
                                    │ HTTP/WebSocket
                    ┌───────────────▼─────────────────┐
                    │      Next.js 15 App Router      │
                    │ ┌─────────────────────────────┐ │
                    │ │  Server Components (SSR)    │ │
                    │ └─────────────────────────────┘ │
                    │ ┌─────────────────────────────┐ │
                    │ │  API Routes (Hono)          │ │
                    │ │  - /api/chat                │ │
                    │ │  - /api/note                │ │
                    │ │  - /api/subscription        │ │
                    │ └──────────┬──────────────────┘ │
                    └────────────┼────────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
┌───────────────┐      ┌──────────────────┐    ┌─────────────────┐
│  BetterAuth   │      │  AI SDK (Vercel) │    │  Prisma ORM     │
│  Auth System  │      │  - AI Gateway    │    │  Data Layer     │
│               │      │  - Streaming     │    │                 │
└───────┬───────┘      └────────┬─────────┘    └────────┬────────┘
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐      ┌──────────────────┐    ┌─────────────────┐
│  Stripe API   │      │  External APIs   │    │  PostgreSQL     │
│  Payment      │      │  - Tavily Search │    │  Database       │
└───────────────┘      │  - Claude API    │    └─────────────────┘
                       │  - OpenAI API    │
                       └──────────────────┘
```

### 2.2 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| **Next.js** | 15.5.3 | Full-stack framework |
| **React** | 19.1.0 | UI library |
| **TypeScript** | 5+ | Type system |
| **Hono** | 4.9.4 | API framework |
| **Prisma** | 6.15.0 | ORM |
| **BetterAuth** | 1.3.7 | Authentication |
| **AI SDK** | 5.0.45 | AI integration |
| **Tailwind CSS** | 4 | Styling |

---

## 3. Database Design

### 3.1 ER Diagram

```
User
├── sessions (1:N) → Session
├── accounts (1:N) → Account
├── notes (1:N) → Note
├── Chat (1:N) → Chat
│   └── messages (1:N) → Message
└── subscriptions (1:N) → Subscription
```

### 3.2 Core Tables

#### User Table

| Field | Type | Description |
|-------|------|-------------|
| id | String (PK) | User ID |
| email | String (unique) | Email |
| name | String | Username |
| stripeCustomerId | String? | Stripe customer ID |

#### Note Table

| Field | Type | Description |
|-------|------|-------------|
| id | UUID (PK) | Note ID |
| title | String | Title |
| content | String | Content (Markdown) |
| userId | String (FK) | Owner |

#### Subscription Table

| Field | Type | Description |
|-------|------|-------------|
| plan | Enum | free/plus/premium |
| stripeSubscriptionId | String? | Stripe subscription ID |
| periodStart | DateTime? | Billing period start |

---

## 4. Core Business Flows

### 4.1 User Registration Flow

```
User fills registration form
  ↓
Frontend validation (React Hook Form + Zod)
  ↓
Call authClient.signUp.email()
  ↓
BetterAuth creates User record
  ↓
Stripe plugin creates Customer
  ↓
createDefaultSubscription() callback
  ↓
Create Subscription record (plan = "free")
  ↓
Create Account & Session
  ↓
Auto login and redirect to /home
```

### 4.2 AI Chat Flow

```
User inputs message
  ↓
useChat hook processes
  ↓
POST /api/chat
  ↓
Verify authentication
  ↓
Check generation limit
  ↓
Find or create Chat record
  ↓
Save user message
  ↓
Call AI streamText API
  ↓
AI decides whether to call tools
  ├─ Call tool → Execute → Return result
  └─ No tool → Generate text directly
  ↓
Stream response to frontend (SSE)
  ↓
onFinish: Save all AI messages
  ↓
Display AI response in real-time
```

---

## 5. Frontend-Backend Interaction

### 5.1 API Calling (Hono RPC)

**Backend**:
```typescript
const app = new Hono();
app.get('/note/:id', getAuthUser, async (c) => {
  return c.json({ success: true, data: note });
});

export type AppType = typeof app;
```

**Frontend**:
```typescript
import { api } from "@/lib/hono/hono-rpc";

const response = await api.note[":id"].$get({
  param: { id: "note-123" }
});
// ✓ Full type inference
```

### 5.2 Data Fetching (React Query)

```typescript
// Query
const { data, isLoading } = useNotes(1, 20);

// Mutation
const { mutate } = useCreateNote();
mutate({ title: "New Note", content: "" });
```

---

## 6. API Documentation

### 6.1 Authentication API

#### POST /api/auth/sign-up/email

**Request**:
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}
```

**Response**:
```json
{
  "user": {
    "id": "user_xxx",
    "email": "john@example.com"
  },
  "session": {
    "token": "...",
    "expiresAt": "2025-12-16T00:00:00Z"
  }
}
```

### 6.2 Chat API

#### POST /api/chat

**Description**: Send message, get AI streaming response

**Authentication**: Required (Cookie)

**Request**:
```json
{
  "id": "chat-uuid-123",
  "message": {
    "role": "user",
    "parts": [
      { "type": "text", "text": "Hello" }
    ]
  },
  "selectedModelId": "anthropic/claude-sonnet-4"
}
```

**Response**: Server-Sent Events (SSE) stream

#### GET /api/chat

**Response**:
```json
{
  "success": true,
  "data": [
    {
      "id": "chat-123",
      "title": "Discussing AI Tools",
      "createdAt": "2025-11-16T00:00:00Z"
    }
  ]
}
```

### 6.3 Notes API

#### POST /api/note/create

**Request**:
```json
{
  "title": "My Note",
  "content": "Content..."
}
```

#### GET /api/note/all

**Query**:
- `page`: Page number (default 1)
- `limit`: Items per page (default 20)

**Response**:
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "total": 45,
    "page": 1,
    "limit": 20,
    "totalPages": 3
  }
}
```

### 6.4 Subscription API

#### POST /api/subscription/upgrade

**Request**:
```json
{
  "plan": "plus",
  "callbackUrl": "https://app.example.com/billing"
}
```

**Response**:
```json
{
  "success": true,
  "checkoutUrl": "https://checkout.stripe.com/..."
}
```

---

## 7. Environment Configuration

### 7.1 Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | ✓ | PostgreSQL connection pool URL |
| `DIRECT_URL` | ✓ | PostgreSQL direct URL |
| `BETTER_AUTH_SECRET` | ✓ | Auth secret key |
| `STRIPE_SECRET_KEY` | ✓ | Stripe secret key |
| `GOOGLE_GENERATIVE_AI_API_KEY` | ✓ | Google AI key |
| `TAVILY_API_KEY` | ✓ | Tavily search key |

### 7.2 Obtaining API Keys

**PostgreSQL (Supabase)**:
1. Visit https://supabase.com
2. Create project
3. Get connection strings from Settings → Database

**Stripe**:
1. Register at https://stripe.com
2. Dashboard → Developers → API keys
3. Create products and get Price IDs

**Google AI**:
1. Visit https://aistudio.google.com/app/apikey
2. Create API key

### 7.3 Local Development

```bash
# Clone repo
git clone <repository>
cd Wave-Secondbrain-AI-Agent-Saas

# Install dependencies
npm install

# Setup env
cp .env.example .env.local
# Edit .env.local with real values

# Initialize database
npx prisma generate --no-engine
npx prisma migrate dev --name init

# Start dev server
npm run dev
```

---

## 8. Development Guide

### 8.1 Project Structure

```
app/
├── (routes)/          # Route groups
├── api/               # API routes
└── actions/           # Server Actions

components/            # UI components
lib/                   # Core libraries
features/              # React Query hooks
hooks/                 # Custom Hooks
prisma/               # Database schema
```

### 8.2 Common Tasks

**Add new model**:
1. Edit `prisma/schema.prisma`
2. Run `npx prisma migrate dev`

**Create API endpoint**:
1. Create file in `app/api/[[...route]]/`
2. Register in main `route.ts`

**Add new page**:
1. Create file in `app/(routes)/`
2. Create React Query hook in `features/`

---

## 9. Deployment Guide

### 9.1 Vercel Deployment

1. Connect GitHub repository
2. Add environment variables in Vercel
3. Deploy
4. Configure Stripe webhook
5. Add custom domain (optional)

### 9.2 Troubleshooting

**Build fails**:
- Check dependencies in package.json

**Database timeout**:
- Verify DATABASE_URL uses connection pooler

**Webhook fails**:
- Check STRIPE_WEBHOOK_SECRET matches

---

## 10. Appendix

### 10.1 Glossary

| Term | Definition |
|------|------------|
| **SSR** | Server-Side Rendering |
| **API** | Application Programming Interface |
| **ORM** | Object-Relational Mapping |
| **Webhook** | HTTP callback |
| **Streaming** | Real-time data transmission |

### 10.2 Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start dev server |
| `npm run build` | Build production |
| `npx prisma studio` | Open database GUI |

### 10.3 Resources

- Next.js: https://nextjs.org/docs
- Prisma: https://prisma.io/docs
- Hono: https://hono.dev
- AI SDK: https://sdk.vercel.ai/docs

---

**© 2025 Wave AI Project Documentation**
