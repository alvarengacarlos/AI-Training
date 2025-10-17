# ADR 006: Security Architecture

## Status
Accepted

## Date
2025-10-16

## Context

MTGMS must comply with LGPD (Brazilian data protection law) and implement secure authentication, authorization, and data protection. Key requirements:
- LGPD compliance (NFR-032)
- Username/password authentication (FR-001, CONSTRAINT-012)
- Data encryption at rest and in transit (NFR-007, NFR-008)
- Multi-tenant data isolation (FR-006)
- Audit logging (NFR-010)
- Session security (NFR-011)

## Decision

**Implement layered security architecture with authentication via NextAuth.js, field-level encryption via pgcrypto, and comprehensive audit logging.**

### Security Layers

```
┌────────────────────────────────────────────────────┐
│  1. Transport Security (HTTPS/TLS)                 │
│     - Automatic via Vercel                         │
│     - TLS 1.2+ only                                │
└────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────┐
│  2. Authentication (NextAuth.js)                   │
│     - Username/password with bcrypt hashing        │
│     - Session-based (HttpOnly cookies)             │
│     - 4-hour timeout                               │
│     - Account lockout after 5 failures             │
└────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────┐
│  3. Authorization (RBAC)                           │
│     - Role: ROOT (all access)                      │
│     - Role: ADMIN (single gym access)              │
│     - Enforced at API route level                  │
└────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────┐
│  4. Data Access Control                            │
│     - Multi-tenancy via gym_id filtering           │
│     - Prisma middleware auto-filtering             │
│     - RLS policies as secondary defense            │
└────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────┐
│  5. Data Encryption                                │
│     - At rest: AES-256 via pgcrypto                │
│     - In transit: TLS 1.2+                         │
│     - Encrypted fields: name, phone, address       │
└────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────┐
│  6. Audit Logging                                  │
│     - All authentication events                    │
│     - All data modifications                       │
│     - Failed authorization attempts                │
│     - 90-day retention                             │
└────────────────────────────────────────────────────┘
```

### Authentication Implementation

```typescript
// lib/auth.ts
import { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import bcrypt from 'bcryptjs';

const MAX_LOGIN_ATTEMPTS = 5;
const LOCKOUT_DURATION = 15 * 60 * 1000; // 15 minutes

export const authOptions: NextAuthOptions = {
  providers: [
    CredentialsProvider({
      credentials: {
        username: { label: "Username", type: "text" },
        password: { label: "Password", type: "password" }
      },
      async authorize(credentials) {
        if (!credentials?.username || !credentials?.password) {
          return null;
        }

        const user = await prisma.administrator.findUnique({
          where: { username: credentials.username },
          include: { gym: true }
        });

        if (!user || !user.isActive) {
          await logFailedLogin(credentials.username);
          return null;
        }

        // Check account lockout
        if (await isAccountLocked(user.id)) {
          throw new Error('Account locked due to failed login attempts');
        }

        // Verify password
        const isValid = await bcrypt.compare(
          credentials.password,
          user.passwordHash
        );

        if (!isValid) {
          await recordFailedAttempt(user.id);
          return null;
        }

        // Reset failed attempts on successful login
        await resetFailedAttempts(user.id);
        await updateLastLogin(user.id);

        return {
          id: user.id,
          username: user.username,
          role: 'ADMIN',
          gymId: user.gymId,
          gymName: user.gym.name
        };
      }
    })
  ],
  session: {
    strategy: 'jwt',
    maxAge: 4 * 60 * 60, // 4 hours
  },
  pages: {
    signIn: '/login',
    error: '/login',
  },
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
  },
  cookies: {
    sessionToken: {
      name: 'next-auth.session-token',
      options: {
        httpOnly: true,
        sameSite: 'lax',
        path: '/',
        secure: process.env.NODE_ENV === 'production'
      }
    }
  }
};
```

### Password Requirements

