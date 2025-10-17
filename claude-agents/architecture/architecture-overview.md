# Muay Thai Gym Management System - Architecture Overview

## Document Information
- **Version**: 1.0
- **Date**: 2025-10-16
- **Status**: Final
- **Project**: MTGMS MVP
- **Target Delivery**: 15 weeks

## Executive Summary

The Muay Thai Gym Management System (MTGMS) is a multi-tenant SaaS web application designed to streamline administrative operations for Muay Thai gyms in Brazil. This document provides a comprehensive overview of the system architecture, technology decisions, and implementation strategy.

### Key Architectural Decisions

1. **Application Architecture**: Modular Monolith (ADR-001)
2. **Technology Stack**: Next.js 14+ (React + TypeScript) + PostgreSQL (ADR-002)
3. **Database**: PostgreSQL 15+ hosted on Neon serverless (ADR-003)
4. **Deployment**: Vercel (frontend + API) + Neon (database) (ADR-004)
5. **Multi-Tenancy**: Shared schema with gym_id row-level isolation (ADR-005)
6. **Security**: NextAuth.js + AES-256 encryption + LGPD compliance (ADR-006)
7. **API Design**: REST API using Next.js App Router (ADR-007)

### Budget and Scale

- **Infrastructure Cost**: $0-39/month (well under $100 budget)
- **Concurrent Users**: 100 users
- **Target Gyms**: 100 gyms
- **Total Students**: ~20,000 students
- **Uptime**: 99% during business hours (6am-10pm BRT)

## System Architecture

### High-Level Architecture (C4 Level 1 - Context)

```
┌────────────────────────────────────────────────────────────────┐
│                     Gym Administrators                         │
│              (Root Users & Gym Admins)                         │
│            Access via web browser from Brazil                  │
└────────────────────┬───────────────────────────────────────────┘
                     │ HTTPS
                     │ (Web Browser)
┌────────────────────▼───────────────────────────────────────────┐
│                                                                │
│         Muay Thai Gym Management System (MTGMS)                │
│                                                                │
│  Multi-tenant SaaS for managing gyms, teachers, students,     │
│  classes, and payment tracking                                │
│                                                                │
└────────────┬───────────────────────────┬───────────────────────┘
             │                           │
             │ (Optional)                │ (Optional)
             │ SMS                       │ WhatsApp
             │                           │
┌────────────▼──────────┐    ┌───────────▼──────────────────────┐
│   Twilio SMS Gateway  │    │   WhatsApp Business API          │
│   (Nice-to-Have)      │    │   (Nice-to-Have)                 │
│                       │    │                                  │
│  Send payment         │    │  Send payment reminders          │
│  reminders to         │    │  to students                     │
│  students             │    │                                  │
└───────────────────────┘    └──────────────────────────────────┘
```

### Container Architecture (C4 Level 2)

