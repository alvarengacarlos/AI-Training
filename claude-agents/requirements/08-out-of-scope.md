# Out of Scope

## Document Information
- **Version**: 1.0
- **Date**: October 16, 2025
- **Status**: Final

## 1. Introduction

This document explicitly defines what is OUT OF SCOPE for the Muay Thai Gym Management System MVP. Clearly documenting excluded features is critical for:
- Managing stakeholder expectations
- Preventing scope creep
- Focusing development effort
- Planning future releases
- Avoiding misunderstandings

### Document Purpose
- Explicitly list features that will NOT be included in MVP
- Provide rationale for each exclusion
- Categorize by potential future consideration
- Set clear boundaries for the project

### Exclusion Categories
1. **Deferred to Post-MVP**: Features that may be added after MVP launch
2. **Not Planned**: Features explicitly rejected or not aligned with product vision
3. **Out of Scope - General**: Categories of functionality not included

---

## 2. Student and Teacher Functionality

### OUT-001: Student Portal/Self-Service
- **Status**: Deferred to Post-MVP
- **Description**: Students will NOT have login access or self-service portal in MVP.
- **Excluded Capabilities**:
  - Student login and authentication
  - View personal class schedule
  - View payment history
  - Update personal information
  - Request class changes
  - Download receipts
  - View gym announcements
  - Message teachers or administrators
- **Rationale**:
  - Increases complexity significantly (additional user role, permissions, UI)
  - Doubles or triples development effort
  - Most students comfortable with admin-managed approach
  - Not critical for gym operations
  - Budget constraints
- **Current Approach**: Administrators manage all student data and provide information directly
- **Future Consideration**: High priority for post-MVP if customer demand justifies investment

### OUT-002: Teacher Portal/Self-Service
- **Status**: Deferred to Post-MVP
- **Description**: Teachers will NOT have login access or self-service portal in MVP.
- **Excluded Capabilities**:
  - Teacher login and authentication
  - View assigned class schedule
  - View student roster for classes
  - Update availability/schedule
  - Mark attendance
  - Communicate with students
  - Submit reports
  - Request schedule changes
- **Rationale**:
  - Similar to student portal - significant complexity increase
  - Teachers can receive schedules via other means (printed, WhatsApp, etc.)
  - Not critical for basic gym operations
  - Budget and timeline constraints
- **Current Approach**: Administrators manage teacher data and communicate schedules directly
- **Future Consideration**: Medium priority for post-MVP

### OUT-003: Teacher Mobile App
- **Status**: Not Planned
- **Description**: No dedicated mobile application for teachers.
- **Rationale**:
  - Teachers don't have system access in MVP
  - Native app development costly and complex
  - Not aligned with MVP scope
- **Current Approach**: N/A - Teachers don't access system
- **Future Consideration**: Only if teacher portal is added and mobile web insufficient

---

## 3. Attendance Tracking

### OUT-004: Attendance Check-In/Check-Out
- **Status**: Deferred to Post-MVP
- **Description**: System will NOT track student attendance at classes.
- **Excluded Capabilities**:
  - Class check-in functionality
  - QR code or barcode scanning
  - Attendance marking by teacher
  - Attendance history per student
  - Attendance reports
  - Attendance-based billing
  - Automatic notifications for absences
  - Attendance statistics and analytics
- **Rationale**:
  - Significant feature requiring hardware (scanners, tablets) or teacher access
  - Not critical for MVP - payment is per month, not per class
  - Adds complexity to class management
  - Budget and timeline constraints
- **Current Approach**: Attendance not tracked in system (gym may track manually if desired)
- **Future Consideration**: High priority for post-MVP - valuable feature for many gyms
- **Alternative**: Gyms can track attendance manually outside the system

### OUT-005: Attendance-Based Billing
- **Status**: Not Planned
- **Description**: Billing based on number of classes attended (pay-per-class).
- **Rationale**:
  - Requires attendance tracking (OUT-004)
  - More complex billing logic
  - Not aligned with typical Muay Thai gym business model (monthly membership)
  - Scope creep