```typescript
// lib/password-validation.ts
export const PASSWORD_REQUIREMENTS = {
  minLength: 8,
  requireUppercase: true,
  requireLowercase: true,
  requireNumber: true,
  requireSpecial: false // Not required for MVP to reduce user friction
};

export function validatePassword(password: string): {
  valid: boolean;
  errors: string[];
} {
  const errors: string[] = [];

  if (password.length < PASSWORD_REQUIREMENTS.minLength) {
    errors.push(`Password must be at least ${PASSWORD_REQUIREMENTS.minLength} characters`);
  }

  if (PASSWORD_REQUIREMENTS.requireUppercase && !/[A-Z]/.test(password)) {
    errors.push('Password must contain at least one uppercase letter');
  }

  if (PASSWORD_REQUIREMENTS.requireLowercase && !/[a-z]/.test(password)) {
    errors.push('Password must contain at least one lowercase letter');
  }

  if (PASSWORD_REQUIREMENTS.requireNumber && !/\d/.test(password)) {
    errors.push('Password must contain at least one number');
  }

  return {
    valid: errors.length === 0,
    errors
  };
}

// Hash password before storage
export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, 10); // Cost factor 10
}
```

### Data Encryption (LGPD Compliance)

```typescript
// lib/encryption.ts
import crypto from 'crypto';

const ALGORITHM = 'aes-256-cbc';
const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY!; // 32 bytes
const IV_LENGTH = 16;

export function encrypt(text: string): string {
  const iv = crypto.randomBytes(IV_LENGTH);
  const cipher = crypto.createCipheriv(
    ALGORITHM,
    Buffer.from(ENCRYPTION_KEY, 'hex'),
    iv
  );

  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');

  return iv.toString('hex') + ':' + encrypted;
}

export function decrypt(text: string): string {
  const [ivHex, encryptedHex] = text.split(':');
  const iv = Buffer.from(ivHex, 'hex');
  const decipher = crypto.createDecipheriv(
    ALGORITHM,
    Buffer.from(ENCRYPTION_KEY, 'hex'),
    iv
  );

  let decrypted = decipher.update(encryptedHex, 'hex', 'utf8');
  decrypted += decipher.final('utf8');

  return decrypted;
}

// Prisma middleware for automatic encryption/decryption
export function applyEncryptionMiddleware(prisma: PrismaClient) {
  prisma.$use(async (params, next) => {
    const encryptedFields = {
      Student: ['name', 'phone', 'address'],
      Teacher: ['name', 'phone', 'address'],
      Administrator: ['fullName']
    };

    const modelFields = encryptedFields[params.model as keyof typeof encryptedFields];

    // Encrypt on create/update
    if ((params.action === 'create' || params.action === 'update') && modelFields) {
      for (const field of modelFields) {
        if (params.args.data[field]) {
          params.args.data[field] = encrypt(params.args.data[field]);
        }
      }
    }

    const result = await next(params);

    // Decrypt on read
    if (result && modelFields) {
      if (Array.isArray(result)) {
        result.forEach(item => decryptFields(item, modelFields));
      } else if (typeof result === 'object') {
        decryptFields(result, modelFields);
      }
    }

    return result;
  });
}

function decryptFields(obj: any, fields: string[]) {
  for (const field of fields) {
    if (obj[field]) {
      try {
        obj[field] = decrypt(obj[field]);
      } catch (err) {
        // Handle decryption errors
        console.error(`Failed to decrypt field ${field}`, err);
      }
    }
  }
}
```

### Authorization Middleware

```typescript
// middleware/authorization.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';

export async function requireAuth(req: NextRequest) {
  const session = await getServerSession(authOptions);

  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  return session;
}

export async function requireGymAccess(
  req: NextRequest,
  gymId: string
) {
  const session = await requireAuth(req);

  if (session.user.role === 'ROOT') {
    return session; // Root can access any gym
  }

  if (session.user.gymId !== gymId) {
    return NextResponse.json(
      { error: 'Forbidden: Access denied to this gym' },
      { status: 403 }
    );
  }

  return session;
}
```

