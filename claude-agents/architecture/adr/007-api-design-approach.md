# ADR 007: API Design Approach

## Status
Accepted

## Date
2025-10-16

## Context

The MTGMS needs a well-designed API for communication between frontend and backend. The API must:
- Support CRUD operations for all entities
- Be RESTful and intuitive
- Handle authentication and authorization
- Provide good error handling
- Be type-safe with TypeScript
- Support future mobile apps or integrations

## Decision

**We will implement a REST API using Next.js App Router API routes with TypeScript, following REST conventions and resource-oriented design.**

### API Structure

```
/api/
├── auth/
│   ├── login/route.ts          # POST /api/auth/login
│   └── logout/route.ts         # POST /api/auth/logout
├── gyms/
│   ├── route.ts                # GET/POST /api/gyms
│   └── [gymId]/
│       ├── route.ts            # GET/PUT/DELETE /api/gyms/:gymId
│       ├── students/
│       │   ├── route.ts        # GET/POST /api/gyms/:gymId/students
│       │   └── [studentId]/
│       │       └── route.ts    # GET/PUT/DELETE /api/gyms/:gymId/students/:studentId
│       ├── teachers/
│       │   ├── route.ts        # GET/POST /api/gyms/:gymId/teachers
│       │   └── [teacherId]/
│       │       └── route.ts    # GET/PUT/DELETE /api/gyms/:gymId/teachers/:teacherId
│       ├── classes/
│       │   ├── route.ts        # GET/POST /api/gyms/:gymId/classes
│       │   └── [classId]/
│       │       ├── route.ts    # GET/PUT/DELETE /api/gyms/:gymId/classes/:classId
│       │       └── enrollments/
│       │           └── route.ts # GET/POST/DELETE /api/gyms/:gymId/classes/:classId/enrollments
│       └── payments/
│           ├── route.ts        # GET/POST /api/gyms/:gymId/payments
│           └── [paymentId]/
│               └── route.ts    # GET /api/gyms/:gymId/payments/:paymentId
└── students/
    └── [studentId]/
        ├── payments/
        │   └── route.ts        # GET /api/students/:studentId/payments
        └── classes/
            └── route.ts        # GET /api/students/:studentId/classes
```

### REST Conventions

| Method | Endpoint | Description | Request Body | Response |
|--------|----------|-------------|--------------|----------|
| GET | /api/gyms/:gymId/students | List students | - | Student[] |
| GET | /api/gyms/:gymId/students/:id | Get student | - | Student |
| POST | /api/gyms/:gymId/students | Create student | CreateStudentDto | Student |
| PUT | /api/gyms/:gymId/students/:id | Update student | UpdateStudentDto | Student |
| DELETE | /api/gyms/:gymId/students/:id | Delete student | - | { success: boolean } |

### API Route Pattern

```typescript
// app/api/gyms/[gymId]/students/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { requireGymAccess } from '@/lib/auth';
import { prisma } from '@/lib/db';

// Request validation schema
const CreateStudentSchema = z.object({
  name: z.string().min(2).max(100),
  phone: z.string().regex(/^\+55\d{10,11}$/),
  address: z.string().optional(),
  birthDate: z.string().datetime(),
  monthlyPrice: z.number().positive(),
  billingCycleDay: z.number().int().min(1).max(31)
});

// GET /api/gyms/:gymId/students
export async function GET(
  req: NextRequest,
  { params }: { params: { gymId: string } }
) {
  try {
    // Authorize
    const session = await requireGymAccess(req, params.gymId);

    // Parse query params
    const searchParams = req.nextUrl.searchParams;
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '50');
    const search = searchParams.get('search') || '';
    const status = searchParams.get('status'); // 'active' | 'all'

    // Query
    const where = {
      gymId: params.gymId,
      ...(status === 'active' && { isActive: true }),
      ...(search && {
        OR: [
          { name: { contains: search, mode: 'insensitive' } },
          { phone: { contains: search } }
        ]
      })
    };

    const [students, total] = await Promise.all([
      prisma.student.findMany({
        where,
        orderBy: { name: 'asc' },
        skip: (page - 1) * limit,
        take: limit,
        include: {
          _count: {
            select: { enrollments: true, payments: true }
          }
        }
      }),
      prisma.student.count({ where })
    ]);

    return NextResponse.json({
      data: students,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    return handleApiError(error);
  }
}

// POST /api/gyms/:gymId/students
export async function POST(
  req: NextRequest,
  { params }: { params: { gymId: string } }
) {
  try {
    const session = await requireGymAccess(req, params.gymId);

    const body = await req.json();
    const data = CreateStudentSchema.parse(body);

    const student = await prisma.student.create({
      data: {
        gymId: params.gymId,
        ...data
      }
    });

    await logAuditEvent({
      userId: session.user.id,
      gymId: params.gymId,
      action: 'create_student',
      entityType: 'student',
      entityId: student.id,
      newValue: student
    });

    return NextResponse.json(student, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    return handleApiError(error);
  }
}
```

