# Functional Requirements

## Document Information
- **Version**: 1.0
- **Date**: October 16, 2025
- **Status**: Final

## 1. Introduction

This document specifies all functional requirements for the Muay Thai Gym Management System (MTGMS). Each requirement is uniquely identified, prioritized, and includes specific acceptance criteria for testing and validation.

### Requirement Format

Each requirement follows this structure:
- **ID**: Unique identifier (FR-XXX)
- **Title**: Brief description
- **Priority**: Must-Have / Nice-to-Have
- **Description**: Detailed explanation
- **Acceptance Criteria**: Testable conditions for completion
- **Related Requirements**: Dependencies or related features

### Priority Definitions
- **Must-Have**: Core MVP functionality, system cannot launch without this
- **Nice-to-Have**: Valuable features that can be deferred to post-MVP

---

## 2. Authentication and Authorization

### FR-001: User Authentication
- **Priority**: Must-Have
- **Description**: The system shall provide username and password authentication for Root Users and Gym Administrators. Authentication is required to access any system functionality.
- **Acceptance Criteria**:
  - User can log in with valid username and password
  - System displays error message for invalid credentials
  - System creates session upon successful login
  - System maintains session across page navigation
  - Session expires after 4 hours of inactivity
  - User can log out manually
  - Passwords are hashed and stored securely (not plaintext)
  - Failed login attempts are logged for security monitoring
- **Related Requirements**: All other requirements depend on authentication

### FR-002: Role-Based Access Control
- **Priority**: Must-Have
- **Description**: The system shall enforce role-based access control, ensuring Root Users can access all gyms while Gym Administrators can only access their assigned gym.
- **Acceptance Criteria**:
  - Root User can view and access all gyms in the system
  - Root User can perform any operation in any gym
  - Gym Administrator can only view their assigned gym
  - Gym Administrator cannot access other gyms' data
  - Gym Administrator cannot view gym selection interface (direct to their gym)
  - System denies access with clear error message if user attempts unauthorized action
  - User role is determined at login and persists throughout session
- **Related Requirements**: FR-001, FR-003

### FR-003: Gym Administrator Management
- **Priority**: Must-Have
- **Description**: Root Users shall be able to create and manage Gym Administrator accounts, assigning them to specific gyms.
- **Acceptance Criteria**:
  - Root User can create new administrator account with username and password
  - Root User can assign administrator to a specific gym
  - Root User can update administrator credentials
  - Root User can deactivate/reactivate administrator accounts
  - Root User can delete administrator accounts
  - System prevents duplicate usernames
  - Administrator is notified of account creation (credentials provided securely)
- **Related Requirements**: FR-001, FR-002, FR-004

---

## 3. Gym Management

### FR-004: Create Gym
- **Priority**: Must-Have
- **Description**: Root Users shall be able to create new gym profiles in the system.
- **Acceptance Criteria**:
  - Root User can create gym with required fields: name, address, owner name
  - Root User can upload gym icon image (optional)
  - System validates all required fields before saving
  - System assigns unique identifier to each gym
  - Gym name must be unique within the system
  - System displays confirmation message upon successful creation
  - Newly created gym appears in gym list immediately
- **Related Requirements**: FR-001, FR-002, FR-005

### FR-005: View and Update Gym Profile
- **Priority**: Must-Have
- **Description**: Root Users and Gym Administrators shall be able to view and update gym profile information.
- **Acceptance Criteria**:
  - User can view gym name, address, owner, and icon
  - Gym Administrator can update gym name, address, owner, and icon
  - Root User can update any gym's profile
  - System validates data before saving updates
  - System displays confirmation message after successful update
  - Changes are reflected immediately in the system
  - Icon image must be in supported format (JPEG, PNG, max 2MB)
- **Related Requirements**: FR-004

### FR-006: Multi-Tenant Data Isolation
- **Priority**: Must-Have
- **Description**: The system shall ensure complete data isolation between gyms, preventing any cross-gym data access or contamination.
- **Acceptance Criteria**:
  - All queries automatically filter by gym context
  - Gym Administrator can only see data for their assigned gym
  - Teachers, students, classes, and payments are isolated per gym
  - Root User can switch between gyms and see appropriate filtered data
  - No data leakage between gyms under any circumstances
  - System logs and monitors for potential isolation violations