- **Current Approach**: Monthly flat-rate billing only
- **Future Consideration**: Low priority - uncommon business model for martial arts gyms

---

## 4. Payment Processing and Financial Features

### OUT-006: Online Payment Processing
- **Status**: Deferred to Post-MVP
- **Description**: System will NOT process payments online (no credit card, PIX, boleto integration).
- **Excluded Capabilities**:
  - Credit/debit card payment processing
  - PIX integration
  - Boleto generation
  - Bank slip generation
  - Payment gateway integration (Stripe, PagSeguro, Mercado Pago, etc.)
  - Automatic payment collection
  - Recurring payment automation
  - Payment failure handling
  - Refund processing
- **Rationale**:
  - Significant complexity and cost
  - Requires PCI-DSS compliance for card data
  - Payment gateway fees eat into margins
  - Transaction fees add operational costs
  - Security requirements substantially higher
  - Many gyms prefer cash/manual payments
  - Budget and timeline constraints
- **Current Approach**: Administrators manually record offline payments (cash, transfer, etc.)
- **Future Consideration**: Medium priority for post-MVP if customer demand is strong
- **Note**: This is a business decision as much as technical - manual payment tracking is sufficient for MVP

### OUT-007: Financial Reporting and Analytics
- **Status**: Deferred to Post-MVP
- **Description**: System will NOT include advanced financial reports and analytics.
- **Excluded Capabilities**:
  - Revenue reports (monthly, quarterly, annual)
  - Payment method analysis
  - Revenue forecasting
  - Outstanding receivables report
  - Payment trend analysis
  - Financial dashboards
  - Export to accounting software (QuickBooks, etc.)
  - Tax reports
  - Profit/loss statements
  - Custom financial reports
- **Rationale**:
  - Adds significant complexity
  - Requires reporting engine
  - Not critical for basic operations
  - Gyms can export raw data if needed
  - Budget and timeline constraints
- **Current Approach**: Basic payment tracking only; gyms can view payment history per student
- **Future Consideration**: High priority for post-MVP - valuable for gym owners
- **Workaround**: Payment data can be manually exported for analysis in Excel

### OUT-008: Invoicing and Receipts
- **Status**: Deferred to Post-MVP
- **Description**: System will NOT automatically generate invoices or payment receipts.
- **Excluded Capabilities**:
  - Automatic invoice generation
  - Invoice numbering and tracking
  - PDF invoice download
  - Email invoices to students
  - Payment receipt generation
  - Receipt download/printing
  - Invoice templates
  - Tax calculation on invoices
  - Fiscal note (nota fiscal) generation
- **Rationale**:
  - Requires document generation functionality
  - Brazilian fiscal note requirements are complex
  - Not critical for MVP
  - Many small gyms don't issue formal invoices
  - Timeline constraints
- **Current Approach**: Gyms issue invoices/receipts manually if required
- **Future Consideration**: Medium priority for post-MVP
- **Note**: Fiscal note integration especially complex in Brazil

### OUT-009: Discounts and Promotions
- **Status**: Deferred to Post-MVP
- **Description**: System will NOT support automated discounts, coupons, or promotional pricing.
- **Excluded Capabilities**:
  - Discount codes/coupons
  - Percentage or fixed-amount discounts
  - Family/multi-student discounts
  - Referral discounts
  - Promotional campaigns
  - Time-limited offers
  - Automatic discount application
  - Discount tracking and reporting
- **Rationale**:
  - Adds complexity to payment logic
  - Variable pricing per student already supported (can manually adjust price)
  - Automation not critical for MVP
  - Timeline constraints
- **Current Approach**: Administrators manually adjust monthly price for individual students
- **Future Consideration**: Medium priority for post-MVP

---

## 5. Communication and Notifications

