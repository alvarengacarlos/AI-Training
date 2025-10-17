# Data Model

## Document Information
- **Version**: 1.0
- **Date**: October 16, 2025
- **Status**: Final

## 1. Introduction

This document defines the conceptual and logical data model for the Muay Thai Gym Management System. It describes all entities, their attributes, relationships, and constraints necessary to support the functional requirements.

### Purpose
- Define data structure and organization
- Establish entity relationships
- Specify data types and constraints
- Support database design and implementation
- Ensure data integrity and consistency

---

## 2. Entity Relationship Overview

### High-Level Entity Diagram

```
┌─────────────┐
│  Root User  │
└──────┬──────┘
       │ manages (1:N)
       │
┌──────▼──────┐       ┌──────────────────┐
│     Gym     │◄──────┤  Administrator   │
└──────┬──────┘       └──────────────────┘
       │ owns (1:1)
       │
       │ contains (1:N)
       ├───────────────┬──────────────────┬────────────────┐
       │               │                  │                │
┌──────▼──────┐ ┌─────▼──────┐  ┌────────▼────┐  ┌───────▼────────┐
│   Teacher   │ │   Student  │  │    Class    │  │  PaymentRecord │
└──────┬──────┘ └─────┬──────┘  └────────┬────┘  └───────┬────────┘
       │               │                  │                │
       │ assigned to   │ enrolled in      │                │ for student
       │ (1:N)         │ (M:N)            │                │ (1:N)
       │               │                  │                │
       └───────────────┴──────────────────┘                │
                       │                                   │
                       └───────────────────────────────────┘
```

### Entity Summary

| Entity | Description | Key Relationships |
|--------|-------------|-------------------|
| Root User | System administrator | Manages multiple Gyms |
| Gym | Physical gym location | Has Administrators, Teachers, Students, Classes |
| Administrator | Gym manager user | Manages one Gym |
| Teacher | Martial arts instructor | Belongs to Gym, Assigned to Classes |
| Student | Gym member | Belongs to Gym, Enrolled in Classes, Has Payments |
| Class | Scheduled training session | Belongs to Gym, Has Teacher, Has Students |
| Payment Record | Payment transaction | Belongs to Student |
| Work Schedule | Teacher availability | Belongs to Teacher |
| Enrollment | Student-Class relationship | Links Student to Class |
| Notification Log | Sent notifications | Belongs to Student |

---

## 3. Detailed Entity Specifications

### 3.1 Root User

**Description**: System-level administrator with full access to all gyms.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| root_user_id | UUID | PK, NOT NULL | Unique identifier |
| username | String(50) | UNIQUE, NOT NULL | Login username |
| password_hash | String(255) | NOT NULL | Hashed password (bcrypt) |
| email | String(100) | UNIQUE | Contact email |
| created_at | Timestamp | NOT NULL | Account creation timestamp |
| last_login | Timestamp | NULL | Last login timestamp |
| is_active | Boolean | NOT NULL, DEFAULT TRUE | Account status |

**Business Rules**:
- Username must be unique across all users
- Password must be hashed before storage
- Cannot be deleted if active gyms exist
- Has unrestricted access to all system features

**Related Entities**:
- Gym (1:N) - Can manage multiple gyms

---

### 3.2 Gym

**Description**: Physical gym location with isolated data.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| gym_id | UUID | PK, NOT NULL | Unique identifier |
| name | String(100) | NOT NULL | Gym name |
| address | Text | NULL | Full address |
| owner_name | String(100) | NOT NULL | Owner's name |
| icon_url | String(500) | NULL | URL to gym icon/logo |
| created_at | Timestamp | NOT NULL | Creation timestamp |
| updated_at | Timestamp | NOT NULL | Last update timestamp |
| is_active | Boolean | NOT NULL, DEFAULT TRUE | Active status |

**Business Rules**:
- Gym name should be unique (soft constraint)
- Icon must be valid image format (JPEG, PNG, max 2MB)
- Cannot be deleted if has active students or classes
- All child entities must be deleted/archived before gym deletion

