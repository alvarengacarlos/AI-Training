# ADR 003: Database Selection and Design

## Status
Accepted

## Date
2025-10-16

## Context

The Muay Thai Gym Management System requires a database solution that supports:

- **Multi-tenancy**: Complete data isolation between gyms (FR-006)
- **ACID transactions**: Critical for payment data integrity (NFR-035)
- **Budget constraint**: Must fit within $100/month total infrastructure budget
- **Performance**: Query response < 500ms for up to 500 students per gym
- **LGPD compliance**: Data encryption at rest and proper data handling (NFR-032)
- **Scalability**: Support 100 gyms, 20,000 students, millions of payment records over time
- **Relational integrity**: Complex relationships between gyms, teachers, students, classes, payments
- **Data retention**: Indefinite payment history for financial compliance

### Data Model Requirements

From requirements analysis:
- **12 core entities**: Root User, Gym, Administrator, Teacher, Student, Class, Work Schedule, Class Schedule, Enrollment, Payment Record, Notification Log, Notification Settings
- **Complex relationships**: One-to-many, many-to-many (enrollments)
- **Data volume estimates** (100 gyms, 5 years):
  - Students: 20,000
  - Payment records: 1,200,000
  - Total storage: < 1GB

## Decision

**We will use PostgreSQL 15+ as our primary database, hosted on Neon (serverless PostgreSQL) or Supabase as alternatives.**

### Database Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    PostgreSQL Database                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              Multi-Tenant Schema Design                 │ │
│  │                                                          │ │
│  │  All tables include gym_id for tenant isolation         │ │
│  │  Row-Level Security policies enforce access control     │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │   root_user  │  │     gym      │  │  administrator  │  │
│  └──────────────┘  └──────┬───────┘  └─────────────────┘  │
│                            │                                │
│           ┌────────────────┼────────────────┐              │
│           │                │                │              │
│  ┌────────▼────┐  ┌────────▼────┐  ┌───────▼──────────┐  │
│  │   teacher   │  │   student   │  │      class       │  │
│  └──────┬──────┘  └──────┬──────┘  └───────┬──────────┘  │
│         │                │                  │              │
│  ┌──────▼───────┐ ┌──────▼─────────┐ ┌─────▼─────────┐   │
│  │work_schedule│ │payment_record  │ │  enrollment   │   │
│  └─────────────┘ └────────────────┘ └───────────────┘   │
│                                                            │
│  ┌─────────────────┐  ┌───────────────────────┐          │
│  │class_schedule   │  │notification_settings  │          │
│  └─────────────────┘  └───────────────────────┘          │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │          Indexes and Constraints                     │  │
│  │  - Primary keys on all entities                      │  │
│  │  - Foreign keys with referential integrity           │  │
│  │  - Composite indexes on (gym_id, ...)                │  │
│  │  - Unique constraints where needed                   │  │
│  │  - Check constraints for business rules              │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │          Security and Encryption                     │  │
│  │  - pgcrypto extension for field-level encryption     │  │
│  │  - RLS policies for multi-tenant isolation           │  │
│  │  - SSL/TLS for data in transit                       │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## Why PostgreSQL?

### Advantages

1. **ACID Compliance**: Full transaction support critical for payment data
2. **Relational Model**: Perfect fit for structured, relational gym management data
3. **Mature and Stable**: 25+ years of development, battle-tested
4. **Performance**: Excellent query optimization, handles complex joins efficiently
5. **JSON Support**: JSONB type if semi-structured data needed later
6. **Full-Text Search**: Built-in search without external service
7. **Extensions**:
   - `pgcrypto`: Field-level encryption for LGPD compliance
   - `uuid-ossp`: UUID generation for primary keys
   - `pg_trgm`: Fuzzy text search (for student/teacher name search)
8. **Row-Level Security (RLS)**: Built-in multi-tenant isolation
9. **Triggers and Functions**: Business logic enforcement at database level
10. **Open Source**: No licensing costs, community support
11. **Ecosystem**: Excellent tool support (Prisma, pgAdmin, DBeaver)
12. **Cost Effective**: Free open-source database, cheap managed hosting