```
┌──────────────────────────────────────────────────────────────┐
│                      User's Browser                          │
│                    (Chrome/Firefox/Edge)                     │
└────────────────────┬─────────────────────────────────────────┘
                     │ HTTPS
                     │
┌────────────────────▼─────────────────────────────────────────┐
│                  Vercel Edge Network                         │
│            (Global CDN + Load Balancing)                     │
└────────────────────┬─────────────────────────────────────────┘
                     │
┌────────────────────▼─────────────────────────────────────────┐
│              Next.js Web Application                         │
│          (Vercel Serverless Functions)                       │
│                                                              │
│  ┌──────────────────────┐  ┌───────────────────────────┐   │
│  │  React Frontend      │  │  API Routes (Backend)     │   │
│  │  - Server Components │  │  - REST API               │   │
│  │  - Client Components │  │  - Authentication         │   │
│  │  - Tailwind CSS UI   │  │  - Business Logic         │   │
│  │  - Forms & Validation│  │  - Prisma ORM             │   │
│  └──────────────────────┘  └─────────────┬─────────────┘   │
│                                           │                 │
│                           Prisma Client   │                 │
└───────────────────────────────────────────┼─────────────────┘
                                            │
                              Database Connection
                                            │
┌───────────────────────────────────────────▼─────────────────┐
│           Neon Serverless PostgreSQL                        │
│              (AWS São Paulo Region)                         │
│                                                             │
│  - Multi-tenant data with gym_id isolation                 │
│  - Field-level encryption (pgcrypto)                       │
│  - Automated backups (7-30 days)                           │
│  - Connection pooling (PgBouncer)                          │
└─────────────────────────────────────────────────────────────┘

External Services (Monitoring & Observability):
┌──────────────┐  ┌──────────────┐  ┌────────────────────┐
│ Vercel       │  │ Sentry       │  │ UptimeRobot        │
│ Analytics    │  │ (Errors)     │  │ (Uptime Monitor)   │
└──────────────┘  └──────────────┘  └────────────────────┘
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Frontend** | React | 18.3+ | UI library |
| | Next.js | 14.2+ | Full-stack framework |
| | TypeScript | 5.5+ | Type safety |
| | Tailwind CSS | 3.4+ | Styling |
| | Shadcn/UI | Latest | UI components |
| **Backend** | Node.js | 20 LTS | Runtime |
| | Next.js API Routes | 14.2+ | Backend framework |
| | Prisma | 5.20+ | ORM |
| | NextAuth.js | 4.24+ | Authentication |
| | Zod | 3.23+ | Validation |
| **Database** | PostgreSQL | 15+ | Relational database |
| | Neon | Latest | Serverless hosting |
| **Hosting** | Vercel | Latest | Application hosting |
| | Neon | Latest | Database hosting |
| **Monitoring** | Vercel Analytics | Built-in | Performance metrics |
| | Sentry | Free Tier | Error tracking |
| | UptimeRobot | Free Tier | Uptime monitoring |

## Architecture Principles

### 1. Modular Monolith

The system is organized as a modular monolith with clear module boundaries:

**Modules**:
- **Auth Module**: Authentication and session management
- **Gym Module**: Gym profile management
- **Teacher Module**: Teacher CRUD and scheduling
- **Student Module**: Student CRUD and enrollment
- **Class Module**: Class CRUD and capacity management
- **Payment Module**: Payment tracking and status calculation
- **Notification Module**: SMS/WhatsApp notifications (nice-to-have)
- **Multi-Tenancy Module**: Tenant isolation logic

**Benefits**:
- Single deployment (simple operations)
- Shared database (cost effective)
- Clear boundaries (maintainable)
- Can extract to microservices later if needed

### 2. Multi-Tenancy via Shared Schema

Every tenant-scoped table includes `gym_id` for row-level isolation:

```sql
-- Example: Student table
CREATE TABLE student (
    student_id UUID PRIMARY KEY,
    gym_id UUID NOT NULL REFERENCES gym(gym_id),
    name TEXT NOT NULL,
    -- ... other fields
);

-- Index for efficient filtering
CREATE INDEX idx_student_gym_id ON student(gym_id);
```

**Enforcement**:
1. **Prisma Middleware**: Automatically adds `gym_id` to all queries
2. **Database RLS**: Secondary defense via Row-Level Security policies
3. **Authorization Checks**: Explicit checks in sensitive operations

### 3. Security-First Design

**Layers of Security**:
1. **Transport**: HTTPS/TLS 1.2+ (automatic via Vercel)
2. **Authentication**: NextAuth.js with bcrypt password hashing
3. **Authorization**: RBAC (ROOT, ADMIN roles)
4. **Data Access**: Multi-tenant filtering enforced
5. **Encryption**: AES-256 for PII (names, phones, addresses)
6. **Audit**: Comprehensive logging of all actions

**LGPD Compliance**:
- Data minimization (collect only necessary data)
- Encryption at rest and in transit
- Audit trail for all data access
- Right to access and deletion supported
- Privacy policy required (legal document)

### 4. Performance Optimization

**Strategies**:
- **CDN**: Vercel edge network for static assets
- **Server Components**: Reduce client JavaScript
- **Database Indexes**: Optimized for multi-tenant queries
- **Caching**: Next.js built-in caching + future Redis if needed
- **Connection Pooling**: Neon built-in PgBouncer

**Expected Performance**:
- Page load: < 3 seconds (95th percentile)
- API response: < 500ms (95th percentile)
- Database query: < 100ms (95th percentile)

### 5. Cost Optimization

**Free Tier Usage**:
- Vercel Free Tier: $0/month (100GB bandwidth, unlimited functions)
- Neon Free Tier: $0/month (0.5GB storage, 3GB transfer)
- Sentry Free Tier: $0/month (5,000 errors/month)
- UptimeRobot Free Tier: $0/month (50 monitors)

**Total MVP Cost**: $0/month

**Production Upgrade** (if needed):
- Vercel Pro: $20/month
- Neon Pro: $19/month
- **Total**: $39/month (well under $100 budget)

## Data Architecture

### Database Schema Overview

**Core Entities** (12 total):
1. **root_user**: System administrators
2. **gym**: Gym profiles
3. **administrator**: Gym-level administrators
4. **teacher**: Instructors
5. **work_schedule**: Teacher availability
6. **student**: Gym members
7. **class**: Training sessions
8. **class_schedule**: Days of week for classes
9. **enrollment**: Student-class relationships (many-to-many)
10. **payment_record**: Payment transactions
11. **notification_log**: Sent notifications (nice-to-have)
12. **notification_settings**: Per-gym notification config (nice-to-have)

### Key Relationships

```
Root User (1) ──── (N) Gym
Gym (1) ──── (N) Administrator
Gym (1) ──── (N) Teacher
Gym (1) ──── (N) Student
Gym (1) ──── (N) Class
Teacher (1) ──── (N) Work Schedule
Teacher (1) ──── (N) Class
Class (1) ──── (N) Class Schedule
Student (N) ──── (N) Class (via Enrollment)
Student (1) ──── (N) Payment Record
```

### Data Volume Estimates

**MVP (100 gyms, 5 years)**:
- Gyms: 100
- Students: 20,000 (200 per gym average)
- Teachers: 1,000 (10 per gym average)
- Classes: 2,000 (20 per gym average)
- Enrollments: 40,000
- Payment Records: 1,200,000 (240k per year)
- **Total Storage**: < 1GB (well within Neon free tier)

## API Architecture

### REST API Design

**Base URL**: `https://mtgms.com.br/api`

