# ADR 004: Deployment Environment and Infrastructure

## Status
Accepted

## Date
2025-10-16

## Context

The Muay Thai Gym Management System requires a deployment solution that:

- **Stays within $100/month budget** (CONSTRAINT-001)
- **Supports 100 concurrent users** with good performance (CONSTRAINT-008)
- **Provides 99% uptime** during business hours (NFR-012)
- **Serves Brazilian users** with low latency (NFR-001, CONSTRAINT-011)
- **Deploys Next.js application** with serverless functions
- **Requires minimal operational overhead** (< 2 hours/week, CONSTRAINT-023)
- **Supports automated deployments** with CI/CD (CONSTRAINT-020)
- **Provides monitoring and observability** (CONSTRAINT-021)
- **Handles HTTPS/SSL automatically** (NFR-008)

### Infrastructure Requirements

- **Application hosting**: Next.js with App Router, serverless functions
- **Database hosting**: PostgreSQL (Neon or Supabase)
- **Static assets**: CDN for images, CSS, JavaScript
- **SSL/TLS**: Automatic certificate management
- **Scalability**: Auto-scaling for traffic spikes
- **Backups**: Automated database backups
- **Monitoring**: Application performance monitoring, error tracking, uptime monitoring
- **CI/CD**: Automated testing and deployment pipeline

## Decision

**We will deploy the MTGMS on Vercel (frontend + API) + Neon (database), with total infrastructure cost of $0-39/month.**

### Deployment Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     USER TRAFFIC                             │
│           (Gym Administrators from Brazil)                   │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     │ HTTPS (443)
                     │
┌────────────────────▼─────────────────────────────────────────┐
│                 VERCEL EDGE NETWORK                          │
│              (Global CDN + Load Balancing)                   │
│                                                              │
│  ┌───────────────┐  ┌────────────────┐  ┌───────────────┐  │
│  │ Edge (Global) │  │ Edge (Americas)│  │ Edge (Brazil) │  │
│  │  - Routing    │  │  - Caching     │  │  - Closest    │  │
│  │  - SSL Term   │  │  - Static      │  │    to users   │  │
│  └───────────────┘  └────────────────┘  └───────────────┘  │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     │ Serverless Invocations
                     │
┌────────────────────▼─────────────────────────────────────────┐
│              VERCEL SERVERLESS FUNCTIONS                     │
│              (São Paulo AWS Region Preferred)                │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Next.js Application                                 │   │
│  │                                                       │   │
│  │  ┌────────────────┐  ┌───────────────────────────┐  │   │
│  │  │  React Pages   │  │  API Routes               │  │   │
│  │  │  - Dashboard   │  │  - /api/students          │  │   │
│  │  │  - Students    │  │  - /api/teachers          │  │   │
│  │  │  - Teachers    │  │  - /api/classes           │  │   │
│  │  │  - Classes     │  │  - /api/payments          │  │   │
│  │  │  - Payments    │  │  - /api/auth              │  │   │
│  │  └────────────────┘  └───────────────────────────┘  │   │
│  │                                                       │   │
│  │  Serverless Function Instance (auto-scales)          │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  Environment Variables (Secrets):                            │
│  - DATABASE_URL                                              │
│  - NEXTAUTH_SECRET                                           │
│  - ENCRYPTION_KEY                                            │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     │ Prisma Client
                     │ (Connection Pooling)
                     │