### OUT-010: In-App Messaging
- **Status**: Not Planned
- **Description**: System will NOT include internal messaging between users.
- **Excluded Capabilities**:
  - Admin to student messaging
  - Admin to teacher messaging
  - Teacher to student messaging
  - Group messaging
  - Message history
  - Message notifications
  - Unread message indicators
- **Rationale**:
  - Significant feature requiring real-time infrastructure
  - Students and teachers don't have system access anyway
  - External tools (WhatsApp, SMS, email) already solve this need
  - Not critical for gym management
  - Timeline and budget constraints
- **Current Approach**: Communication via WhatsApp, SMS, phone, or in-person
- **Future Consideration**: Low priority - external tools (WhatsApp) work well

### OUT-011: Email Notifications
- **Status**: Deferred to Post-MVP
- **Description**: System will NOT send email notifications to administrators or users.
- **Excluded Capabilities**:
  - Email notifications for overdue payments
  - Email reminders for administrators
  - System alert emails
  - Email notification preferences
  - Email templates
  - Bulk email functionality
- **Rationale**:
  - Students don't have email addresses in system (phone only)
  - SMS/WhatsApp more effective in Brazilian market
  - Email deliverability challenges
  - Additional service cost
  - Timeline constraints
- **Current Approach**: SMS/WhatsApp notifications only (nice-to-have feature)
- **Future Consideration**: Low priority - SMS/WhatsApp preferred in Brazil

### OUT-012: Push Notifications
- **Status**: Not Planned
- **Description**: No browser or mobile push notifications.
- **Rationale**:
  - Web-only system (no native apps)
  - Browser push notifications have low opt-in rates
  - Students don't access system
  - SMS/WhatsApp more effective
- **Current Approach**: N/A
- **Future Consideration**: Low priority unless native apps developed

### OUT-013: Announcement System
- **Status**: Deferred to Post-MVP
- **Description**: No system for posting announcements or news to students.
- **Excluded Capabilities**:
  - Create and publish announcements
  - Announcement categories (gym closures, events, etc.)
  - Announcement history
  - Student notification of new announcements
  - Announcement expiration
- **Rationale**:
  - Students don't access system
  - Gyms communicate via WhatsApp groups, Instagram, etc.
  - Not critical for operations
  - Timeline constraints
- **Current Approach**: External communication channels (WhatsApp, social media)
- **Future Consideration**: Low priority

---

## 6. Advanced Class Management

### OUT-014: Waitlist Functionality
- **Status**: Not Planned (Explicitly Excluded)
- **Description**: System will NOT support waitlists for full classes.
- **Excluded Capabilities**:
  - Add students to class waitlist
  - Automatic enrollment when spot opens
  - Waitlist position tracking
  - Waitlist notifications
  - Waitlist management
- **Rationale**:
  - Explicitly excluded per requirements
  - Adds complexity to class management
  - Not commonly needed in martial arts gyms
  - Timeline constraints
- **Current Approach**: Administrators manually track interested students (outside system)
- **Future Consideration**: Low priority - not requested by stakeholders

### OUT-015: Private/Semi-Private Classes
- **Status**: Deferred to Post-MVP
- **Description**: No special handling for private lessons or small group training.
- **Excluded Capabilities**:
  - One-on-one class booking
  - Per-session pricing for private lessons
  - Private class scheduling
  - Private class payment tracking separate from membership
- **Rationale**:
  - Can be modeled as regular class with capacity=1
  - Special billing logic adds complexity
  - Not critical for MVP
  - Timeline constraints
- **Current Approach**: Can create class with capacity=1; track private lesson payments as regular payments
- **Future Consideration**: Medium priority if private lessons common

### OUT-016: Class Cancellation and Rescheduling
- **Status**: Deferred to Post-MVP
- **Description**: No automated handling of class cancellations or schedule changes.
- **Excluded Capabilities**:
  - Cancel individual class sessions
  - Reschedule classes
  - Notify enrolled students of cancellations
  - Track class cancellation history
  - Makeup class scheduling
