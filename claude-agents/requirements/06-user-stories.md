# User Stories

## Document Information
- **Version**: 1.0
- **Date**: October 16, 2025
- **Status**: Final

## 1. Introduction

This document presents user stories for the Muay Thai Gym Management System. User stories describe system functionality from the perspective of end users, using the format: "As a [role], I want [goal] so that [benefit]."

### Story Format

Each user story includes:
- **ID**: Unique identifier (US-XXX)
- **Role**: User role (Root User, Gym Administrator)
- **Story**: User story statement
- **Priority**: Must-Have / Nice-to-Have
- **Story Points**: Complexity estimate (1, 2, 3, 5, 8, 13)
- **Acceptance Criteria**: Specific, testable conditions
- **Related Requirements**: Links to functional requirements

### Story Point Scale
- **1**: Trivial (few hours)
- **2**: Simple (1 day)
- **3**: Moderate (2-3 days)
- **5**: Complex (1 week)
- **8**: Very complex (2 weeks)
- **13**: Epic (needs breakdown)

---

## 2. Authentication and Authorization Stories

### US-001: Login to System
- **Role**: Gym Administrator / Root User
- **Story**: As a gym administrator, I want to log in with my username and password so that I can access the gym management system securely.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can enter my username and password on the login page
  - System validates my credentials
  - If credentials are valid, I am redirected to the dashboard
  - If credentials are invalid, I see a clear error message
  - My session persists as I navigate the system
  - I can log out at any time
- **Related Requirements**: FR-001, NFR-006

### US-002: Automatic Session Timeout
- **Role**: Gym Administrator / Root User
- **Story**: As a system user, I want my session to automatically expire after inactivity so that unauthorized users cannot access my account if I forget to log out.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - Session expires after 4 hours of inactivity
  - I receive a warning 5 minutes before expiration
  - I can extend my session before it expires
  - After expiration, I am redirected to login page
  - A clear message explains why I was logged out
- **Related Requirements**: FR-036, NFR-011

### US-003: View Only My Gym Data
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to see only my gym's data so that I maintain privacy and focus on my gym's operations.
- **Priority**: Must-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - I can only see students, teachers, and classes for my gym
  - I cannot access or view other gyms' data
  - All lists, reports, and dashboards show only my gym's data
  - System prevents me from accessing other gyms' URLs
  - Clear error message if I attempt unauthorized access
- **Related Requirements**: FR-002, FR-006, NFR-009

### US-004: Manage All Gyms
- **Role**: Root User
- **Story**: As a root user, I want to access and manage all gyms in the system so that I can provide support and oversight.
- **Priority**: Must-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - I can view a list of all gyms
  - I can select any gym to view its data
  - I can perform any operation in any gym
  - I can switch between gyms easily
  - System clearly indicates which gym I am currently managing
- **Related Requirements**: FR-002, FR-004

---

## 3. Gym Management Stories

### US-005: Create New Gym
- **Role**: Root User
- **Story**: As a root user, I want to create new gym profiles so that new customers can start using the system.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can access a "Create Gym" form
  - I can enter gym name, address, and owner name
  - I can optionally upload a gym icon/logo
  - System validates all required fields
  - System creates the gym and confirms success
  - New gym appears in the gym list immediately
- **Related Requirements**: FR-004

### US-006: Update Gym Information
- **Role**: Gym Administrator / Root User
- **Story**: As a gym administrator, I want to update my gym's profile information so that details remain current and accurate.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can access gym profile settings
  - I can update gym name, address, owner, and icon
  - System validates my changes
  - System saves changes and confirms success
  - Changes reflect immediately throughout the system
- **Related Requirements**: FR-005

### US-007: Create Administrator Account
- **Role**: Root User
- **Story**: As a root user, I want to create administrator accounts for each gym so that gym owners can manage their own operations.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can create a new administrator account
  - I specify username, password, and assigned gym
  - System validates username uniqueness
  - System creates account and confirms success
  - Administrator can immediately log in with provided credentials
- **Related Requirements**: FR-003

---

## 4. Teacher Management Stories

### US-008: Add New Teacher
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to add new teachers to my gym so that I can assign them to classes.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can access "Add Teacher" form
  - I can enter teacher name, phone, address, and birth date
  - System validates all fields (phone format, age requirement)
  - System creates teacher profile and confirms success
  - New teacher appears in teacher list immediately
- **Related Requirements**: FR-007, NFR-037

### US-009: Define Teacher Schedule
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to define each teacher's available work schedule so that I only assign them to classes during their available hours.
- **Priority**: Must-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - I can select a teacher to edit their schedule
  - I can set availability for each day (Monday-Friday)
  - I can define start and end times for each available day
  - System validates that end time is after start time
  - System saves schedule and confirms success
  - Schedule is used to validate future class assignments