- **Related Requirements**: FR-001, FR-002, All gym-specific features

---

## 4. Teacher Management

### FR-007: Create Teacher Profile
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to create teacher profiles within their gym.
- **Acceptance Criteria**:
  - Administrator can create teacher with: name, phone, address, birth date
  - All fields are required except address (optional)
  - System validates phone number format (Brazilian format)
  - System validates birth date (must be in the past, minimum age 18)
  - Teacher is automatically associated with the current gym
  - System displays confirmation message upon successful creation
  - Teacher appears in teacher list immediately
- **Related Requirements**: FR-002, FR-006

### FR-008: View and Update Teacher Profile
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to view and update teacher profile information.
- **Acceptance Criteria**:
  - Administrator can view all teacher details
  - Administrator can update teacher name, phone, address, birth date
  - System validates all fields before saving
  - System displays confirmation message after update
  - Changes reflect immediately in all places teacher appears
  - System prevents deletion if teacher is assigned to active classes
- **Related Requirements**: FR-007

### FR-009: Delete Teacher Profile
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to delete teacher profiles.
- **Acceptance Criteria**:
  - Administrator can delete teacher not assigned to any active classes
  - System displays warning before deletion
  - System prevents deletion if teacher has active class assignments
  - System provides clear error message explaining why deletion is blocked
  - Upon deletion, teacher is permanently removed from the system
  - Deleted teacher does not appear in any lists or reports
- **Related Requirements**: FR-007, FR-008, FR-016

### FR-010: Define Teacher Work Schedule
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to define each teacher's available work schedule (Monday-Friday with flexible hours).
- **Acceptance Criteria**:
  - Administrator can set availability for each day (Monday-Friday)
  - Administrator can define start and end time for each available day
  - System supports flexible hours (not fixed shifts)
  - Administrator can mark days as unavailable
  - System validates that end time is after start time
  - Work schedule is used for validating class assignments
  - Changes to schedule are reflected immediately
- **Related Requirements**: FR-007, FR-008, FR-016

### FR-011: Multi-Gym Teacher Registration
- **Priority**: Must-Have
- **Description**: The system shall allow the same person to be registered as a teacher in multiple gyms with separate, independent profiles.
- **Acceptance Criteria**:
  - Same person (by name/phone) can exist as teacher in multiple gyms
  - Each gym maintains separate teacher profile (can have different details)
  - Teacher schedule in Gym A is independent of schedule in Gym B
  - Class assignments are gym-specific
  - No data sharing between teacher profiles across gyms
- **Related Requirements**: FR-006, FR-007

---

## 5. Student Management

### FR-012: Create Student Profile
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to create student profiles within their gym.
- **Acceptance Criteria**:
  - Administrator can create student with: name, phone, address, birth date, monthly price
  - All fields are required except address (optional)
  - System validates phone number format (Brazilian format)
  - System validates birth date (must be in the past)
  - Monthly price must be a positive number in BRL
  - Student is automatically associated with the current gym
  - System displays confirmation message upon successful creation
  - Student appears in student list immediately
- **Related Requirements**: FR-002, FR-006

### FR-013: View and Update Student Profile
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to view and update student profile information.
- **Acceptance Criteria**:
  - Administrator can view all student details
  - Administrator can update student name, phone, address, birth date
  - Administrator can update monthly price (historical prices are not changed)
  - System validates all fields before saving
  - System displays confirmation message after update
  - Changes reflect immediately throughout the system
  - Price changes apply to future payments only
- **Related Requirements**: FR-012

### FR-014: Delete Student Profile
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to delete student profiles.
- **Acceptance Criteria**:
  - Administrator can delete any student
  - System displays warning before deletion
  - System asks for confirmation if student has payment history
  - Upon deletion, student is permanently removed
  - Student's payment history is retained for record-keeping
  - Deleted student is removed from all class enrollments
  - Deleted student does not appear in student lists
- **Related Requirements**: FR-012, FR-013, FR-023