**Endpoints Structure**:
```
/api/auth/login                              POST
/api/gyms                                    GET, POST
/api/gyms/:gymId                             GET, PUT, DELETE
/api/gyms/:gymId/students                    GET, POST
/api/gyms/:gymId/students/:studentId         GET, PUT, DELETE
/api/gyms/:gymId/teachers                    GET, POST
/api/gyms/:gymId/teachers/:teacherId         GET, PUT, DELETE
/api/gyms/:gymId/classes                     GET, POST
/api/gyms/:gymId/classes/:classId            GET, PUT, DELETE
/api/gyms/:gymId/classes/:classId/enrollments GET, POST, DELETE
/api/gyms/:gymId/payments                    GET, POST
/api/students/:studentId/payments            GET
```

**Authentication**: JWT tokens in HttpOnly cookies (NextAuth.js)

**Authorization**: Enforced at API route level via middleware

**Error Handling**: Standardized error responses with error codes

## Deployment Architecture

### CI/CD Pipeline

```
Developer Push to GitHub
         ↓
GitHub Actions (CI)
  - Lint code
  - Type check
  - Run tests
  - Build application
         ↓
Vercel (CD)
  - Preview deployment (PRs)
  - Production deployment (main branch)
  - Automatic SSL
  - Edge distribution
         ↓
Production Live
  - Monitor via Vercel Analytics
  - Track errors via Sentry
  - Monitor uptime via UptimeRobot
```

### Environments

1. **Development**: Local machine (localhost:3000)
2. **Preview**: Vercel preview deployments (per PR)
3. **Staging**: Optional Neon branch + Vercel preview
4. **Production**: Vercel production + Neon production

### Disaster Recovery

**RTO**: 4 hours (Recovery Time Objective)
**RPO**: 24 hours (Recovery Point Objective)

**Backup Strategy**:
- Database: Automated daily backups via Neon (7-day retention free, 30-day Pro)
- Code: Git repository on GitHub
- Environment Variables: Documented separately (encrypted)

**Recovery Procedures**:
1. **Vercel Outage**: Deploy to alternative host (Railway, Render)
2. **Neon Outage**: Restore from backup to AWS RDS or Digital Ocean
3. **Complete Loss**: Restore database from backup, redeploy from Git

## Monitoring and Operations

### Application Monitoring

**Vercel Analytics** (Built-in):
- Page views and user analytics
- Core Web Vitals (LCP, FID, CLS)
- Geographic distribution

**Sentry** (Error Tracking):
- JavaScript errors
- API errors
- Performance issues
- User context and breadcrumbs