- **Related Requirements**: FR-010

### US-010: View Teacher Details
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to view complete teacher details including their assigned classes so that I understand their workload and schedule.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can select a teacher from the list
  - I see teacher profile information
  - I see their work schedule
  - I see list of classes they are assigned to
  - I can navigate to class details from this view
- **Related Requirements**: FR-008

### US-011: Update Teacher Information
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to update teacher information so that records remain accurate.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can edit teacher profile fields
  - System validates my changes
  - System saves updates and confirms success
  - Changes reflect immediately throughout the system
- **Related Requirements**: FR-008

### US-012: Remove Teacher
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to remove teachers who are no longer working at my gym so that my teacher list remains current.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can select a teacher to delete
  - System checks if teacher is assigned to active classes
  - If assigned to classes, system prevents deletion with clear message
  - If not assigned, system asks for confirmation
  - Upon confirmation, teacher is removed
  - Teacher no longer appears in teacher list
- **Related Requirements**: FR-009

---

## 5. Student Management Stories

### US-013: Enroll New Student
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to enroll new students in my gym so that they can participate in classes and I can track their payments.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can access "Enroll Student" form
  - I can enter student name, phone, address, birth date
  - I can set their monthly price
  - I can set their billing cycle day (1-31)
  - System validates all fields
  - System creates student profile and confirms success
  - New student appears in student list immediately
- **Related Requirements**: FR-012, NFR-019

### US-014: View Student Profile
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to view complete student profiles including their class enrollments and payment history so that I have full visibility into their membership.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can select a student from the list
  - I see student profile information
  - I see list of classes they are enrolled in
  - I see their payment history
  - I see their current payment status
  - I can navigate to related records (classes, payments)
- **Related Requirements**: FR-013, FR-024

### US-015: Update Student Information
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to update student information so that records remain accurate and reflect current pricing.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can edit student profile fields
  - I can update monthly price (applies to future payments)
  - I can update billing cycle day
  - System validates my changes
  - System saves updates and confirms success
  - Changes reflect immediately
- **Related Requirements**: FR-013, FR-025, FR-026

### US-016: Remove Student
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to remove students who are no longer members so that my active student list is accurate.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can select a student to delete
  - System warns if student has payment history
  - System asks for confirmation
  - Upon confirmation, student is removed from active list
  - Student's payment history is retained
  - Student is unenrolled from all classes automatically
- **Related Requirements**: FR-014

### US-017: Search for Students
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to search for students by name or phone number so that I can quickly find specific students.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can enter search terms in a search box
  - System searches student names and phone numbers
  - Results appear in real-time as I type
  - I can select a student from search results
  - Search is case-insensitive
- **Related Requirements**: FR-031

---

## 6. Class Management Stories

### US-018: Create New Class
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to create new classes with specific schedules and assigned teachers so that students can enroll.
- **Priority**: Must-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - I can access "Create Class" form
  - I can enter class name, room, capacity, time, and duration
  - I can select days of week (e.g., Mon/Wed/Fri)
  - I can assign a teacher from my gym's teachers
  - System validates teacher is available on selected days/times
  - System prevents scheduling conflicts
  - System creates class and confirms success
  - New class appears in class list and schedule
- **Related Requirements**: FR-016, NFR-019

### US-019: View Class Details
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to view class details including enrolled students so that I know class status and attendance capacity.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can select a class from the list
  - I see class schedule, teacher, room, and capacity
  - I see current enrollment count vs. maximum capacity
  - I see list of enrolled students
  - I can navigate to student profiles from this view
- **Related Requirements**: FR-019

### US-020: Update Class Information
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to update class details so that I can adjust schedules, teachers, or capacity as needed.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can edit class fields
  - System validates teacher availability when changing teacher/schedule
  - System prevents capacity reduction below current enrollment
  - System warns if changes affect enrolled students
  - System saves updates and confirms success
- **Related Requirements**: FR-017

### US-021: Delete Class
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to delete classes that are no longer offered so that my schedule remains current.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can select a class to delete
  - System warns if students are enrolled
  - System asks for confirmation
  - Upon confirmation, class is deleted
  - All student enrollments are automatically removed
  - Class no longer appears in schedule
- **Related Requirements**: FR-018

### US-022: Enroll Student in Class
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to enroll students in classes so that they can participate in training sessions.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can select a class and add students
  - System shows available capacity
  - System prevents enrollment if class is full
  - System prevents duplicate enrollment
  - System allows student to be in multiple classes
  - Enrollment is confirmed and student appears in class roster