┌────────────────────▼─────────────────────────────────────────┐
│            NEON SERVERLESS POSTGRESQL                        │
│              (AWS São Paulo Region)                          │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  PostgreSQL 15                                        │  │
│  │  - Multi-tenant data with gym_id isolation            │  │
│  │  - Field-level encryption (pgcrypto)                  │  │
│  │  - Auto-suspend after inactivity (cost saving)        │  │
│  │  - Connection pooling (PgBouncer built-in)            │  │
│  │  - Automated daily backups                            │  │
│  │  - Point-in-time recovery (7 days)                    │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  Storage: 0.5GB (free tier) → 10GB (Pro)                    │
│  Compute: Auto-scaling, 300 hrs/month free                  │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    EXTERNAL SERVICES                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────┐  ┌─────────────────┐                 │
│  │  Sentry (Errors) │  │  UptimeRobot    │                 │
│  │  Free Tier       │  │  (Uptime Mon.)  │                 │
│  │  - Error tracking│  │  Free Tier      │                 │
│  │  - Performance   │  │  - 50 monitors  │                 │
│  └──────────────────┘  └─────────────────┘                 │
│                                                              │
│  ┌──────────────────┐  ┌─────────────────┐                 │
│  │  Twilio (SMS)    │  │  WhatsApp API   │                 │
│  │  Pay-per-use     │  │  Pay-per-use    │                 │
│  │  (Nice-to-have)  │  │  (Nice-to-have) │                 │
│  └──────────────────┘  └─────────────────┘                 │
└──────────────────────────────────────────────────────────────┘
```

## Why Vercel?

### Advantages

1. **Perfect for Next.js**: Built by the same team that created Next.js
2. **Generous Free Tier**:
   - 100GB bandwidth/month
   - Unlimited serverless function invocations
   - Unlimited preview deployments
   - Auto SSL certificates
   - Global CDN included
   - Build hours: 6,000 minutes/month
3. **Zero Configuration Deployment**:
   - Push to GitHub → Automatic deployment
   - No DevOps configuration needed
   - Environment variables via dashboard
4. **Global Edge Network**:
   - CDN with presence in Brazil
   - Fast static asset delivery
   - Edge caching for improved performance
5. **Automatic Scaling**:
   - Scales from 0 to 100+ concurrent users automatically
   - No capacity planning needed
   - Pay only for actual usage
6. **Built-in Features**:
   - Preview deployments for every PR
   - Rollback with one click
   - Web analytics
   - Speed Insights (Core Web Vitals)
7. **Developer Experience**:
   - Fast builds (< 2 minutes typical)
   - Hot reload in preview
   - Vercel CLI for local testing
8. **Cost Effective**: Free tier sufficient for MVP, Pro $20/month if needed

### Vercel Free Tier Limits (Is it Sufficient?)

**Bandwidth: 100GB/month**
- Average page size: 300KB (with images)
- 100GB = ~330,000 page loads/month
- 100 gyms × 5 page loads/day/gym × 30 days = 15,000 page loads/month
- **Verdict**: Sufficient with 20x safety margin

**Function Invocations: Unlimited on Free Tier**
- API calls unlimited
- **Verdict**: No concern

**Function Execution Time: 10 seconds max, 100GB-hours/month**
- Average API response: 300ms
- 100GB-hours = 360,000 seconds/month
- 360,000 seconds / 0.3 seconds = 1,200,000 API calls/month
- Estimated API calls: 100 gyms × 50 calls/day × 30 days = 150,000/month
- **Verdict**: Sufficient with 8x safety margin

**Build Time: 6,000 minutes/month**
- Average build: 2 minutes
- 6,000 / 2 = 3,000 builds/month
- Typical deployments: ~100/month (3-4/day during development)
- **Verdict**: Sufficient with 30x safety margin

**Conclusion**: Vercel Free Tier is sufficient for MVP. May need Pro tier ($20/month) for production for:
- Analytics and Speed Insights
- Better support
- Password protection for staging
- Higher limits for peace of mind

## Why Neon for Database?

(Covered in detail in ADR-003, summary here)

1. **Serverless PostgreSQL**: Pay for actual usage, auto-scales
2. **Free Tier**: 0.5GB storage, 3GB transfer, 300 compute hours/month
3. **Low Latency**: AWS São Paulo region
4. **Auto-suspend**: Saves costs during inactivity
5. **Connection Pooling**: Built-in PgBouncer
6. **Automated Backups**: Included, point-in-time recovery
7. **Cost**: $0/month (free tier) or $19/month (Pro)

## Infrastructure Cost Breakdown

### MVP Cost (Free Tiers)

| Service | Plan | Cost | Justification |
|---------|------|------|---------------|
| Vercel | Free Tier | $0 | Sufficient for 100 concurrent users |
| Neon | Free Tier | $0 | 0.5GB sufficient for MVP data |
| Sentry | Free Tier | $0 | 5,000 errors/month sufficient |
| UptimeRobot | Free Tier | $0 | 50 monitors sufficient |
| **Total** | | **$0/month** | **Perfect for MVP** |

### Production Cost (If Needed)

| Service | Plan | Cost | Justification |
|---------|------|------|---------------|
| Vercel | Pro | $20 | Better analytics, support, staging protection |
| Neon | Pro | $19 | More storage, better performance, 30-day backups |
| Sentry | Free Tier | $0 | Still sufficient |
| UptimeRobot | Free Tier | $0 | Still sufficient |
| **Total** | | **$39/month** | **Well under $100 budget** |

### Optional Services (Nice-to-Have)

| Service | Plan | Estimated Cost | Purpose |
|---------|------|----------------|---------|
| Twilio SMS | Pay-per-use | $5-10/month | SMS notifications to students |
| WhatsApp Business API | Pay-per-use | $5-10/month | WhatsApp notifications |
| **Total with Notifications** | | **$49-59/month** | Still under $100 budget |

## Alternative Hosting Options Considered

### Alternative 1: AWS (EC2 + RDS)

**Cost Estimate**:
- EC2 t4g.micro (São Paulo): $6/month
- RDS PostgreSQL t4g.micro: $15/month
- ALB (Load Balancer): $18/month
- Data transfer: $5/month
- **Total**: ~$44/month minimum

**Pros**:
- Full control over infrastructure
- More configuration options
- Can optimize for specific needs

**Cons**:
- Requires DevOps expertise
- Manual scaling configuration
- More operational overhead (> 2 hours/week)
- Manual SSL certificate management
- No built-in CDN (would need CloudFront, additional cost)
- Slower deployment (need to configure CI/CD)

**Verdict**: REJECTED - Too much operational complexity for budget

### Alternative 2: Digital Ocean (App Platform + Managed DB)

**Cost Estimate**:
- App Platform (Basic): $5/month
- Managed PostgreSQL: $15/month
- Bandwidth overage: $5/month
- **Total**: ~$25/month

**Pros**:
- Simple pricing
- Good for smaller apps
- Brazilian data center available

**Cons**:
- Less optimized for Next.js than Vercel
- Slower builds than Vercel
- Less generous free tier
- Limited serverless functions
- More manual configuration

**Verdict**: REJECTED - More expensive and less optimized than Vercel

### Alternative 3: Netlify + Supabase

**Cost Estimate**:
- Netlify Pro: $19/month (Free tier has limited bandwidth)
- Supabase Free: $0/month
- **Total**: ~$19/month

**Pros**:
- Good Next.js support
- Supabase includes auth, storage, real-time
- Similar developer experience to Vercel

**Cons**:
- Netlify less optimized for Next.js than Vercel
- More expensive than Vercel
- Netlify free tier more restrictive (100GB vs Vercel's 100GB)

**Verdict**: REJECTED - Vercel better for Next.js, more expensive

### Alternative 4: Railway

**Cost Estimate**:
- Railway: $5/month (after $5 free credit consumed)
- PostgreSQL: Included
- **Total**: ~$5/month

**Pros**:
- Simple deployment
- Includes PostgreSQL
- Fair pricing

**Cons**:
- Less mature than Vercel
- Smaller community
- Less optimized for serverless
- $5 free credit runs out quickly with database

**Verdict**: REJECTED - Less mature, Vercel better fit

### Alternative 5: Render

**Cost Estimate**:
- Web Service: $7/month
- PostgreSQL: $7/month
- **Total**: ~$14/month

**Pros**:
- Simple deployment
- Managed database
- Free tier available (with limitations)

**Cons**:
- Free tier services spin down after inactivity (unacceptable for SaaS)
- Paid tier more expensive than Vercel+Neon
- Less optimized for Next.js

**Verdict**: REJECTED - Free tier spin-down unacceptable, more expensive

## Deployment Strategy

### Development Workflow

```
┌─────────────────────────────────────────────────┐
│           Development Environment               │
├─────────────────────────────────────────────────┤
│  Developer Machine:                             │
│  - Next.js dev server (npm run dev)             │
│  - Local PostgreSQL or Neon dev branch          │
│  - Hot module reloading                         │
│  - Prisma Studio for DB browsing                │
└─────────────────────────────────────────────────┘
                     │
                     │ git push to feature branch
                     ▼