**Related Entities**:
- Administrator (1:N) - Has administrators
- Teacher (1:N) - Contains teachers
- Student (1:N) - Contains students
- Class (1:N) - Contains classes

---

### 3.3 Administrator

**Description**: Gym manager with access to single gym.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| admin_id | UUID | PK, NOT NULL | Unique identifier |
| gym_id | UUID | FK, NOT NULL | Associated gym |
| username | String(50) | UNIQUE, NOT NULL | Login username |
| password_hash | String(255) | NOT NULL | Hashed password (bcrypt) |
| email | String(100) | NULL | Contact email |
| full_name | String(100) | NULL | Administrator's name |
| created_at | Timestamp | NOT NULL | Account creation timestamp |
| last_login | Timestamp | NULL | Last login timestamp |
| is_active | Boolean | NOT NULL, DEFAULT TRUE | Account status |

**Business Rules**:
- Username must be unique across all users
- Can only access data for assigned gym
- Password must be hashed before storage
- Cannot be deleted if only administrator for gym
- Must have valid gym_id reference

**Related Entities**:
- Gym (N:1) - Belongs to one gym

---

### 3.4 Teacher

**Description**: Martial arts instructor assigned to classes.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| teacher_id | UUID | PK, NOT NULL | Unique identifier |
| gym_id | UUID | FK, NOT NULL | Associated gym |
| name | String(100) | NOT NULL | Teacher's full name |
| phone | String(20) | NOT NULL | Contact phone (Brazilian format) |
| address | Text | NULL | Full address |
| birth_date | Date | NOT NULL | Date of birth |
| created_at | Timestamp | NOT NULL | Record creation timestamp |
| updated_at | Timestamp | NOT NULL | Last update timestamp |
| is_active | Boolean | NOT NULL, DEFAULT TRUE | Active status |

**Business Rules**:
- Must be at least 18 years old (birth_date validation)
- Phone must be valid Brazilian format
- Cannot be deleted if assigned to active classes
- Same person can exist as separate teacher in multiple gyms
- Birth date must be in the past

**Related Entities**:
- Gym (N:1) - Belongs to one gym
- Work Schedule (1:N) - Has work schedule
- Class (1:N) - Assigned to classes

---

### 3.5 Work Schedule

**Description**: Teacher's available days and hours.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| schedule_id | UUID | PK, NOT NULL | Unique identifier |
| teacher_id | UUID | FK, NOT NULL | Associated teacher |
| day_of_week | Integer | NOT NULL, 1-5 | Day (1=Mon, 5=Fri) |
| start_time | Time | NOT NULL | Availability start time |
| end_time | Time | NOT NULL | Availability end time |
| created_at | Timestamp | NOT NULL | Record creation timestamp |
| updated_at | Timestamp | NOT NULL | Last update timestamp |

**Business Rules**:
- day_of_week: 1=Monday, 2=Tuesday, 3=Wednesday, 4=Thursday, 5=Friday
- end_time must be after start_time
- Teacher can have multiple schedule entries per week
- No overlapping time slots for same teacher on same day
- Used to validate class assignments

**Related Entities**:
- Teacher (N:1) - Belongs to one teacher

---

### 3.6 Student

**Description**: Gym member enrolled in classes.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| student_id | UUID | PK, NOT NULL | Unique identifier |
| gym_id | UUID | FK, NOT NULL | Associated gym |
| name | String(100) | NOT NULL | Student's full name |
| phone | String(20) | NOT NULL | Contact phone (Brazilian format) |
| address | Text | NULL | Full address |
| birth_date | Date | NOT NULL | Date of birth |
| monthly_price | Decimal(10,2) | NOT NULL | Current monthly price (BRL) |
| billing_cycle_day | Integer | NOT NULL, 1-31 | Day of month payment due |
| created_at | Timestamp | NOT NULL | Record creation timestamp |
| updated_at | Timestamp | NOT NULL | Last update timestamp |
| is_active | Boolean | NOT NULL, DEFAULT TRUE | Active status |

**Business Rules**:
- monthly_price must be positive
- billing_cycle_day between 1-31 (handles month-end appropriately)
- Phone must be valid Brazilian format
- Birth date must be in the past
- Same person can exist as separate student in multiple gyms
- Price changes don't affect historical payment records