- **Related Requirements**: FR-020, FR-022

### US-023: Remove Student from Class
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to unenroll students from classes so that I can manage roster changes.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can select an enrolled student and remove them
  - System asks for confirmation
  - Upon confirmation, student is unenrolled
  - Student no longer appears in class roster
  - Class capacity updates immediately
- **Related Requirements**: FR-021

### US-024: View Weekly Schedule
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to view the complete weekly class schedule so that I can see all classes at a glance.
- **Priority**: Must-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - I can view a weekly calendar/grid of all classes
  - Schedule shows class name, time, teacher, room, enrollment
  - Schedule indicates classes near capacity
  - I can filter by day of week or teacher
  - Schedule is printable
  - I can click on a class to see details
- **Related Requirements**: FR-030

---

## 7. Payment Management Stories

### US-025: Record Student Payment
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to record payments received from students so that I maintain accurate financial records.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can select a student and record a payment
  - I can enter payment date, amount, and method
  - System validates amount is positive
  - System validates date is not in future
  - System records payment and confirms success
  - Payment appears immediately in student's history
  - Student's payment status updates automatically
- **Related Requirements**: FR-023, NFR-019

### US-026: View Payment History
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to view complete payment history for each student so that I can track their payment patterns.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can view all payments for a student
  - History shows date, amount, method, and timestamp
  - Payments are sorted by date (most recent first)
  - I can filter by date range
  - I can filter by payment method
  - History shows total amount paid and period covered
- **Related Requirements**: FR-024

### US-027: See Payment Status at a Glance
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to immediately see which students have overdue payments so that I can follow up for payment collection.
- **Priority**: Must-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - Student list shows payment status for each student
  - Overdue payments are marked with red indicator
  - Current payments are marked with green indicator
  - I can see days overdue for overdue payments
  - I can filter student list by payment status
  - Status updates in real-time when payments recorded
- **Related Requirements**: FR-027, FR-031

### US-028: Adjust Student Monthly Price
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to change a student's monthly price so that I can accommodate special pricing or price increases.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can update student's monthly price
  - New price applies to future payments only
  - Historical payment records remain unchanged
  - System records effective date of price change
  - Change is confirmed and takes effect immediately
- **Related Requirements**: FR-025

### US-029: Set Student Billing Cycle
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to set different billing cycle days for each student so that I can accommodate when they prefer to pay each month.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I can set billing cycle day (1-31) for each student
  - System handles months with fewer than 31 days appropriately
  - Billing cycle determines when payment is due
  - I can change billing cycle day with immediate effect
  - System confirms change
- **Related Requirements**: FR-026

### US-030: Track Payment Methods
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to record how students paid (cash, transfer, card) so that I can track payment method preferences.
- **Priority**: Must-Have
- **Story Points**: 1
- **Acceptance Criteria**:
  - When recording payment, I select payment method
  - Options include: Cash, Bank Transfer, Debit Card, Credit Card, Other
  - Payment method is saved with payment record
  - I can view payment method in payment history
  - I can filter payments by method
- **Related Requirements**: FR-028

---

## 8. Dashboard and Reporting Stories

### US-031: View Dashboard Overview
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to see a dashboard with key metrics when I log in so that I immediately understand my gym's status.
- **Priority**: Must-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - Dashboard is my landing page after login
  - I see total active student count
  - I see count of overdue payments
  - I see today's class schedule
  - I see quick links to common tasks
  - Dashboard loads within 2 seconds
- **Related Requirements**: FR-029, NFR-001

### US-032: View Student List
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to view and manage my complete student list so that I can access student information quickly.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can view list of all active students
  - List shows name, phone, monthly price, payment status
  - I can sort by name, price, or payment status
  - I can search by name or phone
  - I can filter by payment status
  - I can click on student to view full profile
- **Related Requirements**: FR-031

### US-033: Print Class Schedule
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to print the weekly class schedule so that I can post it at the gym or share with teachers.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - Schedule has a print button
  - Print view is formatted for paper (portrait or landscape)
  - Print shows all necessary details clearly
  - Print excludes unnecessary navigation elements
  - Print quality is professional
- **Related Requirements**: FR-030

---

## 9. Notification System Stories (Nice-to-Have)

### US-034: Configure Notification Settings
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to configure notification settings so that I can control how and when students are notified about overdue payments.
- **Priority**: Nice-to-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can access notification settings
  - I can enable/disable notifications
  - I can select notification channels (SMS, WhatsApp, or both)
  - I can choose automated or manual mode
  - System saves settings and confirms
  - Changes take effect immediately
