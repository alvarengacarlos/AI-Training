# ADR 001: Application Architecture Pattern

## Status
Accepted

## Date
2025-10-16

## Context

The Muay Thai Gym Management System (MTGMS) requires an architectural approach that balances:

- **Budget constraint**: Maximum $100/month infrastructure cost
- **Team size**: Small development team (1-3 developers)
- **Timeline**: MVP delivery in 15 weeks (~3.75 months)
- **Complexity**: Multi-tenant SaaS with 39 functional requirements (35 must-have)
- **Scale**: 100 concurrent users, 100 gyms, ~20,000 students
- **Maintainability**: Long-term support with minimal team

### Key Requirements Driving Architecture

1. **Multi-tenancy**: Complete data isolation between gyms (FR-006)
2. **Web-only**: Responsive web application, no native mobile apps (CONSTRAINT-004)
3. **Budget**: Infrastructure costs under $100/month (CONSTRAINT-001)
4. **Performance**: Page load < 3s, support 100 concurrent users (NFR-001, CONSTRAINT-008)
5. **Team**: Small team must develop and maintain system (CONSTRAINT-026)
6. **LGPD Compliance**: Brazilian data protection requirements (NFR-032)

### Architectural Options Considered

#### Option 1: Microservices Architecture
**Description**: Decompose system into independent services (User Service, Gym Service, Payment Service, etc.)

**Pros**:
- Independent scaling of services
- Technology diversity per service
- Team can work independently on services
- Better fault isolation

**Cons**:
- HIGH operational complexity (service discovery, API gateway, distributed tracing)
- Increased infrastructure costs (multiple deployments, databases, message queues)
- Distributed transactions complexity
- Overkill for 1-3 developer team
- Network latency between services
- Debugging and testing more difficult
- Would likely exceed $100/month budget

**Verdict**: REJECTED - Inappropriate for small team and budget constraints

#### Option 2: Modular Monolith
**Description**: Single deployable application with well-defined internal module boundaries

**Pros**:
- Clear module boundaries with potential future extraction
- Single deployment simplifies operations
- Shared database simplifies transactions
- Lower infrastructure costs
- Easier development and debugging
- Appropriate for small team
- Can refactor to microservices later if needed

