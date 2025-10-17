# ADR 005: Multi-Tenancy Implementation Strategy

## Status
Accepted

## Date
2025-10-16

## Context

The MTGMS is a multi-tenant SaaS system where each gym operates independently with complete data isolation (FR-006). We need a strategy that:

- **Ensures complete data isolation** between gyms (security requirement)
- **Performs efficiently** (< 500ms query response)
- **Stays within budget** (single database, not per-tenant databases)
- **Prevents data leakage** under all circumstances
- **Scales to 100 gyms** without performance degradation
- **Simplifies development** (automatic tenant filtering)

### Multi-Tenancy Patterns Evaluated

1. **Separate Databases** per tenant
2. **Separate Schemas** per tenant
3. **Shared Schema** with row-level isolation (tenant_id column)

## Decision

**We will implement Shared Schema multi-tenancy with gym_id-based row-level isolation, enforced at both application and database layers.**

### Implementation Architecture

```typescript
// Multi-tenancy enforcement layers

┌──────────────────────────────────────────────────┐
│         1. Authentication Layer                  │
│  Identifies user and their gym_id(s)             │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│         2. Prisma Middleware                     │
│  Automatically adds gym_id to all queries        │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│         3. Database RLS Policies                 │
│  Secondary defense: enforce at DB level          │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│         4. Database                              │
│  All tenant data with gym_id column              │
└──────────────────────────────────────────────────┘
```

### Database Schema Pattern

```sql
-- Every tenant-scoped table includes gym_id
CREATE TABLE student (
    student_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    gym_id UUID NOT NULL REFERENCES gym(gym_id) ON DELETE RESTRICT,
    name TEXT NOT NULL,
    phone TEXT NOT NULL,
    -- ... other fields
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);

-- Index on gym_id for fast filtering
CREATE INDEX idx_student_gym_id ON student(gym_id);

-- Composite index for common queries
CREATE INDEX idx_student_gym_active ON student(gym_id, is_active)
    WHERE is_active = TRUE;

-- Row-Level Security (optional, defense-in-depth)
ALTER TABLE student ENABLE ROW LEVEL SECURITY;

CREATE POLICY student_isolation ON student
    USING (gym_id = current_setting('app.current_gym_id', true)::uuid);
```

### Prisma Middleware Implementation

```typescript
// lib/prisma-multi-tenancy.ts
import { Prisma, PrismaClient } from '@prisma/client';

export function applyMultiTenancyMiddleware(
  prisma: PrismaClient,
  gymId: string | null
) {
  prisma.$use(async (params, next) => {
    // List of models that are tenant-scoped
    const tenantModels = [
      'Student',
      'Teacher',
      'Class',
      'Payment',
      'Enrollment',
      'WorkSchedule',
      'ClassSchedule'
    ];

    // Root user bypass (no gym_id required)
    if (!gymId && isRootUserContext()) {
      return next(params);
    }

    // Enforce gym_id for tenant-scoped models
    if (tenantModels.includes(params.model || '')) {
      if (!gymId) {
        throw new Error('gym_id required for this operation');
      }

      // Add gym_id to all queries automatically
      if (params.action === 'findMany' || params.action === 'findFirst') {
        params.args.where = {
          ...params.args.where,
          gymId: gymId
        };
      }

      if (params.action === 'findUnique') {
        // For findUnique, check gym_id after fetch
        const result = await next(params);
        if (result && result.gymId !== gymId) {
          throw new Error('Access denied: resource belongs to different gym');
        }
        return result;
      }

      if (params.action === 'create' || params.action === 'createMany') {
        if (params.action === 'create') {
          params.args.data = {
            ...params.args.data,
            gymId: gymId
          };
        } else {
          // createMany
          params.args.data = params.args.data.map((item: any) => ({
            ...item,
            gymId: gymId
          }));
        }
      }

      if (params.action === 'update' || params.action === 'updateMany') {
        params.args.where = {
          ...params.args.where,
          gymId: gymId
        };
      }

      if (params.action === 'delete' || params.action === 'deleteMany') {
        params.args.where = {
          ...params.args.where,
          gymId: gymId
        };
      }
    }

    return next(params);
  });
}

// Usage in API route
export async function getPrismaClient(session: Session) {
  const prisma = new PrismaClient();
  const gymId = session.user.role === 'ROOT' ? null : session.user.gymId;
  applyMultiTenancyMiddleware(prisma, gymId);
  return prisma;
}
```

### Authentication Context