┌─────────────────────────────────────────────────┐
│            Preview Environment                  │
├─────────────────────────────────────────────────┤
│  Vercel Preview Deployment:                     │
│  - Automatic deployment per PR                  │
│  - Unique URL per preview                       │
│  - Connected to staging database (Neon branch)  │
│  - Allows testing before merge                  │
└─────────────────────────────────────────────────┘
                     │
                     │ Merge to main branch
                     ▼
┌─────────────────────────────────────────────────┐
│          Production Environment                 │
├─────────────────────────────────────────────────┤
│  Vercel Production Deployment:                  │
│  - Automatic deployment on main branch push     │
│  - Custom domain (mtgms.com.br)                 │
│  - Connected to production database (Neon)      │
│  - 99% uptime SLA                               │
└─────────────────────────────────────────────────┘
```

### CI/CD Pipeline

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test
      - run: npm run build

  # Vercel handles deployment automatically
  # No deployment job needed
```

**Vercel handles**:
- Automatic preview deployments for PRs
- Automatic production deployment on main branch
- Rollback capability
- Environment variable management

### Database Migration Strategy

```bash
# Development
npm run db:migrate:dev    # Create and apply migration

# Preview (Neon branch)
npm run db:migrate:preview # Apply to preview database

# Production
npm run db:migrate:prod   # Apply to production (manual approval)
```