- **Rationale**:
  - Adds complexity to scheduling model
  - Gyms can communicate changes via WhatsApp
  - Not critical for MVP
  - Timeline constraints
- **Current Approach**: Administrators communicate changes directly to students via phone/WhatsApp
- **Future Consideration**: Medium priority for post-MVP

### OUT-017: Substitute Teacher Management
- **Status**: Deferred to Post-MVP
- **Description**: No formal substitute teacher assignment process.
- **Excluded Capabilities**:
  - Assign substitute teacher for specific session
  - Track substitute teacher history
  - Notify students of substitute teacher
  - Substitute teacher approval workflow
- **Rationale**:
  - Adds complexity to class management
  - Can manually update class teacher if permanent change
  - Not critical for MVP
  - Timeline constraints
- **Current Approach**: Administrator can change assigned teacher if needed; communicate substitution externally
- **Future Consideration**: Low priority

---

## 7. Student Management Features

### OUT-018: Student Contracts and Agreements
- **Status**: Deferred to Post-MVP
- **Description**: No digital contract signing or agreement management.
- **Excluded Capabilities**:
  - Upload/store contracts
  - Digital signature capture
  - Contract templates
  - Contract expiration tracking
  - Contract renewal reminders
  - Terms and conditions acceptance
- **Rationale**:
  - Significant feature requiring document management and e-signature
  - Legal complexity
  - Not critical for operations
  - Budget and timeline constraints
- **Current Approach**: Gyms manage contracts offline (paper or separate system)
- **Future Consideration**: Low priority for MVP market

### OUT-019: Medical Information and Health Records
- **Status**: Not Planned
- **Description**: System will NOT store medical information, health conditions, or emergency contacts.
- **Excluded Capabilities**:
  - Emergency contact information
  - Medical conditions and allergies
  - Doctor contact information
  - Medical clearance documentation
  - Health questionnaires
  - Injury tracking
  - Medical alert indicators
- **Rationale**:
  - Explicitly excluded per requirements (birth date only for students)
  - HIPAA-like regulations for health data
  - Liability concerns
  - Additional security requirements
  - Not critical for basic operations
  - Privacy and compliance complexity
- **Current Approach**: Gyms maintain medical information separately (paper forms)
- **Future Consideration**: Low priority - legal and privacy complexity high

### OUT-020: Student Photos and Identification
- **Status**: Deferred to Post-MVP
- **Description**: No student photo upload or ID card generation.
- **Excluded Capabilities**:
  - Upload student photo
  - Photo gallery
  - ID card generation
  - Photo-based check-in
  - Facial recognition
- **Rationale**:
  - Requires image storage and handling
  - LGPD considerations for biometric data
  - Not critical for MVP
  - Storage costs
  - Timeline constraints
- **Current Approach**: N/A - No photos stored
- **Future Consideration**: Medium priority for post-MVP (ID cards useful)

### OUT-021: Family/Household Management
- **Status**: Deferred to Post-MVP
- **Description**: No special handling for family memberships or shared billing.
- **Excluded Capabilities**:
  - Link family members
  - Family/household accounts
  - Shared billing for family
  - Family discounts (automated)
  - Primary account holder designation
  - Dependent management
- **Rationale**:
  - Adds complexity to student and payment models
  - Can manually adjust pricing for family members
  - Not critical for MVP
  - Timeline constraints
- **Current Approach**: Register each family member separately; manually adjust pricing
- **Future Consideration**: Medium priority for post-MVP

### OUT-022: Student Skill Level and Belt Tracking
- **Status**: Deferred to Post-MVP
- **Description**: No tracking of student skill level, rank, or belt progression.
- **Excluded Capabilities**:
  - Student skill level/rank
  - Belt progression tracking
  - Promotion/graduation records
  - Skill assessment recording
  - Rank-based class filtering
  - Belt testing scheduling