**UptimeRobot** (Uptime Monitoring):
- HTTP endpoint checks every 5 minutes
- Email alerts on downtime
- Status page

### Operational Overhead

**Daily** (< 15 minutes):
- Check Vercel dashboard for errors
- Review Sentry new issues
- Check UptimeRobot alerts

**Weekly** (< 90 minutes):
- Review performance metrics
- Database storage usage check
- Close resolved Sentry issues

**Monthly** (< 2 hours):
- Cost review and optimization
- Backup verification
- Security updates (mostly automatic)

**Total**: < 2 hours/week (meets CONSTRAINT-023 requirement)

## Scalability Strategy

### Vertical Scaling (Within Budget)

**Phase 1** (Free Tier - MVP):
- Vercel Free Tier
- Neon Free Tier
- **Cost**: $0/month
- **Capacity**: 100 concurrent users

**Phase 2** (Pro Tiers - Growth):
- Vercel Pro: $20/month
- Neon Pro: $19/month
- **Cost**: $39/month
- **Capacity**: 500 concurrent users

**Phase 3** (Enhanced Performance):
- Add Redis caching (Upstash): $10/month
- Neon Scale tier: $69/month
- **Cost**: ~$80-90/month
- **Capacity**: 1000+ concurrent users

### Horizontal Scaling (Future)

If growth exceeds single-instance capacity:
1. **Extract Notification Module**: Separate service for async notifications
2. **Database Read Replicas**: Neon Scale tier supports read replicas
3. **CDN Optimization**: Custom CDN configuration for better caching
4. **Migrate to Multi-Region**: If international expansion

## Development Workflow

### Local Development

```bash
# Prerequisites
- Node.js 20 LTS
- npm or pnpm
- Git

# Setup
git clone <repository>
cd mtgms
npm install
npm run db:push      # Push schema to database
npm run dev          # Start dev server (localhost:3000)

# Other commands
npm run build        # Build for production
npm run lint         # Lint code
npm run test         # Run tests
npm run type-check   # TypeScript type checking
```

### Code Structure

```
mtgms/
├── src/
│   ├── app/                # Next.js App Router
│   │   ├── (auth)/        # Auth routes (login, logout)
│   │   ├── (dashboard)/   # Dashboard routes (protected)
│   │   ├── api/           # API routes
│   │   └── layout.tsx     # Root layout
│   ├── components/        # React components
│   │   ├── ui/           # Shadcn/UI components
│   │   ├── forms/        # Form components
│   │   └── layouts/      # Layout components
│   ├── lib/              # Core libraries
│   │   ├── db.ts         # Prisma client
│   │   ├── auth.ts       # Auth configuration
│   │   └── utils.ts      # Utility functions
│   ├── modules/          # Business logic modules
│   │   ├── auth/
│   │   ├── gym/
│   │   ├── teacher/
│   │   ├── student/
│   │   ├── class/
│   │   ├── payment/
│   │   └── multi-tenancy/
│   └── types/            # TypeScript types
├── prisma/
│   ├── schema.prisma     # Database schema
│   └── migrations/       # Migration files
├── public/               # Static assets
└── tests/                # Test files
```

## Security Considerations

### Authentication
- Bcrypt password hashing (cost factor 10)
- Session-based auth with 4-hour timeout
- Account lockout after 5 failed attempts (15-minute lockout)
- HttpOnly, Secure, SameSite cookies

### Authorization
- Role-Based Access Control (RBAC)
- ROOT role: Full system access
- ADMIN role: Single gym access only
- Enforced at API route level

### Data Protection
- **Encryption at Rest**: AES-256 for PII fields
- **Encryption in Transit**: TLS 1.2+ (automatic via Vercel)
- **Multi-Tenancy Isolation**: Two-layer enforcement (app + DB RLS)
- **Audit Logging**: All actions logged for 90 days

### LGPD Compliance
- Legal basis: Legitimate interest
- Data minimization principle followed
- Encryption of personal data
- Right to access and deletion supported
- Audit trail for compliance verification
- Privacy policy required (separate legal document)

## Future Enhancements (Post-MVP)

### High Priority
1. **Attendance Tracking**: Check-in/check-out functionality
2. **Financial Reports**: Revenue reports, payment analytics
3. **Student Portal**: Self-service for students
4. **Multi-Factor Authentication**: Enhanced security
5. **Data Export**: CSV/Excel export functionality