**Neon Branching**: Create database branch for testing migrations before production

## Monitoring and Observability

### Application Monitoring

**Vercel Analytics** (Built-in):
- Page views
- Performance metrics (Core Web Vitals)
- Top pages
- Geographic distribution

**Sentry** (Error Tracking):
- Free tier: 5,000 errors/month
- Error tracking with stack traces
- Performance monitoring (transactions)
- Release tracking
- User context

### Uptime Monitoring

**UptimeRobot** (Free tier):
- 50 monitors (sufficient for all endpoints)
- 5-minute intervals
- Email/SMS alerts
- Status page
- 99.9% uptime tracking

### Database Monitoring

**Neon Dashboard**:
- Query performance
- Connection count
- Storage usage
- Compute usage

### Logging Strategy

```typescript
// lib/logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true
    }
  }
});

// In API routes
logger.info({ userId, gymId, action: 'create_student' }, 'Student created');
logger.error({ error, userId, gymId }, 'Failed to create student');
```

**Vercel Logs**:
- Integrated logging for all functions
- Searchable by time range, status code
- Retention: 24 hours (free), longer (Pro)

### Alerting Strategy

**Critical Alerts** (PagerDuty or similar, if needed):
- Database down
- Application error rate > 5%
- API response time > 5s

**Warning Alerts** (Email):
- Error rate > 1%
- Slow queries (> 1s)
- High resource usage (> 80%)

## Security and Compliance

