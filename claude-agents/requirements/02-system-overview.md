# System Overview

## Document Information
- **Version**: 1.0
- **Date**: October 16, 2025
- **Status**: Final

## 1. System Purpose

The Muay Thai Gym Management System (MTGMS) is a multi-tenant Software-as-a-Service (SaaS) web application that provides comprehensive administrative tools for Muay Thai gym operations. The system enables gym administrators to manage all aspects of their gym operations from a centralized, cloud-based platform accessible from any device.

### Primary Purpose
To streamline and automate the administrative operations of Muay Thai gyms, specifically focusing on:
- Student enrollment and class participation management
- Teacher scheduling and class assignments
- Payment tracking with flexible, per-student billing
- Multi-location gym management

### Target Users
- Small to medium-sized Muay Thai gyms in Brazil
- Multi-location gym chains and franchises
- Independent gym owners seeking affordable management tools

## 2. System Context

### Business Context

The MTGMS operates within the Brazilian martial arts and fitness industry, serving gym owners who need professional management tools without enterprise-level complexity or cost. The system addresses the gap between manual management (spreadsheets, paper records) and expensive, over-featured enterprise solutions.

#### Market Position
- **Target Market**: Brazilian Muay Thai gyms with 50-500 students
- **Pricing Model**: Affordable SaaS subscription
- **Differentiation**: Purpose-built for martial arts operations, not generic fitness/CRM software
- **Compliance**: LGPD-compliant from inception, addressing Brazilian regulatory requirements

### Technical Context

```
┌─────────────────────────────────────────────────────────────┐
│                     End Users (Browsers)                    │
│  Firefox / Chrome / Edge (Desktop & Mobile Responsive)     │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTPS
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  MTGMS Web Application                      │
│                   (Cloud-Hosted SaaS)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Presentation Layer (Responsive Web Interface)       │  │
│  └──────────────────┬───────────────────────────────────┘  │
│  ┌──────────────────▼───────────────────────────────────┐  │
│  │  Business Logic Layer                                │  │
│  │  - Multi-tenant isolation                            │  │
│  │  - Authentication & authorization                    │  │
│  │  - Gym/Teacher/Student/Class/Payment management      │  │
│  └──────────────────┬───────────────────────────────────┘  │
│  ┌──────────────────▼───────────────────────────────────┐  │
│  │  Data Layer (Encrypted Storage)                      │  │
│  │  - Multi-tenant database                             │  │
│  │  - LGPD-compliant data handling                      │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
┌────────▼─────────┐   ┌─────────▼──────────┐
│  SMS Gateway     │   │  WhatsApp API      │
│  (Nice-to-Have)  │   │  (Nice-to-Have)    │
└──────────────────┘   └────────────────────┘
```

### System Boundaries

#### Within System Boundaries
- User authentication and session management
- Multi-tenant data isolation
- Gym profile management
- Teacher profile and schedule management
- Student profile and enrollment management
- Class definition and capacity management
- Payment tracking and status calculation
- Dashboard and schedule visualization
- Notification dispatch (nice-to-have)

#### Outside System Boundaries (External Dependencies)
- Payment processing (gyms handle offline)
- SMS delivery (external SMS gateway)
- WhatsApp message delivery (external WhatsApp API)
- Email services (if needed for system notifications)
- Browser rendering engines
- User devices and internet connectivity

#### Explicitly Out of Scope
- Attendance check-in/check-out
- Payment collection and processing
- Student/teacher mobile applications
- Advanced analytics and reporting
- Inventory management
- Point-of-sale functionality
- Marketing automation
- Customer relationship management features

## 3. System Architecture Principles

### Multi-Tenancy Model
The system employs a shared infrastructure, isolated data architecture:
- Single application instance serves all gyms
- Each gym's data is completely isolated from others
- Root users can access multiple gyms
- Gym administrators can only access their assigned gym
- Teachers and students belong to specific gyms but can be registered in multiple gyms independently

