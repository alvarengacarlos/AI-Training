# ADR 002: Technology Stack Selection

## Status
Accepted

## Date
2025-10-16

## Context

We need to select a technology stack for the Muay Thai Gym Management System that:

- **Supports modular monolith architecture** (ADR-001)
- **Fits $100/month budget** including hosting and operational costs
- **Enables rapid development** for 15-week MVP timeline
- **Works with 1-3 developer team** (common skills, minimal learning curve)
- **Provides excellent performance** (< 3s page load, 100 concurrent users)
- **Supports LGPD compliance** with encryption and security features
- **Uses open-source technologies** to minimize licensing costs (CONSTRAINT-024)
- **Works in Brazilian cloud regions** (low latency)

### Technology Selection Criteria

1. **Developer Productivity**: Mature frameworks with good DX
2. **Community Support**: Large community, extensive documentation, rich ecosystem
3. **Performance**: Efficient runtime for budget constraints
4. **Type Safety**: Catch errors at compile time
5. **Cost**: Free/open-source with minimal hosting requirements
6. **Hire-ability**: Common skills in Brazilian developer market
7. **Maintenance**: Long-term stability and support

## Decision

### Full Stack: Next.js (React) with TypeScript

**We will build the system using Next.js 14+ (App Router) with TypeScript, deployed as a full-stack application.**

### Technology Stack Overview

```
┌─────────────────────────────────────────────────────────┐
│                    FRONTEND LAYER                       │
├─────────────────────────────────────────────────────────┤
│  Framework:    Next.js 14+ (App Router)                │
│  UI Library:   React 18+                                │
│  Language:     TypeScript 5+                            │
│  Styling:      Tailwind CSS                             │
│  UI Components: Shadcn/UI (Radix UI + Tailwind)        │
│  Forms:        React Hook Form + Zod                    │
│  State:        React Context + TanStack Query           │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    BACKEND LAYER                        │
├─────────────────────────────────────────────────────────┤
│  Runtime:      Node.js 20 LTS                           │
│  Framework:    Next.js API Routes (App Router)          │
│  Language:     TypeScript 5+                            │
│  ORM:          Prisma 5+                                │
│  Validation:   Zod                                      │
│  Auth:         NextAuth.js (Auth.js)                    │
│  Email:        Nodemailer (system emails only)          │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    DATABASE LAYER                       │
├─────────────────────────────────────────────────────────┤
│  Database:     PostgreSQL 15+                           │
│  Hosting:      Neon (serverless Postgres) or Supabase  │
│  Encryption:   pgcrypto extension                       │
│  Search:       PostgreSQL full-text search              │
│  Cache:        Next.js built-in caching                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              INFRASTRUCTURE & OPERATIONS                │
├─────────────────────────────────────────────────────────┤
│  Hosting:      Vercel (Next.js optimized)               │
│  Database:     Neon (serverless Postgres) Free Tier     │
│  CDN:          Vercel Edge Network (included)           │
│  Monitoring:   Vercel Analytics + Sentry (free tier)    │
│  Logging:      Vercel Logs + Custom aggregation         │
│  Backups:      Neon automated backups (included)        │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              EXTERNAL SERVICES (OPTIONAL)               │
├─────────────────────────────────────────────────────────┤
│  SMS:          Twilio (pay-per-use, nice-to-have)       │
│  WhatsApp:     Twilio WhatsApp API (nice-to-have)       │
└─────────────────────────────────────────────────────────┘
```

## Detailed Technology Justification

### Frontend: Next.js + React + TypeScript

**Why Next.js?**

1. **Full-Stack Framework**: Single codebase for frontend and backend (API routes)
2. **Server Components**: Reduces JavaScript sent to client, improves performance
3. **Built-in Optimization**: Image optimization, font optimization, code splitting
4. **File-based Routing**: Intuitive routing with App Router
5. **Excellent DX**: Fast Refresh, TypeScript support, great tooling
6. **Production Ready**: Used by enterprises, proven at scale
7. **Cost Effective**: Deploys well to Vercel (free tier available)
8. **Brazilian Market**: Widely adopted, easy to hire developers

**Why React?**