### Medium Priority
6. **Teacher Portal**: Self-service for teachers
7. **Online Payment Processing**: Integration with payment gateways
8. **Invoicing**: Automated invoice generation
9. **Student Photos**: ID card generation with photos
10. **Family Management**: Link family members, shared billing

### Low Priority
11. **Mobile Native Apps**: iOS and Android apps
12. **Advanced Analytics**: Business intelligence dashboards
13. **Marketing Automation**: Campaign management
14. **Event Management**: Competitions, seminars, belt tests

## Risks and Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Exceed bandwidth limits | Service degradation | Low | Monitor usage, implement caching, upgrade tier |
| Database storage limits | Service degradation | Low | Archive old data, upgrade to Pro tier |
| Performance issues | Poor UX | Medium | Implement caching, optimize queries, monitoring |
| Security breach | Critical | Low | Security audits, penetration testing, encryption |
| Cost overrun | Budget exceeded | Low | Free tier sufficient for MVP, Pro still under budget |
| Vendor lock-in (Vercel) | Migration difficulty | Medium | Next.js portable, can deploy elsewhere |
| Team unavailability | Delivery delay | Medium | Documentation, code reviews, knowledge sharing |

## Success Criteria

### Technical Metrics
- [ ] Page load time < 3 seconds (95th percentile)
- [ ] API response time < 500ms (95th percentile)
- [ ] Support 100 concurrent users without degradation
- [ ] 99% uptime during business hours
- [ ] Infrastructure cost ≤ $39/month

### Business Metrics
- [ ] Student enrollment time < 2 minutes
- [ ] Payment recording time < 1 minute
- [ ] 100% payment status visibility
- [ ] Zero data loss incidents
- [ ] LGPD compliance verified

### User Satisfaction
- [ ] Administrators can complete tasks without training
- [ ] System faster than previous manual methods
- [ ] Positive user feedback on usability
- [ ] Low error rate (< 0.1% of requests)

## Conclusion

The MTGMS architecture is designed to deliver a cost-effective, secure, and scalable multi-tenant SaaS solution for Muay Thai gym management. The modular monolith approach balances simplicity with maintainability, while the technology stack (Next.js + PostgreSQL + Vercel + Neon) provides excellent developer experience and stays well within the $100/month budget.

The architecture supports 100 concurrent users across 100 gyms with 20,000+ students, with a clear path to scale vertically (upgrade tiers) or horizontally (extract modules) as needed. Security is prioritized with LGPD compliance built-in from day one.

With this architecture, the MVP can be delivered in 15 weeks by a small team (1-3 developers) with minimal operational overhead (< 2 hours/week), while maintaining high quality, security, and performance standards.

## References

### Architecture Decision Records
- [ADR-001: Application Architecture Pattern](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/adr/001-application-architecture-pattern.md)
- [ADR-002: Technology Stack Selection](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/adr/002-technology-stack-selection.md)
- [ADR-003: Database Selection and Design](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/adr/003-database-selection-and-design.md)
- [ADR-004: Deployment Environment and Infrastructure](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/adr/004-deployment-environment-and-infrastructure.md)
- [ADR-005: Multi-Tenancy Implementation Strategy](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/adr/005-multi-tenancy-implementation.md)
- [ADR-006: Security Architecture](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/adr/006-security-architecture.md)
- [ADR-007: API Design Approach](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/adr/007-api-design-approach.md)

### Diagrams
- [System Context Diagram](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/diagrams/system-context.puml)
- [Container Diagram](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/diagrams/container-diagram.puml)
- [Component Diagram](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/diagrams/component-diagram.puml)
- [Deployment Diagram](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/diagrams/deployment-diagram.puml)
- [Database ERD](/home/alvarengacarlos/dev/AI-Training/claude-agents/architecture/diagrams/database-erd.puml)

### External Resources
- [Next.js Documentation](https://nextjs.org/docs)
- [Prisma Documentation](https://www.prisma.io/docs)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Vercel Platform](https://vercel.com/docs)
- [Neon Serverless Postgres](https://neon.tech/docs)
- [LGPD Law (Brazil)](https://www.gov.br/cidadania/pt-br/acesso-a-informacao/lgpd)

---

**Document Revision History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-10-16 | Architecture Team | Initial comprehensive architecture overview |