**Related Entities**:
- Gym (N:1) - Belongs to one gym
- Enrollment (1:N) - Enrolled in classes
- Payment Record (1:N) - Has payment history

---

### 3.7 Class

**Description**: Scheduled training session with specific attributes.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| class_id | UUID | PK, NOT NULL | Unique identifier |
| gym_id | UUID | FK, NOT NULL | Associated gym |
| teacher_id | UUID | FK, NOT NULL | Assigned teacher |
| name | String(100) | NOT NULL | Class name/type |
| room | String(50) | NOT NULL | Room or area designation |
| max_capacity | Integer | NOT NULL | Maximum students |
| class_time | Time | NOT NULL | Class start time (HH:MM) |
| duration_minutes | Integer | NOT NULL | Class duration |
| created_at | Timestamp | NOT NULL | Record creation timestamp |
| updated_at | Timestamp | NOT NULL | Last update timestamp |
| is_active | Boolean | NOT NULL, DEFAULT TRUE | Active status |

**Business Rules**:
- max_capacity must be positive integer
- Cannot reduce capacity below current enrollment count
- Teacher must be available per work schedule
- Same teacher cannot be assigned to overlapping time slots
- duration_minutes typically 60-120 minutes
- Class times validated against teacher's work schedule

**Related Entities**:
- Gym (N:1) - Belongs to one gym
- Teacher (N:1) - Has assigned teacher
- Class Schedule (1:N) - Has schedule (days of week)
- Enrollment (1:N) - Has enrolled students

---

### 3.8 Class Schedule

**Description**: Days of week a class occurs.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| class_schedule_id | UUID | PK, NOT NULL | Unique identifier |
| class_id | UUID | FK, NOT NULL | Associated class |
| day_of_week | Integer | NOT NULL, 1-7 | Day (1=Mon, 7=Sun) |

**Business Rules**:
- day_of_week: 1=Monday, 2=Tuesday, 3=Wednesday, 4=Thursday, 5=Friday, 6=Saturday, 7=Sunday
- Class can have multiple days (e.g., Mon/Wed/Fri)
- No duplicate days for same class
- Must match teacher's work schedule availability

**Related Entities**:
- Class (N:1) - Belongs to one class

---

### 3.9 Enrollment

**Description**: Many-to-many relationship between students and classes.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| enrollment_id | UUID | PK, NOT NULL | Unique identifier |
| student_id | UUID | FK, NOT NULL | Enrolled student |
| class_id | UUID | FK, NOT NULL | Enrolled class |
| enrollment_date | Date | NOT NULL | Date of enrollment |
| created_at | Timestamp | NOT NULL | Record creation timestamp |
| is_active | Boolean | NOT NULL, DEFAULT TRUE | Active enrollment status |

**Business Rules**:
- Unique constraint on (student_id, class_id) for active enrollments
- Cannot enroll if class at max capacity
- Student can be enrolled in multiple classes simultaneously
- Enrollment date defaults to current date
- Soft delete (is_active=false) instead of hard delete for history

**Related Entities**:
- Student (N:1) - Belongs to one student
- Class (N:1) - Belongs to one class

---

### 3.10 Payment Record

**Description**: Record of student payment transactions.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| payment_id | UUID | PK, NOT NULL | Unique identifier |
| student_id | UUID | FK, NOT NULL | Student who paid |
| payment_date | Date | NOT NULL | Date payment made |
| amount | Decimal(10,2) | NOT NULL | Payment amount (BRL) |
| payment_method | Enum | NOT NULL | Payment method |
| reference_month | Date | NOT NULL | Month/year this payment covers |
| recorded_by | UUID | FK, NULL | Administrator who recorded |
| created_at | Timestamp | NOT NULL | Record creation timestamp |
| notes | Text | NULL | Optional payment notes |

**Enums**:
- payment_method: 'CASH', 'BANK_TRANSFER', 'DEBIT_CARD', 'CREDIT_CARD', 'OTHER'