### SSL/TLS
- Automatic HTTPS via Vercel
- Free SSL certificates (Let's Encrypt)
- Automatic renewal
- HTTPS redirect enforced

### Environment Variables (Secrets Management)
- Stored in Vercel dashboard (encrypted)
- Not committed to Git
- Different values per environment (dev, preview, prod)

**Required Secrets**:
```bash
DATABASE_URL              # Neon connection string
NEXTAUTH_SECRET           # Session encryption key
NEXTAUTH_URL              # Application URL
ENCRYPTION_KEY            # Field-level encryption key
TWILIO_API_KEY            # SMS (optional)
TWILIO_API_SECRET         # SMS (optional)
```

### DDoS Protection
- Vercel provides basic DDoS protection
- Rate limiting implemented at application level
- Firewall rules (if needed on Pro tier)

### Backup Strategy
- **Database**: Neon automated daily backups (7-day retention on free, 30-day on Pro)
- **Code**: Git repository (GitHub)
- **Environment variables**: Documented separately (encrypted)

## Disaster Recovery Plan

### Scenario 1: Vercel Outage
**RTO**: 4 hours
**RPO**: 1 hour (database only)

**Steps**:
1. Deploy to alternative host (Railway, Render, Netlify)
2. Update DNS to point to new host
3. Application code already in Git
4. Database remains on Neon (unaffected)

### Scenario 2: Neon Database Outage
**RTO**: 4 hours
**RPO**: 24 hours (daily backups)

**Steps**:
1. Restore from Neon backup (point-in-time recovery)
2. If Neon completely down, restore backup to AWS RDS or Digital Ocean
3. Update DATABASE_URL environment variable
4. Redeploy application

### Scenario 3: Complete Infrastructure Loss
**RTO**: 8 hours
**RPO**: 24 hours

**Steps**:
1. Restore database from backup to new provider
2. Deploy application to new host from Git
3. Configure environment variables
4. Update DNS
5. Validate functionality

## Scaling Plan

### Vertical Scaling (Within Budget)
1. **Free Tier → Vercel Pro**: $20/month (better performance, analytics)
2. **Free Tier → Neon Pro**: $19/month (more storage, better performance)
3. **Total**: $39/month (still under $100 budget)

### Horizontal Scaling (If Needed Post-MVP)
1. **Add caching layer**: Redis (Upstash free tier or $10/month)
2. **Optimize database**: Read replicas (Neon Scale tier, $69/month)
3. **CDN optimization**: Custom CDN configuration
4. **Total with optimizations**: ~$80-90/month (under budget)

### If Growth Exceeds Budget
1. **Migrate to self-hosted**: AWS/DO (~$40-50/month)
2. **Implement aggressive caching**: Reduce database load
3. **Optimize queries**: Reduce compute usage
4. **Increase pricing to customers**: Infrastructure growth = business growth

## Operational Procedures

### Daily Operations
- Monitor Vercel dashboard for errors (< 5 minutes/day)
- Check Sentry for new errors (< 5 minutes/day)
- Review UptimeRobot alerts (automatic, < 5 minutes/day)
- **Total**: < 15 minutes/day

### Weekly Operations
- Review performance metrics (< 30 minutes/week)
- Check database storage usage (< 10 minutes/week)
- Review and close Sentry issues (< 30 minutes/week)
- **Total**: < 90 minutes/week (under 2-hour target)

### Monthly Operations
- Review costs and optimize (< 1 hour/month)
- Database backup verification (< 30 minutes/month)
- Security updates (automatic via Dependabot)
- **Total**: < 2 hours/month

## Consequences

### Positive
1. **Cost Effective**: $0-39/month, well under $100 budget
2. **Zero Configuration**: Automatic deployments, no DevOps needed
3. **Excellent Performance**: CDN, serverless functions, low latency
4. **Scalability**: Auto-scales from 0 to 100+ users seamlessly
5. **Developer Experience**: Fast deployments, preview URLs, easy rollbacks
6. **Reliability**: 99%+ uptime, automated backups
7. **Security**: Auto SSL, DDoS protection, secure secrets management
8. **Low Operational Overhead**: < 2 hours/week maintenance
9. **Brazilian Performance**: Edge nodes and database in São Paulo region
10. **Future Flexibility**: Easy to migrate if outgrow platform

### Negative
1. **Vendor Lock-in**: Optimized for Vercel (mitigated: Next.js portable to other hosts)
2. **Serverless Limitations**: Function timeout (10s), cold starts (mitigated: keep-alive pings)
3. **Free Tier Limits**: May need upgrade to Pro (acceptable, still under budget)
4. **Less Control**: Can't fine-tune infrastructure (acceptable for small team)

### Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| Exceed bandwidth limits | Monitor usage, implement caching, upgrade to Pro ($20/month) |
| Database storage limits | Archive old data, upgrade to Neon Pro ($19/month) |
| Vercel downtime | Have emergency deployment plan to alternative host |
| Cost exceeds budget | Free tier sufficient for MVP, Pro still under budget |
| Performance issues | Implement caching, optimize queries, upgrade tiers |

## Decision Makers
- Technical Lead
- Architecture Team
- Finance/Budget Owner

## Related Decisions
- ADR-001: Application Architecture Pattern (Modular Monolith)
- ADR-002: Technology Stack Selection (Next.js)
- ADR-003: Database Selection (PostgreSQL on Neon)
- ADR-006: Security Architecture

## References
- [Vercel Documentation](https://vercel.com/docs)
- [Neon Documentation](https://neon.tech/docs)
- [Next.js Deployment](https://nextjs.org/docs/deployment)
- [Vercel Pricing](https://vercel.com/pricing)
- [Neon Pricing](https://neon.tech/pricing)