1. **Most Popular**: Largest ecosystem, most developers know React
2. **Component Model**: Reusable UI components
3. **Great Libraries**: Rich ecosystem (forms, tables, charts, etc.)
4. **Hire-ability**: Abundant React developers in Brazil

**Why TypeScript?**

1. **Type Safety**: Catch errors at compile time, not runtime
2. **Better IDE Support**: Autocomplete, refactoring, inline documentation
3. **Maintainability**: Self-documenting code, easier refactoring
4. **Industry Standard**: Expected in modern web development
5. **Prisma Integration**: Type-safe database queries

### Styling: Tailwind CSS + Shadcn/UI

**Why Tailwind CSS?**

1. **Utility-First**: Rapid UI development
2. **Responsive Design**: Mobile-first approach built-in
3. **Consistent Design**: Design system through configuration
4. **Small Bundle**: Purges unused CSS
5. **Developer Productivity**: No context switching between files

**Why Shadcn/UI?**

1. **Copy-Paste Components**: Own the code, no package dependency
2. **Built on Radix UI**: Accessible, unstyled components
3. **Tailwind Integration**: Styled with Tailwind
4. **Customizable**: Easy to modify to match brand
5. **Free**: No licensing costs

### Backend: Next.js API Routes + Prisma

**Why Next.js API Routes?**

1. **Same Framework**: No separate backend framework needed
2. **TypeScript**: Shared types between frontend and backend
3. **Serverless Compatible**: Works well with Vercel serverless functions
4. **Simple Deployment**: Single deployment for full application
5. **Co-located Code**: API routes near related components

**Why Prisma ORM?**

1. **Type Safety**: Auto-generated TypeScript types from schema
2. **Developer Experience**: Intuitive API, excellent documentation
3. **Migrations**: Built-in schema migration system
4. **Multi-Tenancy**: Easy to implement row-level filtering
5. **Prisma Studio**: Visual database browser for development
6. **Performance**: Efficient query generation
7. **PostgreSQL Support**: First-class support for PostgreSQL

**Why NodeAuth.js (NextAuth)?**

1. **Next.js Integration**: Built for Next.js
2. **Flexible**: Supports credentials, OAuth, JWT, sessions
3. **Secure**: Industry-standard security practices
4. **Session Management**: Built-in session handling
5. **Free**: Open-source, no licensing costs

### Database: PostgreSQL (Neon or Supabase)

**Why PostgreSQL?**

1. **ACID Compliance**: Critical for payment data integrity
2. **Relational**: Perfect fit for structured gym management data
3. **JSON Support**: Flexible for semi-structured data if needed
4. **Full-Text Search**: Built-in search capabilities
5. **Encryption**: pgcrypto extension for LGPD compliance
6. **Mature**: 25+ years of development, extremely stable
7. **Performance**: Excellent query optimization
8. **Open Source**: No licensing costs

**Why Neon (Primary Choice)?**

1. **Serverless**: Auto-scaling, pay for what you use
2. **Generous Free Tier**: 0.5GB storage, 3GB data transfer/month (sufficient for MVP)
3. **Fast**: Low latency from São Paulo region (via AWS)
4. **Branching**: Database branching for testing
5. **Automated Backups**: Point-in-time recovery included
6. **PostgreSQL Compatible**: Standard PostgreSQL, no vendor lock-in
7. **Cost Effective**: Can stay on free tier or pay $19/month for Pro

**Alternative: Supabase**

1. **Also Serverless PostgreSQL**: Similar benefits to Neon
2. **Free Tier**: 500MB database, 1GB file storage, 2GB bandwidth
3. **Built-in Features**: Auth, storage, real-time (if needed later)
4. **Brazilian Developers Familiar**: Popular in Brazilian market

### Hosting: Vercel

**Why Vercel?**

1. **Next.js Optimized**: Created by same team, perfect integration
2. **Generous Free Tier**:
   - 100GB bandwidth/month
   - Unlimited serverless function invocations
   - Auto SSL
   - Global CDN
   - Preview deployments
3. **Zero Configuration**: Deploy with `git push`
4. **Edge Network**: Low latency globally including Brazil
5. **Automatic Scaling**: Handles traffic spikes
6. **Built-in Monitoring**: Analytics and Web Vitals
7. **Preview URLs**: Every PR gets preview URL
8. **Cost**: Free tier sufficient for MVP, Pro $20/month if needed