- **Rationale**:
  - Not critical for administrative operations
  - Can be tracked externally
  - Adds complexity to student model
  - Timeline constraints
- **Current Approach**: Gyms track skill levels separately
- **Future Consideration**: Low priority for basic gym management (higher priority for traditional martial arts)

---

## 8. Reporting and Analytics

### OUT-023: Business Intelligence and Analytics
- **Status**: Deferred to Post-MVP
- **Description**: No advanced analytics, dashboards, or business intelligence features.
- **Excluded Capabilities**:
  - Student growth trends
  - Revenue trends and forecasting
  - Class popularity analysis
  - Teacher performance metrics
  - Student retention analysis
  - Churn prediction
  - Custom reports and queries
  - Data visualization (charts, graphs beyond basic)
  - Comparative analysis (month-over-month, year-over-year)
  - Cohort analysis
- **Rationale**:
  - Significant development effort
  - Requires data warehouse or analytics platform
  - Not critical for basic operations
  - Budget and timeline constraints
- **Current Approach**: Basic counts and lists only (dashboard shows total students, overdue count)
- **Future Consideration**: High priority for post-MVP - valuable for business insights
- **Workaround**: Data can be exported for external analysis

### OUT-024: Export and Data Integration
- **Status**: Deferred to Post-MVP
- **Description**: No data export or integration with external tools beyond basic needs.
- **Excluded Capabilities**:
  - CSV/Excel export of data
  - PDF report generation
  - API for external integrations
  - Integration with accounting software
  - Integration with marketing tools
  - Webhook support
  - Data synchronization with other systems
- **Rationale**:
  - Not critical for MVP operations
  - API design for future but no external consumers initially
  - Timeline constraints
- **Current Approach**: Data viewable in system only
- **Future Consideration**: High priority for post-MVP (especially CSV export)

### OUT-025: Custom Report Builder
- **Status**: Not Planned
- **Description**: No user-customizable report builder.
- **Rationale**:
  - Complex feature requiring report engine
  - Significant development effort
  - Not needed for MVP
  - Pre-defined reports sufficient
- **Current Approach**: Fixed views and lists
- **Future Consideration**: Low priority - pre-defined reports likely sufficient

---

## 9. Multi-Language and Localization

### OUT-026: Multi-Language Support
- **Status**: Not Planned
- **Description**: System will ONLY support Brazilian Portuguese (no other languages).
- **Excluded Capabilities**:
  - English interface
  - Spanish interface
  - Other language options
  - Language switching
  - Multi-language content management
  - Translation management
- **Rationale**:
  - Target market is Brazil only
  - Multi-language significantly increases complexity
  - Translation and maintenance costs
  - Not needed for target market
- **Current Approach**: Brazilian Portuguese only
- **Future Consideration**: Low priority unless expanding to other countries

### OUT-027: Multi-Currency Support
- **Status**: Not Planned
- **Description**: System will ONLY support Brazilian Real (BRL).
- **Rationale**:
  - Target market is Brazil only
  - No need for other currencies
  - Simplifies payment tracking
- **Current Approach**: BRL only
- **Future Consideration**: Low priority unless international expansion

---

## 10. Advanced Security and Access Control

### OUT-028: Multi-Factor Authentication (MFA)
- **Status**: Deferred to Post-MVP
- **Description**: No two-factor authentication or multi-factor authentication for MVP.
- **Excluded Capabilities**:
  - SMS-based 2FA
  - Authenticator app 2FA (TOTP)
  - Email-based 2FA
  - Backup codes
  - Biometric authentication
- **Rationale**:
  - Explicitly excluded per technical constraints
  - Adds complexity to authentication
  - Timeline constraints
  - Most small gyms don't require this level of security
- **Current Approach**: Username/password authentication only
- **Future Consideration**: High priority for post-MVP - important security enhancement
- **Risk**: Mitigate with strong password policies and account lockout