```typescript
// lib/auth.ts
import { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';

export const authOptions: NextAuthOptions = {
  providers: [
    CredentialsProvider({
      async authorize(credentials) {
        // Authenticate user
        const user = await authenticateUser(
          credentials.username,
          credentials.password
        );

        if (!user) return null;

        // Return user with gym context
        return {
          id: user.id,
          username: user.username,
          role: user.role, // 'ROOT' | 'ADMIN'
          gymId: user.gymId, // null for ROOT, gym_id for ADMIN
          gymName: user.gymName
        };
      }
    })
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.role = user.role;
        token.gymId = user.gymId;
        token.gymName = user.gymName;
      }
      return token;
    },
    async session({ session, token }) {
      session.user.role = token.role;
      session.user.gymId = token.gymId;
      session.user.gymName = token.gymName;
      return session;
    }
  }
};
```

### API Route Pattern

```typescript
// app/api/students/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';
import { getPrismaClient } from '@/lib/prisma-multi-tenancy';

export async function GET(req: NextRequest) {
  // 1. Authenticate user
  const session = await getServerSession(authOptions);
  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // 2. Get Prisma client with multi-tenancy middleware
  const prisma = await getPrismaClient(session);

  // 3. Query data - gym_id automatically added by middleware
  const students = await prisma.student.findMany({
    where: {
      isActive: true
      // gymId automatically added by middleware
    },
    orderBy: { name: 'asc' }
  });

  return NextResponse.json(students);
}
```

## Why Shared Schema Multi-Tenancy?

### Advantages

1. **Cost Effective**: Single database (free Neon tier sufficient)
2. **Simple Operations**: One database to backup, maintain, monitor
3. **Cross-Tenant Queries**: Root user can query across all gyms if needed
4. **Easy Onboarding**: New gym = new row, not new database
5. **Resource Sharing**: Efficient use of database connections and resources
6. **Consistent Schema**: All tenants use same schema version
7. **Simple Migrations**: One migration applies to all tenants

### Why NOT Separate Databases?

- **Cost**: Would require 100 databases ($$$)
- **Operations**: Complex backup/restore for 100+ databases
- **Migrations**: Must run migration 100+ times
- **Resource Waste**: Each database reserves resources
- **Connection Pooling**: Would need separate pool per database

### Why NOT Separate Schemas?

- **Complexity**: Middle ground complexity without clear benefits
- **Operations**: More complex than shared schema, less isolated than separate DBs
- **PostgreSQL Limitation**: Schema switching adds overhead
- **Not Worth It**: Shared schema with RLS provides similar isolation

## Security Enforcement Layers

### Layer 1: Application (Prisma Middleware)
**Primary defense**: Automatically adds gym_id to all queries

**Pros**:
- Transparent to developers
- Catches 99.9% of cases
- Fast (no extra DB round-trip)

**Cons**:
- Could be bypassed if middleware not applied

### Layer 2: Database (Row-Level Security)
**Secondary defense**: Database policies enforce isolation

```sql
-- Enable RLS on tenant tables
ALTER TABLE student ENABLE ROW LEVEL SECURITY;
ALTER TABLE teacher ENABLE ROW LEVEL SECURITY;
ALTER TABLE class ENABLE ROW LEVEL SECURITY;
-- etc.

-- Policy: Users can only see rows for their gym
CREATE POLICY student_isolation ON student
    USING (
        gym_id = current_setting('app.current_gym_id', true)::uuid
        OR
        current_setting('app.user_role', true) = 'ROOT'
    );

-- Application sets context per request
SET app.current_gym_id = 'gym-uuid-here';
SET app.user_role = 'ADMIN';
```

**Pros**:
- Database-level enforcement (defense-in-depth)
- Catches errors if middleware fails

**Cons**:
- Slight performance overhead
- Requires setting session variables

**Decision**: Implement both layers for maximum security

### Layer 3: Authorization Checks

```typescript
// For sensitive operations, double-check ownership
async function deleteStudent(studentId: string, gymId: string) {
  const student = await prisma.student.findUnique({
    where: { id: studentId }
  });

  if (!student) {
    throw new Error('Student not found');
  }

  if (student.gymId !== gymId) {
    throw new Error('Access denied: student belongs to different gym');
  }

  await prisma.student.delete({
    where: { id: studentId }
  });
}
```

## Special Cases

### Root User Access

Root users can access all gyms:

```typescript
// Root user queries without gym_id filter
if (session.user.role === 'ROOT') {
  // Can query across all gyms
  const allStudents = await prisma.student.findMany();

  // Or query specific gym
  const gymAStudents = await prisma.student.findMany({
    where: { gymId: 'gym-a-uuid' }
  });
}
```

### Cross-Gym Queries (Read-Only for Root)

```typescript
// Dashboard for root user: count students across all gyms
const gymStats = await prisma.gym.findMany({
  include: {
    _count: {
      select: { students: true, teachers: true, classes: true }
    }
  }
});
```