### FR-015: Multi-Gym Student Registration
- **Priority**: Must-Have
- **Description**: The system shall allow the same person to be registered as a student in multiple gyms with separate, independent profiles.
- **Acceptance Criteria**:
  - Same person (by name/phone) can exist as student in multiple gyms
  - Each gym maintains separate student profile
  - Monthly price can differ between gyms
  - Class enrollments are gym-specific
  - Payment history is gym-specific
  - No data sharing between student profiles across gyms
- **Related Requirements**: FR-006, FR-012

---

## 6. Class Management

### FR-016: Create Class
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to create class schedules with specific attributes including name, schedule, room, teacher assignment, and maximum capacity.
- **Acceptance Criteria**:
  - Administrator can create class with: name/type, days of week, time, room/area, assigned teacher, maximum capacity
  - All fields are required
  - Administrator can select multiple days of week (e.g., Mon/Wed/Fri)
  - Time is specified in 24-hour format (HH:MM)
  - Maximum capacity must be positive integer
  - System validates teacher is available on selected days/times per work schedule
  - System prevents scheduling conflicts (same teacher, same time)
  - Class is immediately available for student enrollment
  - System displays confirmation message upon creation
- **Related Requirements**: FR-010, FR-018

### FR-017: View and Update Class
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to view and update class details.
- **Acceptance Criteria**:
  - Administrator can view all class details including enrolled students
  - Administrator can update class name, schedule, room, teacher, capacity
  - System validates teacher availability when changing teacher or schedule
  - System prevents capacity reduction below current enrollment count
  - System displays warning if schedule changes affect enrolled students
  - Changes reflect immediately throughout the system
  - System displays confirmation message after update
- **Related Requirements**: FR-016, FR-018

### FR-018: Delete Class
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to delete classes.
- **Acceptance Criteria**:
  - Administrator can delete any class
  - System displays warning if class has enrolled students
  - System requires confirmation before deletion
  - Upon deletion, all student enrollments are removed
  - Class is permanently removed from system
  - Class no longer appears in schedules or lists
- **Related Requirements**: FR-016, FR-017

### FR-019: View Class Roster
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to view the list of students enrolled in each class.
- **Acceptance Criteria**:
  - Administrator can view complete list of enrolled students for any class
  - List displays student name, phone, and enrollment date
  - List shows current enrollment count vs. maximum capacity
  - List is sortable by student name or enrollment date
  - Administrator can access student profile from roster
- **Related Requirements**: FR-016, FR-020

### FR-020: Enroll Student in Class
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to enroll students in classes, with support for multiple simultaneous class enrollments per student.
- **Acceptance Criteria**:
  - Administrator can enroll any student in any class
  - System checks class capacity before enrollment
  - System prevents enrollment if class is at maximum capacity
  - System allows student to be enrolled in multiple classes simultaneously
  - System prevents duplicate enrollment (same student, same class)
  - System records enrollment date
  - Enrollment reflects immediately in class roster
  - System displays confirmation message after enrollment
- **Related Requirements**: FR-012, FR-016, FR-019

### FR-021: Unenroll Student from Class
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to remove students from class enrollments.
- **Acceptance Criteria**:
  - Administrator can unenroll student from any class
  - System displays confirmation dialog before unenrolling
  - Unenrollment is immediate
  - Student is removed from class roster
  - System displays confirmation message after unenrollment
  - Unenrollment does not affect payment obligations (handled separately)
- **Related Requirements**: FR-020

### FR-022: Class Capacity Enforcement
- **Priority**: Must-Have
- **Description**: The system shall enforce configurable maximum capacity limits for each class, preventing over-enrollment.
- **Acceptance Criteria**:
  - System tracks current enrollment count for each class
  - System displays available spots (capacity - current enrollment)
  - System prevents enrollment when class is at capacity
  - System displays clear error message when capacity is reached
  - Administrator can modify capacity (subject to current enrollment constraints)
  - Capacity limits are enforced in real-time
- **Related Requirements**: FR-016, FR-020

---

## 7. Payment Management