### Data Isolation Strategy
```
Root User
├── Gym A
│   ├── Administrators (Gym A)
│   ├── Teachers (Gym A)
│   ├── Students (Gym A)
│   ├── Classes (Gym A)
│   └── Payments (Gym A)
├── Gym B
│   ├── Administrators (Gym B)
│   ├── Teachers (Gym B)
│   ├── Students (Gym B)
│   ├── Classes (Gym B)
│   └── Payments (Gym B)
└── Gym C
    └── [Same structure as Gym A/B]
```

### Security Architecture
- Role-based access control (RBAC)
  - Root User: Full system access
  - Gym Administrator: Single gym access
- Authentication via username/password
- Session management with timeout
- Data encryption at rest
- HTTPS/TLS for data in transit
- LGPD-compliant data handling

## 4. Key System Capabilities

### Gym Management
- Create and configure gym profiles (name, address, icon, owner)
- Multi-gym support for chains/franchises
- Data isolation between gyms

### Teacher Management
- Store teacher profiles (name, phone, address, birth date)
- Define teacher work schedules (Monday-Friday, flexible hours)
- Assign teachers to specific class time slots
- Support teachers working at multiple gyms (separate registrations)

### Student Management
- Store student profiles (name, phone, address, birth date)
- Assign variable monthly pricing per student
- Enroll students in multiple classes simultaneously
- Track enrollment history
- Support students attending multiple gyms (separate registrations)

### Class Management
- Define class attributes (name/type, schedule, room, capacity)
- Assign teachers to classes
- Manage student enrollments
- Enforce configurable capacity limits
- Track class schedules (days of week, time slots)

### Payment Management
- Track payment history per student
- Record payment date, amount, and method
- Calculate payment status based on due dates
- Visual indicators for overdue payments (red mark)
- Support variable billing cycles per student
- Support variable pricing that can change over time
- Record payment methods for reference (cash, bank transfer, card)

### Dashboard & Reporting
- Display total student count per gym
- Show class schedules
- Provide at-a-glance operational overview

### Notification System (Nice-to-Have)
- Send payment reminders via SMS or WhatsApp
- Support automated or manual notification modes
- Admin-configurable notification settings

## 5. User Roles and Access Levels

### Role Hierarchy

```
┌─────────────────────────────────────────────┐
│           Root User (Superuser)             │
│  - Full system access                       │
│  - Can add new gyms                         │
│  - No restrictions                          │
└─────────────────┬───────────────────────────┘
                  │
      ┌───────────┴───────────┐
      │                       │
┌─────▼──────────────┐  ┌─────▼─────────────┐
│  Gym Administrator │  │ Gym Administrator │
│     (Gym A)        │  │     (Gym B)       │
│  - Manage Gym A    │  │  - Manage Gym B   │
│  - Cannot access   │  │  - Cannot access  │
│    other gyms      │  │    other gyms     │
└────────────────────┘  └───────────────────┘

        [Teachers: No System Access]
        [Students: No System Access]
```

### Detailed Role Permissions

#### Root User
- **Description**: System administrator with unlimited access
- **Access Level**: ALL operations across ALL gyms
- **Key Permissions**:
  - Create new gyms
  - Delete gyms
  - Assign administrators to gyms
  - Access any gym's data
  - Perform any operation without restriction

#### Gym Administrator
- **Description**: Gym manager responsible for daily operations
- **Access Level**: Full access to assigned gym ONLY
- **Key Permissions**:
  - Manage gym profile
  - Create/update/delete teachers
  - Create/update/delete students
  - Create/update/delete classes
  - Enroll/unenroll students in classes
  - Record and track payments
  - View dashboard and schedules
  - Configure notification settings
  - Cannot access other gyms' data
  - Cannot create new gyms

#### Teacher (No System Access)
- **Description**: Martial arts instructor assigned to classes
- **System Access**: NONE - Teachers do not log in to the system
- **Management**: Completely managed by gym administrators
- **Data Stored**: Profile information, work schedule, class assignments
- **Visibility**: Teachers may see their schedule through admin (not direct access)