### Multi-Gym Teachers/Students

Same person can be registered in multiple gyms (separate profiles):

```typescript
// Same person, different gyms = different records
const teacher1 = await prisma.teacher.create({
  data: {
    gymId: 'gym-a-uuid',
    name: 'John Doe',
    phone: '+5511999999999'
    // ... separate profile for Gym A
  }
});

const teacher2 = await prisma.teacher.create({
  data: {
    gymId: 'gym-b-uuid',
    name: 'John Doe',
    phone: '+5511999999999'
    // ... separate profile for Gym B
  }
});

// No relationship between teacher1 and teacher2
```

## Performance Optimization

### Indexes

```sql
-- Every tenant-scoped table needs gym_id index
CREATE INDEX idx_student_gym_id ON student(gym_id);
CREATE INDEX idx_teacher_gym_id ON teacher(gym_id);
CREATE INDEX idx_class_gym_id ON class(gym_id);

-- Composite indexes for common filters
CREATE INDEX idx_student_gym_active
    ON student(gym_id, is_active) WHERE is_active = TRUE;

CREATE INDEX idx_payment_gym_date
    ON payment_record(gym_id, payment_date DESC);
```

### Query Patterns

```typescript
// GOOD: Index on gym_id makes this fast
const students = await prisma.student.findMany({
  where: {
    gymId: currentGymId,
    isActive: true
  }
});

// GOOD: Uses composite index
const recentPayments = await prisma.paymentRecord.findMany({
  where: { gymId: currentGymId },
  orderBy: { paymentDate: 'desc' },
  take: 20
});

// AVOID: Scanning without gym_id (Root user only)
const allStudents = await prisma.student.findMany(); // Slow for 20,000 students
```

## Testing Multi-Tenancy

### Unit Tests

```typescript
describe('Multi-tenancy isolation', () => {
  it('should only return students for current gym', async () => {
    const gymA = await createGym({ name: 'Gym A' });
    const gymB = await createGym({ name: 'Gym B' });

    await createStudent({ gymId: gymA.id, name: 'Student A' });
    await createStudent({ gymId: gymB.id, name: 'Student B' });

    const prismaA = getPrismaClient({ user: { gymId: gymA.id } });
    const studentsA = await prismaA.student.findMany();

    expect(studentsA).toHaveLength(1);
    expect(studentsA[0].name).toBe('Student A');
  });

  it('should prevent access to other gym data', async () => {
    const gymA = await createGym({ name: 'Gym A' });
    const gymB = await createGym({ name: 'Gym B' });

    const studentB = await createStudent({ gymId: gymB.id, name: 'Student B' });

    const prismaA = getPrismaClient({ user: { gymId: gymA.id } });

    await expect(
      prismaA.student.findUnique({ where: { id: studentB.id } })
    ).rejects.toThrow('Access denied');
  });
});
```

## Monitoring

```typescript
// Log all cross-tenant access attempts
logger.warn({
  userId: session.user.id,
  userGymId: session.user.gymId,
  attemptedGymId: requestedGymId,
  action: 'cross_tenant_access_attempt'
}, 'Cross-tenant access attempt detected');
```

## Consequences

### Positive
1. **Cost Effective**: Single database (free tier sufficient)
2. **Simple Operations**: One backup, one migration, one monitor
3. **Good Performance**: Indexed gym_id makes queries fast
4. **Defense in Depth**: Application + Database RLS layers
5. **Easy Onboarding**: New gym = new row
6. **Scalable**: Supports 100+ gyms easily

### Negative
1. **Noisy Neighbor**: One gym's heavy queries can affect others (mitigated: connection limits, query optimization)
2. **Discipline Required**: Developers must remember multi-tenancy (mitigated: automatic middleware)
3. **Cannot Independently Scale Tenants**: All share same resources (acceptable for similar-sized gyms)

### Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| Data leakage between gyms | Two-layer enforcement (app + DB RLS) |
| Developer forgets gym_id | Automatic Prisma middleware, code reviews |
| Performance degradation | Proper indexing, query optimization, monitoring |
| One gym affects others | Connection pooling, query timeouts, resource limits |

## Decision Makers
- Technical Lead
- Architecture Team
- Security Team

## Related Decisions
- ADR-001: Application Architecture Pattern
- ADR-003: Database Selection and Design
- ADR-006: Security Architecture

## References
- [Multi-Tenancy with Prisma](https://www.prisma.io/docs/guides/database/multi-tenancy)
- [PostgreSQL Row-Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [Multi-Tenant SaaS Patterns](https://docs.microsoft.com/en-us/azure/architecture/guide/multitenant/overview)