- **Related Requirements**: FR-032

### US-035: Send Payment Reminder Manually
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to manually send payment reminders to selected students so that I have control over communication timing.
- **Priority**: Nice-to-Have
- **Story Points**: 5
- **Acceptance Criteria**:
  - I can select one or more students with overdue payments
  - I can choose notification channel (SMS, WhatsApp, or both)
  - System shows preview of notification message
  - I can customize message if desired
  - System asks for confirmation before sending
  - System confirms notifications were sent
  - Notifications are logged
- **Related Requirements**: FR-034

### US-036: Automatic Payment Reminders
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want the system to automatically send payment reminders to overdue students so that I don't have to remember to do it manually.
- **Priority**: Nice-to-Have
- **Story Points**: 8
- **Acceptance Criteria**:
  - When automated mode is enabled, system checks for overdue payments daily
  - System sends reminder to each overdue student
  - System uses configured channels (SMS, WhatsApp, or both)
  - System does not send duplicate notifications same day
  - System retries failed notifications (max 3 attempts)
  - I can view notification history to see what was sent
- **Related Requirements**: FR-033

### US-037: View Notification History
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to view history of all notifications sent so that I can verify students were notified and troubleshoot delivery issues.
- **Priority**: Nice-to-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - I can access notification history
  - History shows student, date/time, channel, status
  - I can filter by date range, student, or status
  - Failed notifications show error reason
  - I can see notification content that was sent
  - History is retained for at least 90 days
- **Related Requirements**: FR-035

---

## 10. System Administration Stories

### US-038: Recover from Session Timeout
- **Role**: Gym Administrator / Root User
- **Story**: As a system user, I want to be warned before my session expires so that I don't lose work due to automatic logout.
- **Priority**: Must-Have
- **Story Points**: 2
- **Acceptance Criteria**:
  - I receive warning 5 minutes before session expires
  - Warning shows remaining time
  - I can extend session by clicking button
  - If I don't respond, session expires and I'm logged out
  - Upon re-login, I'm returned to the page I was on (if appropriate)
- **Related Requirements**: FR-036, NFR-011

### US-039: Receive Clear Error Messages
- **Role**: Gym Administrator / Root User
- **Story**: As a system user, I want to receive clear, helpful error messages when something goes wrong so that I understand what happened and how to fix it.
- **Priority**: Must-Have
- **Story Points**: 3
- **Acceptance Criteria**:
  - Error messages are in plain Portuguese (no technical jargon)
  - Error messages explain what went wrong
  - Error messages suggest how to fix the problem
  - Errors don't expose sensitive information
  - Critical errors provide support contact information
  - I can recover from errors without losing my work
- **Related Requirements**: FR-038, NFR-024

### US-040: Use System on Mobile Device
- **Role**: Gym Administrator
- **Story**: As a gym administrator, I want to use the system on my mobile phone so that I can manage my gym on the go.
- **Priority**: Must-Have
- **Story Points**: 8
- **Acceptance Criteria**:
  - All features are accessible on mobile browser
  - Interface adapts to small screen sizes
  - Text is readable without zooming
  - Buttons and links are easy to tap
  - Forms are easy to fill out on mobile
  - Performance is acceptable on mobile connection
- **Related Requirements**: NFR-020, NFR-021

---

## 11. User Story Summary

### By Priority

**Must-Have Stories**: 36
- Authentication: 4
- Gym Management: 3
- Teacher Management: 5
- Student Management: 5
- Class Management: 7
- Payment Management: 6
- Dashboard: 3
- System: 3

**Nice-to-Have Stories**: 4
- Notification System: 4

**Total Stories**: 40

### By Story Points

| Points | Count | Stories |
|--------|-------|---------|
| 1 | 1 | US-030 |
| 2 | 13 | US-002, US-006, US-010, US-011, US-017, US-019, US-021, US-023, US-026, US-028, US-029, US-033, US-038 |
| 3 | 15 | US-001, US-005, US-007, US-008, US-012, US-013, US-014, US-016, US-020, US-022, US-025, US-032, US-034, US-037, US-039 |
| 5 | 9 | US-003, US-004, US-009, US-018, US-024, US-027, US-031, US-035 |
| 8 | 2 | US-036, US-040 |

**Total Story Points (Must-Have)**: 127
**Total Story Points (Nice-to-Have)**: 19
**Total Story Points**: 146

### Estimated Development Effort
- Assuming 10 story points per week per developer
- Must-Have: ~13 weeks (3.25 months)
- Nice-to-Have: ~2 weeks
- Total: ~15 weeks (3.75 months)

---

## 12. Story Mapping