### FR-023: Record Student Payment
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to record offline payments made by students.
- **Acceptance Criteria**:
  - Administrator can record payment for any student
  - Administrator specifies: payment date, amount (BRL), payment method
  - Payment methods include: Cash, Bank Transfer, Debit Card, Credit Card, Other
  - Payment amount must be positive number
  - Payment date cannot be in the future
  - System records timestamp of data entry
  - Payment appears immediately in payment history
  - System displays confirmation message after recording
- **Related Requirements**: FR-012, FR-024

### FR-024: View Payment History
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to view complete payment history for each student.
- **Acceptance Criteria**:
  - Administrator can view all payments for any student
  - Payment history displays: date, amount, payment method, entry timestamp
  - Payments are sorted by date (most recent first)
  - Administrator can filter by date range
  - Administrator can filter by payment method
  - History shows total amount paid and period covered
  - Administrator can access payment history from student profile
- **Related Requirements**: FR-023

### FR-025: Update Student Monthly Price
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to change a student's monthly price, with changes applying to future payments only.
- **Acceptance Criteria**:
  - Administrator can update monthly price for any student
  - New price must be positive number in BRL
  - Price change does not affect historical payment records
  - System records effective date of price change
  - System displays confirmation message after update
  - New price is used for future payment status calculations
- **Related Requirements**: FR-012, FR-013, FR-026

### FR-026: Configure Student Billing Cycle
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to configure different billing cycle start dates for each student.
- **Acceptance Criteria**:
  - Administrator can set billing cycle day (1-31) for each student
  - Billing cycle day determines when payment is due each month
  - Different students can have different billing cycle days
  - System handles months with fewer than 31 days appropriately
  - Administrator can change billing cycle day with immediate effect
  - System displays confirmation message after change
- **Related Requirements**: FR-012, FR-013, FR-027

### FR-027: Payment Status Calculation
- **Priority**: Must-Have
- **Description**: The system shall automatically calculate payment status for each student based on due dates and payment history, with zero grace period.
- **Acceptance Criteria**:
  - System calculates due date based on billing cycle day
  - Payment is considered overdue on the day after due date
  - System compares last payment date against current due date
  - Overdue status is calculated in real-time
  - System displays visual indicator (red mark) for overdue payments
  - Administrator can see overdue amount and days overdue
  - Status updates automatically when payment is recorded
- **Related Requirements**: FR-023, FR-026, FR-030

### FR-028: Payment Method Tracking
- **Priority**: Must-Have
- **Description**: The system shall track payment method for reference purposes.
- **Acceptance Criteria**:
  - System records payment method for each payment
  - Payment methods available: Cash, Bank Transfer, Debit Card, Credit Card, Other
  - Administrator can view payment method in payment history
  - Administrator can filter/report by payment method
  - Payment method is for reference only (no processing)
- **Related Requirements**: FR-023, FR-024

---

## 8. Dashboard and Reporting

### FR-029: Admin Dashboard
- **Priority**: Must-Have
- **Description**: Gym Administrators shall have access to a dashboard providing an overview of gym operations.
- **Acceptance Criteria**:
  - Dashboard displays total number of active students
  - Dashboard displays current class schedule
  - Dashboard displays count of overdue payments
  - Dashboard is accessible immediately after login
  - Dashboard data updates in real-time
  - Dashboard is the default landing page for administrators
  - Root User can access dashboard for any gym
- **Related Requirements**: FR-001, FR-002, FR-030

### FR-030: View Class Schedule
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to view complete class schedules.
- **Acceptance Criteria**:
  - Administrator can view weekly class schedule
  - Schedule displays: class name, days, time, room, teacher, enrollment count
  - Schedule can be filtered by day of week
  - Schedule can be filtered by teacher
  - Schedule indicates classes near capacity (90%+ full)
  - Schedule is printable
  - Administrator can navigate to class details from schedule
- **Related Requirements**: FR-016, FR-029

### FR-031: Student List View
- **Priority**: Must-Have
- **Description**: Gym Administrators shall be able to view and search complete student lists.
- **Acceptance Criteria**:
  - Administrator can view list of all students
  - List displays: name, phone, monthly price, payment status
  - List can be searched by name or phone number
  - List can be filtered by payment status (current/overdue)
  - List can be sorted by name, monthly price, or payment status
  - Payment status is visually indicated (green=current, red=overdue)
  - Administrator can access student profile from list