#### Student (No System Access)
- **Description**: Gym member enrolled in classes
- **System Access**: NONE - Students do not log in to the system
- **Management**: Completely managed by gym administrators
- **Data Stored**: Profile information, class enrollments, payment history
- **Visibility**: Students may receive notifications but cannot access system

## 6. System Constraints

### Technical Constraints
- **Platform**: Web application only (no native mobile apps)
- **Browsers**: Must support Firefox, Edge, Google Chrome (current versions)
- **Responsive Design**: Must function on mobile devices via responsive web design
- **Concurrent Users**: Maximum 100 simultaneous users
- **Budget**: Infrastructure costs must not exceed $100/month
- **No Integration**: No third-party payment processing integration

### Operational Constraints
- **User Authentication**: Username/password only (no SSO, no MFA)
- **Offline Access**: Not required (online-only system)
- **Payment Processing**: Manual/offline only (no automated payment collection)
- **Notification Delivery**: Depends on third-party SMS/WhatsApp services

### Regulatory Constraints
- **LGPD Compliance**: Must comply with Brazilian data protection law
- **Data Encryption**: Required for sensitive personal data
- **Data Retention**: Must maintain historical payment records
- **Data Privacy**: Must implement proper data isolation between gyms

### Business Constraints
- **Scope**: MVP functionality only (advanced features deferred)
- **Languages**: Portuguese (Brazil) - primary language
- **Time Zone**: Brazilian Time (BRT/BRST)
- **Currency**: Brazilian Real (BRL)

## 7. Key Assumptions

### User Assumptions
- Gym administrators have basic computer literacy
- Administrators have internet-connected devices (desktop or mobile)
- Administrators understand their gym's business processes
- Students and teachers will not require direct system access

### Technical Assumptions
- Users have modern web browsers (last 2 major versions)
- Internet connectivity is generally reliable
- SMS/WhatsApp gateways will handle message delivery (when implemented)
- Cloud infrastructure will provide adequate performance within budget

### Business Assumptions
- Gyms manage payments offline (cash, bank transfer, etc.)
- Gym administrators will manually record payments in the system
- Class schedules follow weekly patterns (same days/times each week)
- Gyms operate primarily Monday-Friday with occasional weekend classes
- Student payment cycles remain relatively stable (not changing constantly)

### Data Assumptions
- Each gym has fewer than 500 students
- Each gym has fewer than 20 teachers
- Each gym offers fewer than 50 distinct classes
- Payment history retention: indefinite (no automatic deletion)

## 8. Dependencies

### External Service Dependencies
- **Cloud Hosting Provider**: Required for application hosting
- **Database Service**: Required for data storage
- **SMS Gateway** (nice-to-have): For sending SMS notifications
- **WhatsApp Business API** (nice-to-have): For WhatsApp notifications
- **SSL/TLS Certificate Provider**: For HTTPS encryption

### Internal Dependencies
- **Authentication System**: Required before any other functionality
- **Multi-Tenant Infrastructure**: Required before gym-specific features
- **Payment Tracking**: Depends on student management
- **Class Enrollment**: Depends on student and class management
- **Dashboard**: Depends on all core entities (students, classes, payments)

## 9. Success Metrics

### Performance Metrics
- Page load time < 3 seconds (95th percentile)
- Support 100 concurrent users
- 99% uptime during business hours (6am-10pm BRT)
- Database query response time < 500ms

### Business Metrics
- Time to enroll a student: < 2 minutes
- Time to record a payment: < 1 minute
- Payment status visibility: 100% immediate
- Administrative task time reduction: 50%

### User Satisfaction Metrics
- Gym administrator can complete common tasks without training
- System perceived as easier than previous methods
- Zero data loss incidents
- Positive feedback on usability and reliability

## 10. Future Expansion Opportunities

While out of scope for MVP, the system architecture should accommodate:
- Attendance tracking and reporting
- Payment processing integrations
- Student/teacher self-service portals
- Advanced analytics and business intelligence
- Multi-factor authentication
- Mobile native applications
- Automated marketing campaigns
- Inventory management
- Contract management

---

**Document Revision History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-10-16 | Requirements Team | Initial version |