### Epic 1: User Management (23 points)
- US-001: Login to System (3)
- US-002: Automatic Session Timeout (2)
- US-003: View Only My Gym Data (5)
- US-004: Manage All Gyms (5)
- US-005: Create New Gym (3)
- US-006: Update Gym Information (2)
- US-007: Create Administrator Account (3)

### Epic 2: Teacher Management (15 points)
- US-008: Add New Teacher (3)
- US-009: Define Teacher Schedule (5)
- US-010: View Teacher Details (2)
- US-011: Update Teacher Information (2)
- US-012: Remove Teacher (3)

### Epic 3: Student Management (15 points)
- US-013: Enroll New Student (3)
- US-014: View Student Profile (3)
- US-015: Update Student Information (2)
- US-016: Remove Student (3)
- US-017: Search for Students (2)

### Epic 4: Class Management (20 points)
- US-018: Create New Class (5)
- US-019: View Class Details (2)
- US-020: Update Class Information (3)
- US-021: Delete Class (2)
- US-022: Enroll Student in Class (3)
- US-023: Remove Student from Class (2)
- US-024: View Weekly Schedule (5)

### Epic 5: Payment Management (17 points)
- US-025: Record Student Payment (3)
- US-026: View Payment History (2)
- US-027: See Payment Status at a Glance (5)
- US-028: Adjust Student Monthly Price (2)
- US-029: Set Student Billing Cycle (2)
- US-030: Track Payment Methods (1)

### Epic 6: Dashboard and Reporting (10 points)
- US-031: View Dashboard Overview (5)
- US-032: View Student List (3)
- US-033: Print Class Schedule (2)

### Epic 7: Notifications (19 points - Nice-to-Have)
- US-034: Configure Notification Settings (3)
- US-035: Send Payment Reminder Manually (5)
- US-036: Automatic Payment Reminders (8)
- US-037: View Notification History (3)

### Epic 8: System Quality (13 points)
- US-038: Recover from Session Timeout (2)
- US-039: Receive Clear Error Messages (3)
- US-040: Use System on Mobile Device (8)

---

## 13. Release Planning

### MVP Release 1 (Sprint 1-4, 8 weeks, 80 points)
**Goal**: Core functionality for single gym operations

**Sprint 1 (20 points)**:
- US-001: Login to System (3)
- US-003: View Only My Gym Data (5)
- US-005: Create New Gym (3)
- US-006: Update Gym Information (2)
- US-007: Create Administrator Account (3)
- US-038: Recover from Session Timeout (2)
- US-039: Receive Clear Error Messages (3)

**Sprint 2 (20 points)**:
- US-008: Add New Teacher (3)
- US-009: Define Teacher Schedule (5)
- US-010: View Teacher Details (2)
- US-011: Update Teacher Information (2)
- US-013: Enroll New Student (3)
- US-014: View Student Profile (3)
- US-015: Update Student Information (2)

**Sprint 3 (20 points)**:
- US-018: Create New Class (5)
- US-019: View Class Details (2)
- US-020: Update Class Information (3)
- US-022: Enroll Student in Class (3)
- US-023: Remove Student from Class (2)
- US-024: View Weekly Schedule (5)

**Sprint 4 (20 points)**:
- US-025: Record Student Payment (3)
- US-026: View Payment History (2)
- US-027: See Payment Status at a Glance (5)
- US-028: Adjust Student Monthly Price (2)
- US-029: Set Student Billing Cycle (2)
- US-030: Track Payment Methods (1)
- US-031: View Dashboard Overview (5)

### MVP Release 2 (Sprint 5-6, 4 weeks, 47 points)
**Goal**: Complete MVP with all must-have features

**Sprint 5 (25 points)**:
- US-002: Automatic Session Timeout (2)
- US-004: Manage All Gyms (5)
- US-012: Remove Teacher (3)
- US-016: Remove Student (3)
- US-017: Search for Students (2)
- US-021: Delete Class (2)
- US-040: Use System on Mobile Device (8)

**Sprint 6 (22 points)**:
- US-032: View Student List (3)
- US-033: Print Class Schedule (2)
- Bug fixes and polish
- User acceptance testing
- Documentation
- Deployment preparation

### Post-MVP (Sprint 7-8, 4 weeks, 19 points)
**Goal**: Add notification features (nice-to-have)

**Sprint 7 (11 points)**:
- US-034: Configure Notification Settings (3)
- US-036: Automatic Payment Reminders (8)

**Sprint 8 (8 points)**:
- US-035: Send Payment Reminder Manually (5)
- US-037: View Notification History (3)

---

**Document Revision History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-10-16 | Requirements Team | Initial version |