- **Related Requirements**: FR-012, FR-027, FR-029

---

## 9. Notification System (Nice-to-Have)

### FR-032: Configure Notification Settings
- **Priority**: Nice-to-Have
- **Description**: Gym Administrators shall be able to configure notification settings for their gym.
- **Acceptance Criteria**:
  - Administrator can enable/disable notification system
  - Administrator can select notification channel: SMS, WhatsApp, or Both
  - Administrator can select notification mode: Automated or Manual
  - Automated mode: System sends notifications automatically when payment overdue
  - Manual mode: Administrator manually triggers notifications
  - Settings can be updated at any time
  - Changes take effect immediately
  - System displays confirmation message after settings update
- **Related Requirements**: FR-033, FR-034

### FR-033: Automated Payment Notifications
- **Priority**: Nice-to-Have
- **Description**: When automated notification mode is enabled, the system shall automatically send payment reminders to students with overdue payments.
- **Acceptance Criteria**:
  - System checks for overdue payments daily
  - System sends notification to each overdue student via selected channel(s)
  - Notification includes: gym name, overdue amount, days overdue
  - System logs all notification attempts
  - System retries failed notifications (max 3 attempts)
  - System does not send duplicate notifications on same day
  - Administrator can view notification history
- **Related Requirements**: FR-027, FR-032

### FR-034: Manual Payment Notifications
- **Priority**: Nice-to-Have
- **Description**: When manual notification mode is enabled, Gym Administrators shall be able to manually send payment reminders to selected students.
- **Acceptance Criteria**:
  - Administrator can select one or more overdue students
  - Administrator can send notification via SMS, WhatsApp, or both
  - System displays notification preview before sending
  - Administrator can customize notification message
  - System confirms sending before transmitting
  - System displays confirmation after notifications sent
  - System logs all manual notifications
  - Administrator can view notification history
- **Related Requirements**: FR-027, FR-032

### FR-035: Notification History
- **Priority**: Nice-to-Have
- **Description**: Gym Administrators shall be able to view history of all notifications sent.
- **Acceptance Criteria**:
  - Administrator can view list of all notifications sent
  - History displays: student name, date/time, channel, status (sent/failed)
  - History can be filtered by date range
  - History can be filtered by student
  - History can be filtered by status
  - History shows notification content
  - Failed notifications display error reason
- **Related Requirements**: FR-032, FR-033, FR-034

---

## 10. System Administration

### FR-036: Session Management
- **Priority**: Must-Have
- **Description**: The system shall manage user sessions securely with automatic timeout.
- **Acceptance Criteria**:
  - Session is created upon successful login
  - Session persists across page navigation
  - Session expires after 4 hours of inactivity
  - User is redirected to login page when session expires
  - User can manually log out
  - Manual logout immediately invalidates session
  - System displays timeout warning 5 minutes before expiration
- **Related Requirements**: FR-001

### FR-037: Data Validation
- **Priority**: Must-Have
- **Description**: The system shall validate all user input before processing or storage.
- **Acceptance Criteria**:
  - All required fields are validated for presence
  - Phone numbers are validated for Brazilian format
  - Email addresses (if added) are validated for format
  - Dates are validated for logical consistency
  - Numeric values are validated for positive numbers where required
  - Text fields are validated for length limits
  - System displays clear error messages for validation failures
  - Validation occurs on client-side and server-side
- **Related Requirements**: All data entry requirements

### FR-038: Error Handling
- **Priority**: Must-Have
- **Description**: The system shall handle errors gracefully and provide clear feedback to users.
- **Acceptance Criteria**:
  - System displays user-friendly error messages (not technical errors)
  - Error messages explain what went wrong and how to fix it
  - System logs all errors for administrator review
  - System does not expose sensitive information in error messages
  - Critical errors are escalated for immediate attention
  - User can recover from errors without losing work
  - System provides contact information for support on critical errors
- **Related Requirements**: All functional requirements