### OUT-029: Single Sign-On (SSO)
- **Status**: Not Planned
- **Description**: No SSO integration (Google, Facebook, Microsoft, etc.).
- **Rationale**:
  - Adds complexity
  - Small user base doesn't justify
  - Timeline constraints
- **Current Approach**: Local username/password authentication
- **Future Consideration**: Low priority

### OUT-030: Granular Permission Management
- **Status**: Not Planned
- **Description**: No fine-grained role-based access control or custom permissions.
- **Excluded Capabilities**:
  - Custom roles beyond Root User and Administrator
  - Permission assignment per feature/module
  - Read-only administrators
  - Limited administrators (e.g., can't delete students)
  - Temporary access delegation
- **Rationale**:
  - Two roles (Root User, Administrator) sufficient for MVP
  - Granular permissions add significant complexity
  - Small gyms typically have 1-2 administrators
  - Timeline constraints
- **Current Approach**: Two roles only - Root User (all access) and Administrator (gym-specific access)
- **Future Consideration**: Low priority for target market

### OUT-031: Audit Log User Interface
- **Status**: Deferred to Post-MVP
- **Description**: No user interface to view audit logs and user activity.
- **Excluded Capabilities**:
  - View audit log interface
  - Search and filter audit logs
  - User activity reports
  - Export audit logs
- **Rationale**:
  - Audit logging exists (backend) for compliance
  - UI not critical for MVP
  - Timeline constraints
- **Current Approach**: Audit logs exist in database; viewable by developers/support if needed
- **Future Consideration**: Medium priority for post-MVP

---

## 11. Marketing and CRM Features

### OUT-032: Lead Management
- **Status**: Not Planned
- **Description**: No lead tracking or prospective student management.
- **Excluded Capabilities**:
  - Capture prospect information
  - Lead source tracking
  - Lead status (interested, contacted, enrolled)
  - Lead follow-up reminders
  - Lead conversion tracking
  - Trial class scheduling
- **Rationale**:
  - System focuses on enrolled students, not prospects
  - CRM features out of scope
  - Gyms can use external tools (spreadsheet, CRM)
  - Timeline constraints
- **Current Approach**: N/A - System for enrolled students only
- **Future Consideration**: Low priority - separate CRM more appropriate

### OUT-033: Marketing Campaigns
- **Status**: Not Planned
- **Description**: No marketing campaign management or automation.
- **Excluded Capabilities**:
  - Email campaigns
  - SMS campaigns
  - Campaign templates
  - Campaign scheduling
  - Campaign analytics
  - Segmentation and targeting
- **Rationale**:
  - Marketing tools out of scope
  - External tools (Mailchimp, WhatsApp Business) better suited
  - Timeline constraints
- **Current Approach**: External marketing tools
- **Future Consideration**: Low priority - external tools sufficient

### OUT-034: Referral Program
- **Status**: Not Planned
- **Description**: No referral tracking or reward program.
- **Excluded Capabilities**:
  - Track referral source
  - Referral rewards/credits
  - Referral code generation
  - Referral analytics
- **Rationale**: Marketing feature out of scope
- **Current Approach**: Track manually outside system if desired
- **Future Consideration**: Low priority

---

## 12. Equipment and Inventory Management

### OUT-035: Equipment and Inventory Tracking
- **Status**: Not Planned
- **Description**: No inventory management for gym equipment or retail items.
- **Excluded Capabilities**:
  - Equipment inventory tracking
  - Equipment maintenance schedules
  - Equipment checkout/reservation
  - Retail inventory (gloves, wraps, etc.)
  - Sales tracking
  - Low stock alerts
- **Rationale**:
  - Out of scope for gym management system
  - Separate inventory system more appropriate if needed
  - Not critical for core operations
  - Significant development effort
- **Current Approach**: N/A - Not tracked in system
- **Future Consideration**: Low priority - different system type

### OUT-036: Locker Management
- **Status**: Not Planned
- **Description**: No locker assignment or management.
- **Rationale**: Out of scope, not critical
- **Current Approach**: Managed separately
- **Future Consideration**: Low priority

---

## 13. Advanced Scheduling

### OUT-037: Resource/Room Booking
- **Status**: Not Planned
- **Description**: No resource booking beyond basic room assignment to classes.
- **Excluded Capabilities**:
  - Room availability calendar
  - Resource conflict detection (beyond class scheduling)
  - Equipment booking
  - Room booking for events/private sessions
  - Resource utilization reports
- **Rationale**:
  - Classes have room assignments (sufficient for MVP)
  - Advanced resource management out of scope
  - Timeline constraints
- **Current Approach**: Room assigned to class; manual conflict avoidance
- **Future Consideration**: Low priority

### OUT-038: Calendar Integration
- **Status**: Not Planned
- **Description**: No integration with external calendars (Google Calendar, Outlook, etc.).
- **Excluded Capabilities**:
  - Export schedule to Google Calendar
  - iCal feed
  - Calendar synchronization
  - Calendar invitations
- **Rationale**:
  - Students don't access system
  - Teachers don't access system
  - Not critical for administrators
  - Integration complexity
- **Current Approach**: View schedule in system only
- **Future Consideration**: Low priority

---

## 14. Competition and Event Management

### OUT-039: Event Management
- **Status**: Not Planned
- **Description**: No event scheduling or management (competitions, seminars, belt tests).
- **Excluded Capabilities**:
  - Create and schedule events
  - Event registration
  - Event participant tracking
  - Event-specific payments
  - Event calendar
- **Rationale**:
  - Out of scope for basic gym operations
  - Can track separately
  - Significant feature
  - Timeline constraints
- **Current Approach**: Gyms manage events separately
- **Future Consideration**: Low priority

### OUT-040: Competition Tracking
- **Status**: Not Planned
- **Description**: No tracking of student competition participation or results.
- **Rationale**: Out of scope, not critical for administrative operations
- **Current Approach**: N/A
- **Future Consideration**: Low priority

---

## 15. Technical Features

### OUT-041: Mobile Native Applications
- **Status**: Not Planned (MVP)
- **Description**: No iOS or Android native applications.
- **Rationale**:
  - Explicitly excluded per technical constraints
  - Cost and complexity
  - Responsive web sufficient for MVP
- **Current Approach**: Responsive web application only
- **Future Consideration**: Medium priority post-MVP if demand exists

### OUT-042: Offline Mode
- **Status**: Not Planned
- **Description**: No offline functionality or offline data access.
- **Rationale**:
  - Online-only system
  - Adds significant complexity
  - Not critical for target users (administrators with internet)
- **Current Approach**: Requires internet connection
- **Future Consideration**: Low priority

### OUT-043: Third-Party Integrations
- **Status**: Not Planned (MVP)
- **Description**: No integrations with third-party tools beyond SMS/WhatsApp (nice-to-have).
- **Excluded Integrations**:
  - Accounting software (QuickBooks, Xero)
  - CRM systems
  - Marketing automation
  - Email marketing platforms
  - Social media
  - Analytics platforms (beyond basic)
  - Zapier/IFTTT
- **Rationale**:
  - Each integration requires development and maintenance
  - Not critical for MVP
  - Budget and timeline constraints
- **Current Approach**: Standalone system
- **Future Consideration**: Medium priority for accounting integration post-MVP

### OUT-044: API for External Developers
- **Status**: Deferred to Post-MVP
- **Description**: No public API for third-party developers.
- **Rationale**:
  - Not needed for MVP
  - Requires documentation, versioning, support
  - Internal API exists (frontend to backend)
- **Current Approach**: Internal API only (not documented for external use)
- **Future Consideration**: Low priority unless ecosystem development opportunity emerges

### OUT-045: White-Label/Multi-Brand
- **Status**: Not Planned
- **Description**: No white-label or multi-brand capabilities.
- **Rationale**:
  - Single brand (MTGMS)
  - White-label adds complexity
  - Not in business model
- **Current Approach**: Single brand
- **Future Consideration**: Low priority

---

## 16. Out of Scope Summary

### By Category

#### Student/Teacher Features (5 items)
- Student portal/self-service
- Teacher portal/self-service
- Teacher mobile app
- Student mobile app (not listed separately, covered by web-only)
- In-app messaging

#### Attendance and Presence (2 items)
- Attendance tracking
- Attendance-based billing

#### Payments and Finance (5 items)
- Online payment processing
- Financial reporting and analytics
- Invoicing and receipts
- Discounts and promotions
- Referral programs

#### Communication (5 items)
- In-app messaging
- Email notifications
- Push notifications
- Announcement system
- Marketing campaigns

#### Class Management (4 items)
- Waitlist functionality
- Private/semi-private class special handling
- Class cancellation/rescheduling automation
- Substitute teacher management

#### Student Management (5 items)
- Contracts and agreements
- Medical information and health records
- Student photos and ID cards
- Family/household management
- Skill level and belt tracking

#### Reporting (3 items)
- Business intelligence and analytics
- Export and data integration
- Custom report builder

#### Localization (2 items)
- Multi-language support
- Multi-currency support

#### Security (4 items)
- Multi-factor authentication (MFA)
- Single sign-on (SSO)
- Granular permission management
- Audit log user interface

#### Marketing/CRM (3 items)
- Lead management
- Marketing campaigns
- Referral program

#### Operations (3 items)
- Equipment and inventory management
- Locker management
- Resource/room booking (advanced)

#### Events (2 items)
- Event management
- Competition tracking

#### Technical (6 items)
- Native mobile applications
- Offline mode
- Third-party integrations (beyond SMS/WhatsApp)
- Public API for external developers
- White-label/multi-brand
- Calendar integration

### Total Out-of-Scope Items: 49

---

## 17. Future Prioritization

### High Priority for Post-MVP
1. Attendance tracking (OUT-004)
2. Financial reporting and analytics (OUT-007)
3. Student portal (OUT-001)
4. Multi-factor authentication (OUT-028)
5. CSV/Excel export (OUT-024)

### Medium Priority for Post-MVP
6. Teacher portal (OUT-002)
7. Online payment processing (OUT-006)
8. Invoicing and receipts (OUT-008)
9. Student photos and ID cards (OUT-020)
10. Family/household management (OUT-021)
11. Class cancellation/rescheduling (OUT-016)
12. Accounting software integration (OUT-043)
13. Private class handling (OUT-015)
14. Audit log UI (OUT-031)
15. Discounts and promotions (OUT-009)

### Low Priority or Not Planned
16. Everything else (29 items)

---

## 18. Boundary Setting

### What This System IS:
- Gym administrative management tool
- Student and teacher profile management
- Class scheduling and enrollment
- Payment tracking (manual/offline)
- Basic dashboard and reporting

### What This System IS NOT:
- CRM or lead management system
- Point-of-sale or inventory system
- Marketing automation platform
- Online payment processor
- Student/teacher mobile app
- Attendance tracking system (MVP)
- Event management system
- Equipment/facility management system

---

## 19. Stakeholder Communication

### How to Use This Document
- Reference when stakeholders request features
- Set clear expectations in sales/onboarding
- Manage scope creep during development
- Plan future roadmap based on priorities
- Justify MVP scope decisions

### Template Response for Out-of-Scope Requests
"Thank you for the suggestion regarding [feature]. This feature is documented as out-of-scope for the MVP in our requirements document (OUT-XXX). The reason for exclusion is [rationale]. We've noted it as [priority] priority for post-MVP consideration. For now, the recommended approach is [workaround/alternative]."

---

**Document Revision History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-10-16 | Requirements Team | Initial version |