**Business Rules**:
- amount must be positive
- payment_date cannot be in future
- Payment records are immutable (no updates, only corrections via new records)
- reference_month indicates which billing period payment covers
- Multiple payments can exist for same reference_month (partial payments)
- Historical records retained indefinitely for financial compliance

**Related Entities**:
- Student (N:1) - Belongs to one student
- Administrator (N:1) - Recorded by administrator

---

### 3.11 Notification Log (Nice-to-Have)

**Description**: Log of notifications sent to students.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| notification_id | UUID | PK, NOT NULL | Unique identifier |
| student_id | UUID | FK, NOT NULL | Student notified |
| notification_type | Enum | NOT NULL | Type of notification |
| channel | Enum | NOT NULL | Delivery channel |
| message | Text | NOT NULL | Notification content |
| sent_at | Timestamp | NOT NULL | Send timestamp |
| status | Enum | NOT NULL | Delivery status |
| error_message | Text | NULL | Error details if failed |
| triggered_by | UUID | FK, NULL | Admin who triggered (manual) |

**Enums**:
- notification_type: 'PAYMENT_REMINDER', 'PAYMENT_OVERDUE', 'GENERAL'
- channel: 'SMS', 'WHATSAPP'
- status: 'SENT', 'FAILED', 'PENDING'

**Business Rules**:
- Notifications are immutable log records
- Failed notifications can be retried (new record created)
- Automatic deduplication (no duplicate notifications same day)
- Retained for minimum 90 days

**Related Entities**:
- Student (N:1) - Belongs to one student
- Administrator (N:1) - Triggered by administrator (if manual)

---

### 3.12 Notification Settings (Nice-to-Have)

**Description**: Gym-level notification configuration.

**Attributes**:

| Attribute | Data Type | Constraints | Description |
|-----------|-----------|-------------|-------------|
| settings_id | UUID | PK, NOT NULL | Unique identifier |
| gym_id | UUID | FK, NOT NULL, UNIQUE | Associated gym |
| enabled | Boolean | NOT NULL, DEFAULT FALSE | Notifications enabled |
| mode | Enum | NOT NULL | Notification mode |
| channels | Array | NOT NULL | Enabled channels |
| updated_at | Timestamp | NOT NULL | Last update timestamp |
| updated_by | UUID | FK, NOT NULL | Admin who updated |

**Enums**:
- mode: 'AUTOMATED', 'MANUAL'
- channels: Array of ['SMS', 'WHATSAPP']

**Business Rules**:
- One settings record per gym
- At least one channel must be enabled if notifications enabled
- Settings changes take effect immediately

**Related Entities**:
- Gym (1:1) - One settings record per gym
- Administrator (N:1) - Updated by administrator

---

## 4. Data Integrity Constraints

### 4.1 Primary Keys
- All entities use UUID as primary key for global uniqueness
- UUIDs prevent enumeration attacks and support distributed systems

### 4.2 Foreign Keys
- All foreign key relationships enforce referential integrity
- Cascade rules:
  - Gym deletion: Prevent if has active data (require cleanup first)
  - Teacher deletion: Prevent if assigned to active classes
  - Student deletion: Cascade to enrollments, retain payment records
  - Class deletion: Cascade to enrollments and class schedules
  - Administrator deletion: Prevent if only admin for gym

### 4.3 Unique Constraints
- Username: Unique across Root Users and Administrators
- (student_id, class_id): Unique for active enrollments
- gym_id in Notification Settings: Unique (one settings per gym)

### 4.4 Check Constraints
- monthly_price > 0
- max_capacity > 0
- amount > 0
- billing_cycle_day BETWEEN 1 AND 31
- day_of_week BETWEEN 1 AND 7
- end_time > start_time
- birth_date < CURRENT_DATE
- payment_date <= CURRENT_DATE
- Birth date implies minimum age 18 for teachers

### 4.5 Default Values
- created_at: CURRENT_TIMESTAMP
- updated_at: CURRENT_TIMESTAMP
- is_active: TRUE
- Notification Settings enabled: FALSE

---

## 5. Indexes

### 5.1 Primary Indexes
- Primary key indexes on all entity ID fields (automatic)