### PostgreSQL Features We'll Use

- **UUID Primary Keys**: Prevent enumeration attacks, support distributed systems
- **JSONB columns**: For flexible metadata if needed
- **Check constraints**: Enforce business rules (positive amounts, valid dates)
- **Foreign keys**: Maintain referential integrity
- **Indexes**: Optimize query performance (especially on gym_id)
- **Partial indexes**: Index only active records for performance
- **Triggers**: Audit logging, updated_at timestamp automation

## Hosting: Neon (Primary) vs Supabase (Alternative)

### Primary Choice: Neon Serverless PostgreSQL

**Why Neon?**

1. **Serverless**: Auto-scaling, pay for actual usage
2. **Generous Free Tier**:
   - 0.5GB storage (sufficient for MVP)
   - 3GB data transfer/month
   - Unlimited connections (with pooling)
   - 300 compute hours/month (sufficient for 100 concurrent users)
3. **Branching**: Database branches for testing/staging
4. **Low Latency**: Available via AWS São Paulo region
5. **Automated Backups**: Point-in-time recovery included
6. **Connection Pooling**: Built-in PgBouncer
7. **Cost**:
   - Free tier: $0/month (sufficient for MVP)
   - Pro tier: $19/month (if needed later)
8. **Standard PostgreSQL**: No vendor lock-in, can migrate easily

**Free Tier Limits Analysis**:
- 0.5GB storage = can store ~500,000 payment records (well above MVP needs)
- 3GB transfer = ~300,000 page loads/month (sufficient for 100 gyms)
- 300 compute hours = continuous availability with auto-suspend after inactivity

### Alternative: Supabase

**Why Supabase?**

1. **Also Serverless PostgreSQL**: Similar benefits to Neon
2. **Free Tier**:
   - 500MB database
   - 1GB file storage
   - 2GB bandwidth
   - Unlimited API requests