**Cons**:
- Entire application deployed together
- Modules share resources (can't independently scale)
- Risk of tight coupling if discipline lacking

**Verdict**: STRONG CANDIDATE - Best balance for MVP

#### Option 3: Pure Monolith (No Module Boundaries)
**Description**: Single application without enforced internal structure

**Pros**:
- Fastest initial development
- Simplest architecture
- Lowest infrastructure cost
- Easy for small team

**Cons**:
- Technical debt accumulates quickly
- Difficult to maintain as system grows
- Tight coupling between features
- Hard to test components in isolation
- Refactoring becomes painful

**Verdict**: REJECTED - Technical debt risk too high

## Decision

**We will implement a MODULAR MONOLITH architecture.**

### Architecture Structure

```
┌─────────────────────────────────────────────────────┐
│         MTGMS Web Application (Monolith)            │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ Presentation │  │ Presentation │  │   API    │ │
│  │  Layer (Web) │  │  Layer (API) │  │  Layer   │ │
│  └──────┬───────┘  └──────┬───────┘  └────┬─────┘ │
│         │                 │                │       │
│  ┌──────┴─────────────────┴────────────────┴─────┐ │
│  │         Application Services Layer            │ │
│  │  (Orchestration, Transaction Management)      │ │
│  └──────┬─────────────────────────────────────────┘ │
│         │                                           │
│  ┌──────┴─────────────────────────────────────┐   │
│  │         Business Logic Modules             │   │
│  │                                            │   │
│  │  ┌─────────┐  ┌─────────┐  ┌───────────┐  │   │
│  │  │  Auth   │  │   Gym   │  │  Teacher  │  │   │
│  │  │ Module  │  │ Module  │  │  Module   │  │   │
│  │  └─────────┘  └─────────┘  └───────────┘  │   │
│  │                                            │   │
│  │  ┌─────────┐  ┌─────────┐  ┌───────────┐  │   │
│  │  │ Student │  │  Class  │  │  Payment  │  │   │
│  │  │ Module  │  │ Module  │  │  Module   │  │   │
│  │  └─────────┘  └─────────┘  └───────────┘  │   │
│  │                                            │   │
│  │  ┌─────────────┐  ┌──────────────────┐    │   │
│  │  │Notification │  │   Multi-Tenancy  │    │   │
│  │  │   Module    │  │      Module      │    │   │
│  │  └─────────────┘  └──────────────────┘    │   │
│  └────────────────────────────────────────────┘   │
│         │                                          │
│  ┌──────┴───────────────────────────────────────┐ │
│  │         Data Access Layer                    │ │
│  │  (Repository Pattern, Multi-Tenant Filtering)│ │
│  └──────┬───────────────────────────────────────┘ │
│         │                                          │
└─────────┼──────────────────────────────────────────┘
          │
          ▼
   ┌──────────────┐
   │  PostgreSQL  │
   │   Database   │
   └──────────────┘
```

### Module Boundaries

Each module encapsulates:
- **Domain Logic**: Business rules and validation
- **Data Models**: Entity definitions
- **Repositories**: Data access interfaces
- **Services**: Module-specific operations
- **API Endpoints**: External interfaces (REST)

### Module Dependencies

```
Auth Module → Multi-Tenancy Module
Gym Module → Auth Module
Teacher Module → Gym Module
Student Module → Gym Module
Class Module → Gym Module, Teacher Module
Payment Module → Student Module
Notification Module → Student Module, Payment Module
```

### Key Architectural Principles

1. **Single Database**: All modules share PostgreSQL database with multi-tenant isolation
2. **Clear Boundaries**: Modules communicate through well-defined interfaces
3. **Dependency Direction**: Dependencies flow inward (outer layers depend on inner)
4. **Transaction Management**: Application layer manages cross-module transactions
5. **Multi-Tenancy First**: Every query/operation includes gym_id filtering
6. **Shared Kernel**: Common utilities (logging, error handling, validation) shared across modules

## Consequences

### Positive

1. **Cost Efficiency**: Single deployment keeps infrastructure costs under $100/month
2. **Development Speed**: Faster development than microservices (appropriate for 15-week timeline)
3. **Team Fit**: 1-3 developers can effectively develop and maintain monolith
4. **Operational Simplicity**: Single deployment, single database, simple monitoring
5. **Performance**: No network hops between modules, faster than distributed architecture
6. **Transactions**: ACID transactions across modules (critical for payment data)
7. **Debugging**: Easier to trace bugs across module boundaries
8. **Testability**: Easier integration testing with all modules together
9. **Future Flexibility**: Can extract modules to services later if needed

### Negative

1. **Scaling Limitations**: Cannot independently scale individual modules (acceptable for 100 concurrent users)
2. **Deployment Coupling**: All modules deployed together (requires comprehensive testing)
3. **Technology Lock-in**: Entire stack must use same technology (acceptable tradeoff)
4. **Discipline Required**: Team must maintain module boundaries to avoid tight coupling
5. **Shared Resources**: Modules compete for resources (database connections, memory)

### Mitigation Strategies

1. **Module Boundaries**: Enforce through code structure and code reviews
2. **Interface Contracts**: Define clear APIs between modules
3. **Automated Testing**: Comprehensive test suite prevents regression when deploying
4. **Monitoring**: Track module performance within monolith to identify bottlenecks
5. **Caching**: Implement caching to reduce database load and improve performance

## Compliance and Security

- **LGPD Compliance**: Multi-tenancy module ensures data isolation between gyms
- **Data Encryption**: Encryption handled at application and database layers
- **Audit Logging**: Centralized logging across all modules
- **Session Management**: Authentication module manages secure sessions

## Performance Considerations

- **Expected Response Times**: < 500ms for API calls (monolith eliminates inter-service latency)
- **Database Optimization**: Proper indexing on gym_id for multi-tenant queries
- **Caching Strategy**: Redis or in-memory caching for frequently accessed data
- **Connection Pooling**: Efficient database connection management

## Deployment Model

- **Single Container**: Entire application in one Docker container
- **Horizontal Scaling**: Multiple application instances behind load balancer (if needed)
- **Database**: Single PostgreSQL instance (managed service)
- **Static Assets**: Served via CDN for performance

## Migration Path

If system outgrows monolith architecture:

1. **Extract Notification Module**: Independent service for async notifications (first candidate)
2. **Extract Payment Module**: If payment processing volume justifies
3. **Extract Reporting Module**: If added post-MVP with heavy analytics workload

Well-defined module boundaries make future extraction feasible without complete rewrite.

## Alternatives Not Chosen

### Why Not Microservices?
- Too complex for 1-3 developer team
- Operational overhead exceeds budget
- Distributed transactions problematic for payment data
- Network latency impacts user experience
- Development timeline too short

### Why Not Serverless Functions?
- Cold start latency unacceptable for user-facing application
- Difficult to manage transactions across functions
- More expensive at consistent load (100 concurrent users)
- Debugging complexity

### Why Not Pure Monolith?
- Technical debt risk
- Difficult to maintain as features grow
- Module boundaries provide structure and discipline

## Decision Makers

- Architecture Team
- Technical Lead
- Project Stakeholders

## Related Decisions

- ADR-002: Technology Stack Selection
- ADR-003: Database Selection and Design
- ADR-004: Deployment Environment and Infrastructure
- ADR-005: Multi-Tenancy Implementation Strategy

## References

- [Modular Monoliths](https://www.kamilgrzybek.com/design/modular-monolith-primer/)
- [Monolith First](https://martinfowler.com/bliki/MonolithFirst.html)
- [The Majestic Monolith](https://m.signalvnoise.com/the-majestic-monolith/)