**Cost Calculation**:
- Vercel Free Tier: $0 (sufficient for 100 concurrent users)
- Vercel Pro (if needed): $20/month
- Neon Free Tier: $0 (sufficient for MVP)
- Neon Pro (if needed): $19/month
- **Total: $0-$39/month** (well under $100 budget)

## Alternatives Considered

### Alternative 1: Python Django + PostgreSQL

**Pros**:
- Mature framework with admin interface
- Strong ORM (Django ORM)
- Good for data-heavy applications
- Python popular in Brazil

**Cons**:
- Separate frontend needed (React, Vue)
- More complex deployment (frontend + backend)
- Slower development (two codebases)
- Higher hosting costs (needs separate frontend hosting)
- Python runtime more expensive than Node.js serverless

**Verdict**: REJECTED - Increased complexity and cost

### Alternative 2: Ruby on Rails + PostgreSQL

**Pros**:
- Convention over configuration
- Rapid development (scaffolding)
- Mature ecosystem
- Good for CRUD applications

**Cons**:
- Smaller community than React/Node.js
- Less common in Brazilian market
- Performance limitations for high concurrency
- Separate frontend needed
- Higher hosting costs

**Verdict**: REJECTED - Smaller ecosystem, deployment complexity

### Alternative 3: Laravel (PHP) + Vue.js

**Pros**:
- Popular in Brazilian market
- Laravel has Livewire/Inertia for full-stack
- Good ecosystem (Laravel Forge, Vapor)
- Mature framework

**Cons**:
- PHP hosting more expensive than Node.js serverless
- Less modern than React/Next.js ecosystem
- Smaller TypeScript support
- Frontend/backend still somewhat separate

**Verdict**: REJECTED - Less optimal for serverless deployment

### Alternative 4: Remix (React) + PostgreSQL

**Pros**:
- Modern React framework
- Great developer experience
- Similar to Next.js

**Cons**:
- Smaller ecosystem than Next.js
- Less hosting options
- Newer (less mature)
- Fewer Brazilian developers familiar

**Verdict**: REJECTED - Next.js more mature and better hosting options

### Alternative 5: SvelteKit + PostgreSQL

**Pros**:
- Excellent performance
- Great developer experience
- Full-stack framework

**Cons**:
- Smaller ecosystem
- Fewer developers know Svelte
- Less mature than React/Next.js
- Harder to hire developers in Brazil

**Verdict**: REJECTED - Hiring challenges, smaller ecosystem

## Component Library Comparison

### Why Shadcn/UI over Material-UI or Chakra UI?

**Material-UI (MUI)**:
- Large bundle size impacts performance
- Opinionated design (Material Design)
- Package dependency (harder to customize deeply)

**Chakra UI**:
- Smaller than MUI but still package dependency
- Good but less trendy in 2025
- Component composition can be verbose

**Shadcn/UI**:
- Copy components, own the code
- Minimal bundle (only include what you use)
- Built on Radix (accessibility built-in)
- Tailwind styling (consistent with rest of app)
- Easy deep customization
- Increasingly popular in 2025

## ORM Comparison

### Why Prisma over TypeORM or Drizzle?

**TypeORM**:
- More complex API
- Decorator-based (less functional)
- Less intuitive

**Drizzle**:
- Newer, excellent performance
- Type-safe
- But less mature, smaller community

**Prisma**:
- Best developer experience
- Excellent TypeScript integration
- Great documentation
- Prisma Studio for database browsing
- Large community
- Migration system built-in

## Development Workflow

### Local Development

```bash
# Prerequisites
- Node.js 20 LTS
- PostgreSQL (Docker or local)
- npm or pnpm or yarn

# Setup
git clone <repository>
npm install
npm run db:push       # Push schema to database
npm run dev          # Start development server

# Access
http://localhost:3000 # Application
http://localhost:5555 # Prisma Studio
```

### Code Structure