### 5.2 Foreign Key Indexes
- Index on all foreign key columns for join performance:
  - admin.gym_id
  - teacher.gym_id
  - student.gym_id
  - class.gym_id
  - class.teacher_id
  - enrollment.student_id
  - enrollment.class_id
  - payment_record.student_id
  - work_schedule.teacher_id
  - class_schedule.class_id

### 5.3 Query Optimization Indexes
- (username) - Login queries
- (gym_id, is_active) - Active entity queries per gym
- (student_id, payment_date) - Payment history queries
- (student_id, billing_cycle_day) - Payment status calculations
- (class_id, is_active) - Active enrollments per class
- (teacher_id, day_of_week) - Teacher schedule lookups
- (notification_log.student_id, sent_at) - Notification history

### 5.4 Composite Indexes
- (gym_id, created_at) - Chronological queries per gym
- (student_id, enrollment_date) - Student enrollment history
- (teacher_id, day_of_week, start_time) - Schedule conflict detection

---

## 6. Data Volume Estimates

### 6.1 MVP Estimates (Per Gym)
- Teachers: 5-20 (average 10)
- Students: 50-500 (average 200)
- Classes: 10-50 (average 20)
- Enrollments: 100-1000 (average 400)
- Payment Records per Student per Year: 12
- Payment Records per Gym per Year: 2,400

### 6.2 System-Wide Estimates (100 Gyms)
- Total Gyms: 100
- Total Teachers: 1,000
- Total Students: 20,000
- Total Classes: 2,000
- Total Enrollments: 40,000
- Total Payment Records per Year: 240,000
- Total Payment Records after 5 Years: 1,200,000

### 6.3 Storage Estimates
- Average record sizes:
  - Gym: 1 KB
  - Teacher: 2 KB
  - Student: 2 KB
  - Class: 1 KB
  - Enrollment: 0.5 KB
  - Payment Record: 0.5 KB

- Total storage (100 gyms, 5 years):
  - Core entities: ~50 MB
  - Payment records: ~600 MB
  - Indexes: ~200 MB
  - Total: <1 GB

---

## 7. Data Security and Privacy

### 7.1 Encryption Requirements
**Encrypted at Rest**:
- Student personal data (name, address, phone)
- Teacher personal data (name, address, phone)
- Administrator personal data
- Payment information

**Encrypted in Transit**:
- All data transmission via HTTPS/TLS 1.2+

### 7.2 Access Controls
- Row-level security enforcing gym_id filtering
- Administrator can only query their gym's data
- Root User can query any gym's data
- Application-level enforcement of data isolation

### 7.3 Sensitive Data Handling
- Passwords: Hashed with bcrypt (cost factor 10+)
- Personal data: Encrypted with AES-256
- Payment amounts: Stored as-is (not sensitive)
- Phone numbers: Encrypted, indexed by hash for search

### 7.4 Audit Trail
- created_at and updated_at on all entities
- Payment records immutable (audit trail of changes)
- Notification logs retained for compliance
- User activity logged separately

---

## 8. Data Migration and Versioning

### 8.1 Schema Versioning
- Database migrations tracked and versioned
- Backward-compatible changes preferred
- Rollback scripts for all migrations
- Migration testing in staging environment

### 8.2 Data Seeding
- Initial root user account
- Sample gym for testing
- Reference data (payment methods, etc.)

### 8.3 Historical Data
- Payment records retained indefinitely
- Soft deletes for students/teachers (is_active flag)
- Enrollment history preserved
- Class changes tracked via updated_at

---

## 9. Data Quality Rules

### 9.1 Validation Rules
- Phone numbers: Brazilian format validation
- Email: RFC 5322 format validation
- Dates: Logical consistency (birth date < today)
- Monetary values: Maximum 2 decimal places
- Names: Minimum 2 characters, maximum 100 characters

### 9.2 Business Logic Constraints
- Student cannot enroll in same class twice (active)
- Class cannot exceed max_capacity
- Teacher assignment must respect work schedule
- Payment date cannot be future
- Monthly price must be positive