### Error Handling

```typescript
// lib/api-errors.ts
export class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public code?: string
  ) {
    super(message);
  }
}

export function handleApiError(error: unknown) {
  console.error('API Error:', error);

  if (error instanceof ApiError) {
    return NextResponse.json(
      { error: error.message, code: error.code },
      { status: error.statusCode }
    );
  }

  if (error instanceof z.ZodError) {
    return NextResponse.json(
      {
        error: 'Validation failed',
        details: error.errors.map(e => ({
          field: e.path.join('.'),
          message: e.message
        }))
      },
      { status: 400 }
    );
  }

  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    if (error.code === 'P2002') {
      return NextResponse.json(
        { error: 'Resource already exists', code: 'DUPLICATE' },
        { status: 409 }
      );
    }
    if (error.code === 'P2025') {
      return NextResponse.json(
        { error: 'Resource not found', code: 'NOT_FOUND' },
        { status: 404 }
      );
    }
  }

  return NextResponse.json(
    { error: 'Internal server error', code: 'INTERNAL_ERROR' },
    { status: 500 }
  );
}
```

### Type-Safe API Client

```typescript
// lib/api-client.ts
import { z } from 'zod';

// Type definitions
export const StudentSchema = z.object({
  id: z.string().uuid(),
  gymId: z.string().uuid(),
  name: z.string(),
  phone: z.string(),
  address: z.string().nullable(),
  birthDate: z.date(),
  monthlyPrice: z.number(),
  billingCycleDay: z.number(),
  isActive: z.boolean(),
  createdAt: z.date(),
  updatedAt: z.date()
});

export type Student = z.infer<typeof StudentSchema>;

// API client
export class ApiClient {
  constructor(private baseUrl: string = '/api') {}

  async getStudents(gymId: string, params?: {
    page?: number;
    limit?: number;
    search?: string;
    status?: 'active' | 'all';
  }) {
    const queryParams = new URLSearchParams();
    if (params?.page) queryParams.set('page', params.page.toString());
    if (params?.limit) queryParams.set('limit', params.limit.toString());
    if (params?.search) queryParams.set('search', params.search);
    if (params?.status) queryParams.set('status', params.status);

    const response = await fetch(
      `${this.baseUrl}/gyms/${gymId}/students?${queryParams}`,
      {
        headers: { 'Content-Type': 'application/json' }
      }
    );

    if (!response.ok) {
      throw new Error(`API error: ${response.statusText}`);
    }

    return response.json();
  }

  async createStudent(gymId: string, data: CreateStudentDto) {
    const response = await fetch(`${this.baseUrl}/gyms/${gymId}/students`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message || 'Failed to create student');
    }

    return response.json();
  }

  // ... other methods
}

export const api = new ApiClient();
```

### Frontend Usage with TanStack Query

```typescript
// hooks/use-students.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '@/lib/api-client';

export function useStudents(gymId: string, filters?: StudentFilters) {
  return useQuery({
    queryKey: ['students', gymId, filters],
    queryFn: () => api.getStudents(gymId, filters)
  });
}

export function useCreateStudent(gymId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateStudentDto) => api.createStudent(gymId, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['students', gymId] });
    }
  });
}

// Component usage
function StudentList() {
  const { data, isLoading, error } = useStudents(gymId, {
    status: 'active',
    page: 1,
    limit: 50
  });

  const createStudent = useCreateStudent(gymId);

  const handleCreate = async (formData: CreateStudentDto) => {
    try {
      await createStudent.mutateAsync(formData);
      toast.success('Student created successfully');
    } catch (error) {
      toast.error('Failed to create student');
    }
  };

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;

  return <StudentTable students={data.data} />;
}
```

### API Response Formats

**Success Response**:
```json
{
  "data": { /* resource or array of resources */ },
  "pagination": {  // For list endpoints
    "page": 1,
    "limit": 50,
    "total": 150,
    "totalPages": 3
  }
}
```

**Error Response**:
```json
{
  "error": "Validation failed",
  "code": "VALIDATION_ERROR",
  "details": [
    {
      "field": "name",
      "message": "Name must be at least 2 characters"
    }
  ]
}
```

## API Versioning Strategy

For MVP, no versioning. If breaking changes needed post-MVP:

```
/api/v1/...  # Current version
/api/v2/...  # New version with breaking changes
```

## Consequences

### Positive
1. RESTful, intuitive API design
2. Type-safe with TypeScript + Zod
3. Built-in authentication/authorization
4. Comprehensive error handling
5. Easy to consume from frontend
6. Future-proof for mobile apps

### Negative
1. More boilerplate than tRPC (acceptable for REST benefits)
2. Manual API client code (mitigated: type generation possible)

## Decision Makers
- Technical Lead, Frontend Team, Backend Team

## Related Decisions
- ADR-002: Technology Stack (Next.js)
- ADR-006: Security Architecture

## References
- [REST API Design Best Practices](https://restfulapi.net/)
- [Next.js API Routes](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)