```
mtgms/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # Auth routes group
│   │   ├── (dashboard)/       # Dashboard routes group
│   │   ├── api/               # API routes
│   │   └── layout.tsx         # Root layout
│   ├── components/            # React components
│   │   ├── ui/               # Shadcn/UI components
│   │   ├── forms/            # Form components
│   │   └── layouts/          # Layout components
│   ├── lib/                   # Core libraries
│   │   ├── db.ts             # Prisma client
│   │   ├── auth.ts           # Auth configuration
│   │   └── utils.ts          # Utilities
│   ├── modules/               # Business logic modules
│   │   ├── auth/             # Authentication module
│   │   ├── gym/              # Gym management module
│   │   ├── teacher/          # Teacher module
│   │   ├── student/          # Student module
│   │   ├── class/            # Class module
│   │   ├── payment/          # Payment module
│   │   └── multi-tenancy/    # Multi-tenancy logic
│   └── types/                 # TypeScript types
├── prisma/
│   ├── schema.prisma          # Database schema
│   └── migrations/            # Migration files
├── public/                    # Static assets
└── tests/                     # Test files
```

## Consequences

### Positive

1. **Single Language**: TypeScript everywhere (frontend, backend, database types)
2. **Rapid Development**: Next.js + Prisma = high productivity
3. **Type Safety**: Compile-time error catching across entire stack
4. **Cost Efficient**: Free tier hosting sufficient for MVP, max $39/month
5. **Performance**: Server components, edge functions, CDN = fast application
6. **Developer Experience**: Hot reload, TypeScript, great tooling
7. **Hiring**: Easy to find React/TypeScript developers in Brazil
8. **Scalability**: Can scale vertically (better hosting) or horizontally (more instances)
9. **Maintainability**: Single codebase, type safety, good separation of concerns
10. **Future Proof**: Modern stack, active development, long-term support

### Negative

1. **JavaScript Fatigue**: Ecosystem changes rapidly (mitigated by using stable versions)
2. **Next.js Updates**: Breaking changes between major versions (use LTS versions)
3. **Vercel Lock-in**: Optimized for Vercel (mitigated: can deploy elsewhere if needed)
4. **Serverless Cold Starts**: First request may be slow (mitigated: keep-alive pings)
5. **Learning Curve**: Team must learn Next.js App Router if unfamiliar

### Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| Vercel costs exceed budget | Monitor usage, can migrate to other hosts (Railway, Render, AWS) |
| Neon database too small | Upgrade to Pro ($19/month) or migrate to managed PostgreSQL |
| Next.js breaking changes | Lock to stable versions, test before upgrading |
| Performance issues | Implement caching, optimize queries, upgrade hosting tier |
| Team unfamiliarity | Allocate time for learning, use documentation extensively |

## Technology Version Pinning

```json
{
  "dependencies": {
    "next": "^14.2.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "@prisma/client": "^5.20.0",
    "next-auth": "^4.24.0",
    "zod": "^3.23.0",
    "@tanstack/react-query": "^5.50.0"
  },
  "devDependencies": {
    "typescript": "^5.5.0",
    "tailwindcss": "^3.4.0",
    "prisma": "^5.20.0",
    "@types/node": "^20.14.0",
    "@types/react": "^18.3.0"
  }
}
```

## Performance Benchmarks (Expected)

- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 2.5s
- **API Response**: < 300ms (95th percentile)
- **Database Query**: < 100ms (95th percentile)
- **Bundle Size**: < 200KB (gzipped JavaScript)

## Security Considerations

- **TypeScript**: Type safety reduces security vulnerabilities
- **Prisma**: Parameterized queries prevent SQL injection
- **NextAuth.js**: Secure session management, CSRF protection
- **Environment Variables**: Secrets stored securely (Vercel environment)
- **HTTPS**: Enforced by Vercel
- **Rate Limiting**: Implement at API route level

## Decision Makers

- Technical Lead
- Architecture Team
- Project Stakeholders

## Related Decisions

- ADR-001: Application Architecture Pattern (Modular Monolith)
- ADR-003: Database Selection and Design
- ADR-004: Deployment Environment and Infrastructure
- ADR-006: Security Architecture

## References

- [Next.js Documentation](https://nextjs.org/docs)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Vercel Platform](https://vercel.com/docs)
- [Neon Serverless Postgres](https://neon.tech/docs)
- [Shadcn/UI](https://ui.shadcn.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