### 9.3 Data Cleanup
- Inactive records retained for historical reference
- Hard delete only for test/demo data
- Personal data deletion on request (LGPD compliance)
- Backup retention: 30 days

---

## 10. Entity Relationship Diagram (ERD)

### Detailed ERD Notation

```
┌─────────────────────────────────────────────────────────────┐
│ ROOT_USER                                                   │
├─────────────────────────────────────────────────────────────┤
│ PK  root_user_id         UUID                              │
│     username              VARCHAR(50)  UNIQUE              │
│     password_hash         VARCHAR(255)                     │
│     email                 VARCHAR(100) UNIQUE              │
│     created_at            TIMESTAMP                        │
│     last_login            TIMESTAMP                        │
│     is_active             BOOLEAN                          │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ 1:N manages
                            │
┌─────────────────────────────────────────────────────────────┐
│ GYM                                                         │
├─────────────────────────────────────────────────────────────┤
│ PK  gym_id               UUID                              │
│     name                  VARCHAR(100)                     │
│     address               TEXT                             │
│     owner_name            VARCHAR(100)                     │
│     icon_url              VARCHAR(500)                     │
│     created_at            TIMESTAMP                        │
│     updated_at            TIMESTAMP                        │
│     is_active             BOOLEAN                          │
└─────────────────────────────────────────────────────────────┘
       │                │               │               │
       │ 1:N            │ 1:N           │ 1:N           │ 1:1
       │                │               │               │
┌──────▼──────┐  ┌──────▼──────┐  ┌────▼─────┐  ┌─────▼───────────┐
│ADMINISTRATOR│  │  TEACHER    │  │ STUDENT  │  │NOTIFICATION_    │
│             │  │             │  │          │  │   SETTINGS      │
└─────────────┘  └──────┬──────┘  └────┬─────┘  └─────────────────┘
                        │               │
                        │ 1:N           │ 1:N
                        │               │
                 ┌──────▼──────┐  ┌─────▼────────┐
                 │WORK_SCHEDULE│  │PAYMENT_RECORD│
                 └─────────────┘  └──────────────┘
                                         │
                                         │
┌─────────────────────────────────────┐  │
│ CLASS                                │  │
├──────────────────────────────────────┤  │
│ PK  class_id          UUID          │  │
│ FK  gym_id            UUID          │  │
│ FK  teacher_id        UUID          │  │
│     name               VARCHAR(100) │  │
│     room               VARCHAR(50)  │  │
│     max_capacity       INTEGER      │  │
│     class_time         TIME         │  │
│     duration_minutes   INTEGER      │  │
│     created_at         TIMESTAMP    │  │
│     updated_at         TIMESTAMP    │  │
│     is_active          BOOLEAN      │  │
└─────────────────────────────────────┘  │
       │                │                │
       │ 1:N            │ 1:N            │
       │                │                │
┌──────▼─────────┐  ┌───▼──────────┐    │
│CLASS_SCHEDULE  │  │ ENROLLMENT   │◄───┘
│                │  │              │ N:1
│ days of week   │  │ M:N Student  │
└────────────────┘  │   to Class   │
                    └──────────────┘
```

---

## 11. Data Model Summary

### Total Entities: 11
1. Root User
2. Gym
3. Administrator
4. Teacher
5. Work Schedule
6. Student
7. Class
8. Class Schedule
9. Enrollment
10. Payment Record
11. Notification Log (nice-to-have)
12. Notification Settings (nice-to-have)

### Total Relationships: 14
- Root User → Gym (1:N)
- Gym → Administrator (1:N)
- Gym → Teacher (1:N)
- Gym → Student (1:N)
- Gym → Class (1:N)
- Gym → Notification Settings (1:1)
- Teacher → Work Schedule (1:N)
- Teacher → Class (1:N)
- Student → Enrollment (1:N)
- Student → Payment Record (1:N)
- Student → Notification Log (1:N)
- Class → Class Schedule (1:N)
- Class → Enrollment (1:N)
- Administrator → Payment Record (1:N, recorded_by)

---

**Document Revision History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-10-16 | Requirements Team | Initial version |