### Audit Logging

```sql
-- Audit log table
CREATE TABLE audit_log (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    gym_id UUID,
    action VARCHAR(50) NOT NULL, -- 'login', 'create_student', 'delete_teacher', etc.
    entity_type VARCHAR(50), -- 'student', 'teacher', 'payment', etc.
    entity_id UUID,
    old_value JSONB,
    new_value JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_audit_log_user ON audit_log(user_id, created_at DESC);
CREATE INDEX idx_audit_log_gym ON audit_log(gym_id, created_at DESC);
CREATE INDEX idx_audit_log_entity ON audit_log(entity_type, entity_id);
```

```typescript
// lib/audit-log.ts
export async function logAuditEvent(params: {
  userId: string;
  gymId?: string;
  action: string;
  entityType?: string;
  entityId?: string;
  oldValue?: any;
  newValue?: any;
  ipAddress?: string;
  userAgent?: string;
}) {
  await prisma.auditLog.create({
    data: {
      userId: params.userId,
      gymId: params.gymId,
      action: params.action,
      entityType: params.entityType,
      entityId: params.entityId,
      oldValue: params.oldValue,
      newValue: params.newValue,
      ipAddress: params.ipAddress,
      userAgent: params.userAgent
    }
  });
}

// Usage in API route
export async function POST(req: NextRequest) {
  const session = await requireAuth(req);

  const student = await prisma.student.create({
    data: req.body
  });

  await logAuditEvent({
    userId: session.user.id,
    gymId: session.user.gymId,
    action: 'create_student',
    entityType: 'student',
    entityId: student.id,
    newValue: student,
    ipAddress: req.ip,
    userAgent: req.headers.get('user-agent')
  });

  return NextResponse.json(student);
}
```

### Security Headers

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-XSS-Protection', '1; mode=block');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
  );

  return response;
}
```

### Rate Limiting

```typescript
// lib/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

// Create Redis client (Upstash free tier)
const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!
});

// Rate limit: 10 requests per 10 seconds per IP
export const rateLimiter = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '10 s'),
  analytics: true
});

// Usage in API route
export async function POST(req: NextRequest) {
  const ip = req.ip ?? '127.0.0.1';
  const { success } = await rateLimiter.limit(ip);

  if (!success) {
    return NextResponse.json(
      { error: 'Too many requests' },
      { status: 429 }
    );
  }

  // Process request...
}
```

## LGPD Compliance Checklist

- [x] **Legal Basis**: Legitimate interest (gym management)
- [x] **Data Minimization**: Only collect necessary data
- [x] **Encryption at Rest**: AES-256 via pgcrypto
- [x] **Encryption in Transit**: TLS 1.2+
- [x] **Access Control**: RBAC + multi-tenancy
- [x] **Audit Logging**: All data access logged
- [x] **Data Retention**: Defined policies
- [x] **Right to Access**: Can export user data
- [x] **Right to Deletion**: Soft delete with purge capability
- [x] **Privacy Policy**: Required (not implemented in code, legal document)

## Consequences

### Positive
1. LGPD compliant
2. Secure authentication with industry best practices
3. Encrypted sensitive data
4. Comprehensive audit trail
5. Defense in depth security model

### Negative
1. Encryption/decryption adds processing overhead (~5-10ms per operation)
2. Password requirements may frustrate some users (mitigated: clear error messages)
3. Session timeout may interrupt long sessions (mitigated: 4-hour window, warning)

## Decision Makers
- Technical Lead, Security Team, Legal Team

## Related Decisions
- ADR-002: Technology Stack
- ADR-005: Multi-Tenancy Implementation

## References
- [NextAuth.js Documentation](https://next-auth.js.org/)
- [LGPD Law (Brazil)](https://www.gov.br/cidadania/pt-br/acesso-a-informacao/lgpd)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