### FR-039: Data Backup
- **Priority**: Must-Have
- **Description**: The system shall automatically backup all data to prevent data loss.
- **Acceptance Criteria**:
  - System performs daily automated backups
  - Backups include all gym, teacher, student, class, and payment data
  - Backups are encrypted
  - Backups are stored in separate geographic location
  - System retains backups for minimum 30 days
  - System can restore from backup within 4 hours
  - Backup success/failure is monitored and alerted
- **Related Requirements**: All data management requirements

---

## 11. Requirement Traceability Matrix

| Requirement ID | Category | Priority | Depends On | Supports |
|---------------|----------|----------|------------|----------|
| FR-001 | Authentication | Must-Have | - | All others |
| FR-002 | Authorization | Must-Have | FR-001 | All others |
| FR-003 | Admin Management | Must-Have | FR-001, FR-002 | - |
| FR-004 | Gym Management | Must-Have | FR-001, FR-002 | FR-005, FR-006 |
| FR-005 | Gym Management | Must-Have | FR-004 | - |
| FR-006 | Multi-Tenancy | Must-Have | FR-001, FR-002 | All gym-specific |
| FR-007 | Teacher Management | Must-Have | FR-002, FR-006 | FR-008-011 |
| FR-008 | Teacher Management | Must-Have | FR-007 | - |
| FR-009 | Teacher Management | Must-Have | FR-007, FR-008 | - |
| FR-010 | Teacher Management | Must-Have | FR-007, FR-008 | FR-016 |
| FR-011 | Teacher Management | Must-Have | FR-006, FR-007 | - |
| FR-012 | Student Management | Must-Have | FR-002, FR-006 | FR-013-015 |
| FR-013 | Student Management | Must-Have | FR-012 | - |
| FR-014 | Student Management | Must-Have | FR-012, FR-013 | - |
| FR-015 | Student Management | Must-Have | FR-006, FR-012 | - |
| FR-016 | Class Management | Must-Have | FR-010 | FR-017-022 |
| FR-017 | Class Management | Must-Have | FR-016 | - |
| FR-018 | Class Management | Must-Have | FR-016, FR-017 | - |
| FR-019 | Class Management | Must-Have | FR-016 | - |
| FR-020 | Class Management | Must-Have | FR-012, FR-016 | FR-021, FR-022 |
| FR-021 | Class Management | Must-Have | FR-020 | - |
| FR-022 | Class Management | Must-Have | FR-016, FR-020 | - |
| FR-023 | Payment Management | Must-Have | FR-012 | FR-024-028 |
| FR-024 | Payment Management | Must-Have | FR-023 | - |
| FR-025 | Payment Management | Must-Have | FR-012, FR-013 | FR-027 |
| FR-026 | Payment Management | Must-Have | FR-012, FR-013 | FR-027 |
| FR-027 | Payment Management | Must-Have | FR-023, FR-026 | FR-030-035 |
| FR-028 | Payment Management | Must-Have | FR-023, FR-024 | - |
| FR-029 | Dashboard | Must-Have | FR-001, FR-002 | - |
| FR-030 | Dashboard | Must-Have | FR-016, FR-029 | - |
| FR-031 | Dashboard | Must-Have | FR-012, FR-027 | - |
| FR-032 | Notifications | Nice-to-Have | - | FR-033-035 |
| FR-033 | Notifications | Nice-to-Have | FR-027, FR-032 | - |
| FR-034 | Notifications | Nice-to-Have | FR-027, FR-032 | - |
| FR-035 | Notifications | Nice-to-Have | FR-032-034 | - |
| FR-036 | System Admin | Must-Have | FR-001 | - |
| FR-037 | System Admin | Must-Have | All data entry | - |
| FR-038 | System Admin | Must-Have | All requirements | - |
| FR-039 | System Admin | Must-Have | All data management | - |

---

## 12. Requirements Summary

### Must-Have Requirements: 35
- Authentication and Authorization: 3
- Gym Management: 3
- Teacher Management: 5
- Student Management: 4
- Class Management: 7
- Payment Management: 6
- Dashboard and Reporting: 3
- System Administration: 4

### Nice-to-Have Requirements: 4
- Notification System: 4

### Total Requirements: 39

---

**Document Revision History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-10-16 | Requirements Team | Initial version |