3. **Built-in Features**:
   - Authentication (though we'll use NextAuth.js)
   - Real-time subscriptions (if needed later)
   - Storage for files (gym icons)
   - Edge functions
4. **Brazilian Developer Community**: Popular in Brazil
5. **Cost**:
   - Free tier: $0/month
   - Pro tier: $25/month

**When to Choose Supabase Over Neon**:
- If file storage needed for gym icons (Supabase includes storage)
- If real-time features needed later (class roster updates)
- Team prefers Supabase ecosystem

**Decision**: Start with Neon (simpler, focused on database), can migrate to Supabase if storage/real-time features needed

## Multi-Tenancy Implementation

### Strategy: Shared Database, Shared Schema, Row-Level Isolation

```sql
-- Every tenant-specific table includes gym_id
CREATE TABLE student (
    student_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    gym_id UUID NOT NULL REFERENCES gym(gym_id) ON DELETE RESTRICT,
    name TEXT NOT NULL,
    phone TEXT NOT NULL,
    -- ... other fields
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Composite index for efficient tenant filtering
CREATE INDEX idx_student_gym_id ON student(gym_id);

-- Row-Level Security policy (optional, for defense-in-depth)
ALTER TABLE student ENABLE ROW LEVEL SECURITY;

CREATE POLICY student_isolation ON student
    USING (gym_id = current_setting('app.current_gym_id')::uuid);
```

### Multi-Tenancy Enforcement

1. **Application Layer**: Prisma middleware automatically adds gym_id filter
2. **Database Layer**: RLS policies provide secondary enforcement
3. **Query Pattern**:
   ```typescript
   // Every query automatically filtered by gym_id
   const students = await prisma.student.findMany({
     where: {
       gym_id: currentUser.gymId, // Always included
       isActive: true
     }
   });
   ```

### Benefits of This Approach

- **Simple**: Single database, no complex routing
- **Cost Effective**: Shared resources, no per-tenant databases
- **Performant**: Indexed gym_id makes filtering fast
- **Secure**: Application + RLS = defense in depth
- **Scalable**: Can support 100+ gyms easily

## Schema Design Highlights

### Primary Key Strategy: UUIDs

```sql
-- All tables use UUID primary keys
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE gym (
    gym_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    -- ...
);
```

**Why UUIDs?**
- Non-sequential prevents enumeration attacks
- Globally unique (no collisions across gyms)
- Can generate client-side if needed
- Better for distributed systems

### Timestamps and Audit Fields

```sql
-- All tables include audit timestamps
CREATE TABLE teacher (
    teacher_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    gym_id UUID NOT NULL REFERENCES gym(gym_id),
    -- ... fields ...
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Trigger to auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_teacher_updated_at
    BEFORE UPDATE ON teacher
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

### Soft Deletes for Historical Data

```sql
-- Important entities use soft deletes
CREATE TABLE student (
    student_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    gym_id UUID NOT NULL,
    -- ... fields ...
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    deleted_at TIMESTAMP NULL
);

-- Index for active records only (partial index)
CREATE INDEX idx_student_active ON student(gym_id) WHERE is_active = TRUE;
```

### Data Encryption (LGPD Compliance)

```sql
-- Enable pgcrypto extension
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Encrypt sensitive fields
CREATE TABLE student (
    student_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    gym_id UUID NOT NULL,
    name_encrypted BYTEA NOT NULL,  -- Encrypted name
    phone_encrypted BYTEA NOT NULL, -- Encrypted phone
    phone_hash TEXT NOT NULL,        -- Hash for searching
    -- ... other fields ...
);

-- Application layer handles encryption/decryption
-- Prisma middleware encrypts before write, decrypts after read
```

**Encryption Strategy**:
- **Application-level encryption**: Encrypt in application before DB write
- **Symmetric encryption**: AES-256 via pgcrypto
- **Key management**: Encryption keys stored in environment variables (Vercel secrets)
- **Searchable fields**: Store hash of encrypted field for exact match search
- **Balance**: Encrypt PII (names, addresses, phones), don't encrypt IDs, dates, amounts

### Indexes for Performance

```sql
-- Critical indexes for multi-tenant queries
CREATE INDEX idx_student_gym_id ON student(gym_id);
CREATE INDEX idx_teacher_gym_id ON teacher(gym_id);
CREATE INDEX idx_class_gym_id ON class(gym_id);
CREATE INDEX idx_payment_student_id ON payment_record(student_id);
CREATE INDEX idx_enrollment_class_id ON enrollment(class_id);
CREATE INDEX idx_enrollment_student_id ON enrollment(student_id);

-- Composite indexes for common queries
CREATE INDEX idx_student_gym_active ON student(gym_id, is_active);
CREATE INDEX idx_payment_student_date ON payment_record(student_id, payment_date DESC);

-- Unique constraints
CREATE UNIQUE INDEX idx_username_unique ON administrator(username) WHERE is_active = TRUE;
CREATE UNIQUE INDEX idx_enrollment_unique ON enrollment(student_id, class_id) WHERE is_active = TRUE;
```

## Prisma Schema Example

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model RootUser {
  id           String   @id @default(uuid()) @map("root_user_id") @db.Uuid
  username     String   @unique @db.VarChar(50)
  passwordHash String   @map("password_hash") @db.VarChar(255)
  email        String?  @unique @db.VarChar(100)
  createdAt    DateTime @default(now()) @map("created_at") @db.Timestamp(6)
  lastLogin    DateTime? @map("last_login") @db.Timestamp(6)
  isActive     Boolean  @default(true) @map("is_active")

  @@map("root_user")
}

model Gym {
  id        String   @id @default(uuid()) @map("gym_id") @db.Uuid
  name      String   @db.VarChar(100)
  address   String?  @db.Text
  ownerName String   @map("owner_name") @db.VarChar(100)
  iconUrl   String?  @map("icon_url") @db.VarChar(500)
  createdAt DateTime @default(now()) @map("created_at") @db.Timestamp(6)
  updatedAt DateTime @updatedAt @map("updated_at") @db.Timestamp(6)
  isActive  Boolean  @default(true) @map("is_active")

  administrators Administrator[]
  teachers       Teacher[]
  students       Student[]
  classes        Class[]

  @@map("gym")
}

model Administrator {
  id           String   @id @default(uuid()) @map("admin_id") @db.Uuid
  gymId        String   @map("gym_id") @db.Uuid
  username     String   @unique @db.VarChar(50)
  passwordHash String   @map("password_hash") @db.VarChar(255)
  email        String?  @db.VarChar(100)
  fullName     String?  @map("full_name") @db.VarChar(100)
  createdAt    DateTime @default(now()) @map("created_at") @db.Timestamp(6)
  lastLogin    DateTime? @map("last_login") @db.Timestamp(6)
  isActive     Boolean  @default(true) @map("is_active")

  gym Gym @relation(fields: [gymId], references: [id], onDelete: Restrict)

  @@index([gymId])
  @@map("administrator")
}

model Student {
  id              String   @id @default(uuid()) @map("student_id") @db.Uuid
  gymId           String   @map("gym_id") @db.Uuid
  name            String   @db.VarChar(100)
  phone           String   @db.VarChar(20)
  address         String?  @db.Text
  birthDate       DateTime @map("birth_date") @db.Date
  monthlyPrice    Decimal  @map("monthly_price") @db.Decimal(10, 2)
  billingCycleDay Int      @map("billing_cycle_day") @db.SmallInt
  createdAt       DateTime @default(now()) @map("created_at") @db.Timestamp(6)
  updatedAt       DateTime @updatedAt @map("updated_at") @db.Timestamp(6)
  isActive        Boolean  @default(true) @map("is_active")

  gym         Gym             @relation(fields: [gymId], references: [id], onDelete: Restrict)
  enrollments Enrollment[]
  payments    PaymentRecord[]

  @@index([gymId])
  @@index([gymId, isActive])
  @@map("student")
}

// ... Additional models for Teacher, Class, Enrollment, PaymentRecord, etc.
```

## Data Migration Strategy

### Initial Setup
```bash
# Initialize Prisma
npx prisma init

# Create migration
npx prisma migrate dev --name init

# Generate Prisma Client
npx prisma generate

# Seed initial data (root user)
npx prisma db seed
```

### Schema Evolution
- Use Prisma Migrate for schema changes
- Always create migration files (version control)
- Test migrations in Neon branch before production
- Rollback plan for every migration

## Backup and Recovery Strategy

### Neon Built-in Backups
- **Automated backups**: Daily snapshots (included)
- **Point-in-time recovery**: Restore to any point in last 7 days (free tier) or 30 days (Pro)
- **Backup retention**: 7 days (free), 30 days (Pro)

### Additional Backup Strategy
```bash
# Manual backup script (for extra safety)
#!/bin/bash
# Daily backup to S3 or local storage
pg_dump $DATABASE_URL | gzip > backup_$(date +%Y%m%d).sql.gz

# Upload to S3 (optional, additional cost)
aws s3 cp backup_$(date +%Y%m%d).sql.gz s3://mtgms-backups/
```

### Disaster Recovery Plan
1. **RPO (Recovery Point Objective)**: 24 hours (daily backups)
2. **RTO (Recovery Time Objective)**: 4 hours (restore from backup)
3. **Testing**: Quarterly disaster recovery drills

## Performance Optimization

### Query Optimization
- **Prisma select**: Only fetch needed fields
- **Eager loading**: Use `include` to avoid N+1 queries
- **Pagination**: Cursor-based pagination for large lists
- **Indexes**: Ensure all foreign keys and frequently queried fields indexed

### Connection Pooling
```typescript
// lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

**Connection Pooling**: Neon provides built-in connection pooling via PgBouncer

### Caching Strategy
```typescript
// Cache frequently accessed data (gym settings, etc.)
import { unstable_cache } from 'next/cache';

export const getGymSettings = unstable_cache(
  async (gymId: string) => {
    return await prisma.gym.findUnique({
      where: { id: gymId }
    });
  },
  ['gym-settings'],
  { revalidate: 3600 } // Cache for 1 hour
);
```

## Cost Analysis

### Neon Free Tier (MVP)
- **Storage**: 0.5GB (more than sufficient for 100 gyms)
- **Transfer**: 3GB/month (adequate for MVP traffic)
- **Compute**: 300 hours/month (sufficient with auto-suspend)
- **Cost**: $0/month

### Neon Pro Tier (If Needed)
- **Storage**: 10GB included, $0.125/GB beyond
- **Transfer**: Unlimited
- **Compute**: 750 hours/month, $0.16/hour beyond
- **Estimated cost**: $19/month (for better performance and more storage)

### Supabase Free Tier (Alternative)
- **Database**: 500MB
- **Storage**: 1GB (for files)
- **Bandwidth**: 2GB
- **Cost**: $0/month

### Migration to Self-Hosted (If Exceeds Budget)
- AWS RDS PostgreSQL: ~$15-30/month (t4g.micro in São Paulo)
- Digital Ocean Managed PostgreSQL: $15/month (1GB RAM)
- Always an option if managed services become expensive

## Alternatives Considered and Rejected

### MySQL
**Rejected because**:
- Less feature-rich than PostgreSQL (no JSONB, weaker full-text search)
- PostgreSQL has better JSON support for flexible data
- PostgreSQL ecosystem better for modern apps
- Both have similar performance

### MongoDB (NoSQL)
**Rejected because**:
- Relational data model (gyms, students, classes, payments) better fit for SQL
- Transactions across collections more complex than SQL joins
- Data integrity harder to enforce
- Team more familiar with SQL
- Payment data requires ACID guarantees

### SQLite
**Rejected because**:
- Not suitable for multi-user web application (file-based locking)
- Poor concurrency support
- No remote hosting
- Not production-ready for SaaS

### PlanetScale (MySQL-compatible)
**Considered but rejected**:
- Good option (serverless MySQL)
- More expensive than Neon/Supabase for small scale
- Team prefers PostgreSQL ecosystem
- Less flexible for complex queries

## Consequences

### Positive
1. **ACID Compliance**: Payment data integrity guaranteed
2. **Cost Effective**: Free tier sufficient for MVP, $0-19/month
3. **Performance**: Excellent query performance with proper indexes
4. **LGPD Compliant**: pgcrypto enables field-level encryption
5. **Mature Ecosystem**: Extensive tooling, documentation, community
6. **Type Safety**: Prisma generates TypeScript types from schema
7. **Multi-Tenancy**: Row-level isolation with gym_id filtering
8. **Scalability**: Can handle millions of records efficiently
9. **Developer Experience**: Prisma provides excellent DX
10. **Backup**: Automated backups included

### Negative
1. **Encryption Overhead**: Field-level encryption adds performance cost (mitigated: encrypt only PII)
2. **Connection Limits**: Serverless has connection limits (mitigated: connection pooling)
3. **Cold Starts**: Neon may have cold starts after inactivity (mitigated: keep-alive pings)
4. **Vendor Lock-in**: Neon-specific features (mitigated: standard PostgreSQL, easy migration)

### Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| Exceed free tier storage | Monitor storage, archive old data, upgrade to Pro ($19/month) |
| Exceed free tier bandwidth | Implement caching, optimize queries, upgrade tier |
| Database performance issues | Add indexes, optimize queries, implement caching |
| Data loss | Automated backups, test restore procedures quarterly |
| Neon service issues | Have backup plan to migrate to AWS RDS or Digital Ocean |

## Decision Makers
- Technical Lead
- Architecture Team

## Related Decisions
- ADR-001: Application Architecture Pattern (Modular Monolith)
- ADR-002: Technology Stack Selection (Next.js + Prisma)
- ADR-005: Multi-Tenancy Implementation Strategy
- ADR-006: Security Architecture (encryption, RLS)

## References
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [Neon Documentation](https://neon.tech/docs/introduction)
- [Multi-Tenancy Patterns](https://www.prisma.io/docs/guides/database/multi-tenancy)
- [pgcrypto Extension](https://www.postgresql.org/docs/current/pgcrypto.html)
