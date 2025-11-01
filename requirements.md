# ARBAR Clone System - Technical Documentation

## System Overview

ARBAR is a comprehensive property booking platform that connects guests with property hosts, facilitating seamless booking experiences with integrated payment processing and management capabilities.

## System Architecture

### Core Components

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Gateway    │    │   Microservices │
│   Client        │────│   & Load         │────│   Architecture  │
│                 │    │   Balancer       │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
               ┌──────────────┼──────────────┐
               │              │              │
         ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐
         │  Auth     │  │  Property │  │  Booking  │
         │ Service   │  │  Service  │  │  Service  │
         └───────────┘  └───────────┘  └───────────┘
               │              │              │
         ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐
         │  Payment  │  │  Search   │  │ Notification│
         │ Service   │  │  Service  │  │  Service   │
         └───────────┘  └───────────┘  └───────────┘
```

## User Registration & Authentication

### 1. User Authentication System

#### 1.1 Overview
The User Authentication System manages user registration, login, email verification, and session management for guests, hosts, and admins.

#### 1.2 Functional Requirements

**FR-AUTH-001: User Registration**
- **Description**: Allow users to create accounts as guests or hosts
- **Business Rules**:
  - Email must be unique across the system
  - Password must meet security requirements
  - Phone number is optional
  - Email verification is mandatory before account activation
  - Host accounts require admin approval after email verification

**User Stories**:
- As a guest, I want to register with email/password so I can book properties
- As a host, I want to register so I can list my properties

**FR-AUTH-002: User Login**
- **Description**: Authenticate users and provide secure access tokens
- **Business Rules**:
  - Support email/password authentication
  - Support OAuth 2.0 (Google, Facebook)
  - Failed login attempts are rate-limited (5 attempts per 15 minutes)
  - JWT tokens expire after 24 hours
  - Refresh tokens valid for 30 days

**FR-AUTH-003: Email Verification**
- **Description**: Verify user email addresses through token-based verification
- **Business Rules**:
  - Verification token valid for 24 hours
  - Users can request token resend (max 3 times per hour)
  - Unverified accounts cannot perform main actions

**FR-AUTH-004: Password Reset**
- **Description**: Allow users to reset forgotten passwords securely
- **Business Rules**:
  - Reset token valid for 1 hour
  - Old password cannot be reused
  - Rate limit: 3 reset requests per hour per email

#### 1.3 API Endpoints

**1.3.1 POST /api/v1/auth/register**
```json
{
  "first_name": "string (required, 2-50 chars)",
  "last_name": "string (required, 2-50 chars)",
  "email": "string (required, valid email format)",
  "password": "string (required, 8-128 chars)",
  "phone_number": "string (optional, E.164 format)",
  "role": "enum (required: 'guest' | 'host')"
}
```

**1.3.2 POST /api/v1/auth/login**
```json
{
  "email": "string (required)",
  "password": "string (required)"
}
```

**1.3.3 POST /api/v1/auth/verify-email**
```json
{
  "token": "string (required, UUID format)"
}
```

**1.3.4 POST /api/v1/auth/refresh-token**
```json
{
  "refresh_token": "string (required)"
}
```

**1.3.5 POST /api/v1/auth/logout**
```json
{
  "refresh_token": "string (optional)"
}
```

**1.3.6 POST /api/v1/auth/password-reset/request**
```json
{
  "email": "string (required)"
}
```

**1.3.7 POST /api/v1/auth/password-reset/confirm**
```json
{
  "token": "string (required)",
  "new_password": "string (required)"
}
```

#### 1.4 Non-Functional Requirements

**NFR-AUTH-001: Security**
- Passwords hashed using bcrypt (cost factor 12)
- JWT tokens signed with RS256 algorithm
- HTTPS required for all endpoints
- Rate limiting on all authentication endpoints
- Failed login attempts logged for security monitoring
- CORS configured for allowed origins only

**NFR-AUTH-002: Performance**
- 99.9% uptime SLA
- Support 1000 concurrent users
- Response times:
  - Login: < 300ms (P95)
  - Registration: < 500ms (P95)
  - Token refresh: < 200ms (P95)
- Database queries optimized with indexes on email field

**NFR-AUTH-003: Scalability**
- Stateless authentication (JWT)
- Horizontally scalable
- Redis for token blacklist and rate limiting
- Database connection pooling

**NFR-AUTH-004: Compliance**
- GDPR compliant (user data deletion on request)
- Password storage follows OWASP guidelines
- PCI DSS compliant (no credit card data stored)

#### 1.5 Data Models

**User Table**
```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20),
    role ENUM('guest', 'host', 'admin') NOT NULL,
    email_verified BOOLEAN DEFAULT FALSE,
    email_verified_at TIMESTAMP,
    account_status ENUM('active', 'suspended', 'pending') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP,
    
    INDEX idx_email (email),
    INDEX idx_role (role),
    INDEX idx_created_at (created_at)
);
```

**Verification Tokens Table**
```sql
CREATE TABLE verification_tokens (
    token_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    token UUID UNIQUE NOT NULL,
    token_type ENUM('email_verification', 'password_reset') NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    used_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    INDEX idx_token (token),
    INDEX idx_expires_at (expires_at)
);
```

#### 1.6 Error Codes
| Code | HTTP Status | Description |
|------|-------------|-------------|
| AUTH_INVALID_CREDENTIALS | 401 | Invalid email or password |
| AUTH_EMAIL_EXISTS | 400 | Email already registered |
| AUTH_EMAIL_NOT_VERIFIED | 403 | Email verification required |
| AUTH_INVALID_TOKEN | 400 | Invalid or expired token |
| AUTH_ACCOUNT_LOCKED | 423 | Account locked due to failed attempts |

## 2. Property Management System

### 2.1 Overview
The Property Management System allows hosts to create, update, and manage property listings, and enables guests to search and view properties.

### 2.2 Functional Requirements

**FR-PROP-001: Create Property Listing**
- **Description**: Hosts can create new property listings with details, images, and pricing
- **Business Rules**:
  - Only verified hosts can create listings
  - Minimum 3 images required
  - Property must have valid location
  - Price must be greater than $0
  - Admin approval required before listing goes live

**FR-PROP-002: Search Properties**
- **Description**: Users can search properties by various criteria
- **Business Rules**:
  - Search available to all users (authenticated or not)
  - Results paginated (20 per page)
  - Results cached for 5 minutes
  - Only active/approved listings shown

**FR-PROP-003: Update Property**
- **Description**: Hosts can update their property details
- **Business Rules**:
  - Only property owner can update
  - Major changes require re-approval
  - Cannot update if active bookings exist for changed dates

**FR-PROP-004: Delete Property**
- **Description**: Hosts can delete property listings
- **Business Rules**:
  - Cannot delete if active bookings exist
  - Soft delete (data retained for 90 days)
  - Guest bookings must be canceled first

### 2.3 API Endpoints

**2.3.1 POST /api/v1/properties**
```json
{
  "name": "string (required, 10-200 chars)",
  "description": "string (required, 50-5000 chars)",
  "property_type": "enum (required: 'apartment' | 'house' | 'villa' | 'condo' | 'cabin')",
  "location": {
    "address": "string (required)",
    "city": "string (required)",
    "state": "string (optional)",
    "country": "string (required)",
    "zip_code": "string (optional)",
    "latitude": "number (required, -90 to 90)",
    "longitude": "number (required, -180 to 180)"
  },
  "pricing": {
    "base_price": "number (required, > 0)",
    "currency": "string (required, ISO 4217)",
    "cleaning_fee": "number (optional, >= 0)",
    "service_fee_percentage": "number (optional, 0-100)"
  },
  "details": {
    "bedrooms": "integer (required, >= 0)",
    "bathrooms": "number (required, >= 0)",
    "beds": "integer (required, >= 0)",
    "max_guests": "integer (required, >= 1)",
    "square_feet": "integer (optional, > 0)"
  },
  "amenities": "array[string] (required, min 1 item)",
  "house_rules": "string (optional, max 2000 chars)",
  "cancellation_policy": "enum (required: 'flexible' | 'moderate' | 'strict')",
  "instant_book": "boolean (optional, default: false)",
  "min_nights": "integer (optional, default: 1, >= 1)",
  "max_nights": "integer (optional, >= min_nights)"
}
```

**2.3.2 POST /api/v1/properties/{property_id}/images**
- **Content-Type**: multipart/form-data
- **Files**: images (required, min 3, max 20 files)
- **Validation**: JPEG, PNG, WebP only, max 10MB per image

**2.3.3 GET /api/v1/properties/search**
- **Query Parameters**: location, latitude, longitude, radius, check_in, check_out, guests, min_price, max_price, bedrooms, bathrooms, property_type, amenities, instant_book, sort_by, page, page_size

**2.3.4 GET /api/v1/properties/{property_id}**
- **Path Parameters**: property_id (required, UUID)

**2.3.5 PUT /api/v1/properties/{property_id}**
- **Authorization**: Property owner required
- **Body**: Same structure as create, all fields optional

**2.3.6 DELETE /api/v1/properties/{property_id}**
- **Authorization**: Property owner or admin required
- **Validation**: Cannot delete if active bookings exist

**2.3.7 GET /api/v1/properties/{property_id}/availability**
- **Query Parameters**: check_in (required), check_out (required)

### 2.4 Non-Functional Requirements

**NFR-PROP-001: Performance**
- Search response: < 500ms (P95)
- Property details: < 300ms (P95)
- Support 10,000 concurrent property searches
- Image CDN delivery with 99.9% uptime
- Database query optimization with proper indexes

**NFR-PROP-002: Scalability**
- Support 100,000+ property listings
- Horizontal scaling for API servers
- Database read replicas for search queries
- CDN for static images
- Elasticsearch for full-text search (optional enhancement)

**NFR-PROP-003: Data Integrity**
- Foreign key constraints enforced
- Transaction management for multi-table operations
- Audit logging for all property changes
- Soft delete with 90-day retention

**NFR-PROP-004: Security**
- Authorization checks on all endpoints
- Input sanitization to prevent XSS
- SQL injection prevention
- Rate limiting on search (100 requests/minute per IP)

### 2.5 Data Models

**Property Table**
```sql
CREATE TABLE properties (
    property_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_id UUID NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT NOT NULL,
    property_type ENUM('apartment', 'house', 'villa', 'condo', 'cabin') NOT NULL,
    
    -- Location
    address TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100),
    country VARCHAR(100) NOT NULL,
    zip_code VARCHAR(20),
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    
    -- Pricing
    base_price DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    cleaning_fee DECIMAL(10, 2) DEFAULT 0,
    service_fee_percentage DECIMAL(5, 2) DEFAULT 0,
    
    -- Details
    bedrooms INTEGER NOT NULL,
    bathrooms DECIMAL(3, 1) NOT NULL,
    beds INTEGER NOT NULL,
    max_guests INTEGER NOT NULL,
    square_feet INTEGER,
    
    -- Rules & Policies
    house_rules TEXT,
    cancellation_policy ENUM('flexible', 'moderate', 'strict') NOT NULL,
    instant_book BOOLEAN DEFAULT FALSE,
    min_nights INTEGER DEFAULT 1,
    max_nights INTEGER,
    
    -- Status
    status ENUM('draft', 'pending_approval', 'active', 'suspended', 'deleted') DEFAULT 'draft',
    approved_at TIMESTAMP,
    approved_by UUID,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    
    -- Foreign keys
    FOREIGN KEY (host_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (approved_by) REFERENCES users(user_id) ON DELETE SET NULL,
    
    -- Indexes
    INDEX idx_host_id (host_id),
    INDEX idx_property_type (property_type),
    INDEX idx_status (status),
    INDEX idx_location (city, country),
    INDEX idx_price (base_price),
    INDEX idx_created_at (created_at),
    SPATIAL INDEX idx_coordinates (latitude, longitude)
);
```

## 3. Booking Management System

### 3.1 Overview
The Booking Management System handles the complete booking lifecycle from creation to completion, including availability checks, pricing calculations, and status management.

### 3.2 Functional Requirements

**FR-BOOK-001: Create Booking**
- **Description**: Guests can create bookings for available properties
- **Business Rules**:
  - Property must be available for selected dates
  - Minimum night stay requirements must be met
  - Instant book properties can be booked without host approval
  - Non-instant book properties require host approval
  - Payment authorization required at booking time

**FR-BOOK-002: Cancel Booking**
- **Description**: Users can cancel bookings according to cancellation policies
- **Business Rules**:
  - Guests can cancel their own bookings
  - Hosts can cancel with valid reasons (penalties may apply)
  - Refund amount determined by cancellation policy and timing
  - Cancellation fees may apply based on policy

**FR-BOOK-003: View Bookings**
- **Description**: Users can view their booking history and details
- **Business Rules**:
  - Guests can view their own bookings
  - Hosts can view bookings for their properties
  - Admins can view all bookings
  - Sensitive information masked based on user role

**FR-BOOK-004: Update Booking**
- **Description**: Modify booking details (dates, guest count)
- **Business Rules**:
  - Only pending or confirmed bookings can be modified
  - Changes subject to availability and host approval
  - Price recalculated for changes
  - Modification fees may apply

### 3.3 API Endpoints

**3.3.1 POST /api/v1/bookings**
```json
{
  "property_id": "uuid (required)",
  "check_in": "date (required, ISO 8601)",
  "check_out": "date (required, ISO 8601)",
  "guests": "integer (required, >= 1)",
  "special_requests": "string (optional, max 1000 chars)",
  "payment_method_id": "string (required for instant book)"
}
```

**3.3.2 GET /api/v1/bookings**
- **Query Parameters**: status, page, page_size, sort_by
- **Authorization**: User can only see their own bookings

**3.3.3 GET /api/v1/bookings/{booking_id}**
- **Path Parameters**: booking_id (required, UUID)
- **Authorization**: Booking participant (guest/host) or admin

**3.3.4 POST /api/v1/bookings/{booking_id}/cancel**
```json
{
  "cancellation_reason": "string (required)",
  "cancelled_by": "enum (required: 'guest' | 'host' | 'system')"
}
```

**3.3.5 PUT /api/v1/bookings/{booking_id}**
```json
{
  "check_in": "date (optional)",
  "check_out": "date (optional)",
  "guests": "integer (optional)",
  "special_requests": "string (optional)"
}
```

**3.3.6 POST /api/v1/bookings/{booking_id}/approve**
- **Authorization**: Property host only
- **Description**: Approve a pending booking request

**3.3.7 POST /api/v1/bookings/{booking_id}/reject**
```json
{
  "rejection_reason": "string (required)"
}
```

### 3.4 Booking States
```
Pending → Approved → Confirmed → Completed
         ↘ Rejected ↗
         ↘ Cancelled ↗
```

### 3.5 Data Models

**Bookings Table**
```sql
CREATE TABLE bookings (
    booking_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID NOT NULL,
    guest_id UUID NOT NULL,
    
    -- Booking Details
    check_in DATE NOT NULL,
    check_out DATE NOT NULL,
    guests INTEGER NOT NULL,
    nights INTEGER NOT NULL,
    special_requests TEXT,
    
    -- Pricing
    base_price DECIMAL(10, 2) NOT NULL,
    cleaning_fee DECIMAL(10, 2) DEFAULT 0,
    service_fee DECIMAL(10, 2) DEFAULT 0,
    tax_amount DECIMAL(10, 2) DEFAULT 0,
    total_amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    
    -- Status
    status ENUM('pending', 'approved', 'confirmed', 'cancelled', 'completed', 'rejected') NOT NULL,
    cancellation_policy ENUM('flexible', 'moderate', 'strict') NOT NULL,
    cancellation_reason TEXT,
    cancelled_by ENUM('guest', 'host', 'system'),
    cancelled_at TIMESTAMP,
    
    -- Payment
    payment_status ENUM('pending', 'authorized', 'paid', 'refunded', 'failed') NOT NULL,
    payment_intent_id VARCHAR(255),
    refund_amount DECIMAL(10, 2) DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    confirmed_at TIMESTAMP,
    completed_at TIMESTAMP,
    
    -- Foreign keys
    FOREIGN KEY (property_id) REFERENCES properties(property_id) ON DELETE CASCADE,
    FOREIGN KEY (guest_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    -- Indexes
    INDEX idx_property_id (property_id),
    INDEX idx_guest_id (guest_id),
    INDEX idx_status (status),
    INDEX idx_dates (check_in, check_out),
    INDEX idx_created_at (created_at)
);
```

**Booking Calendar Table**
```sql
CREATE TABLE booking_calendar (
    calendar_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID NOT NULL,
    date DATE NOT NULL,
    status ENUM('available', 'booked', 'blocked') NOT NULL,
    booking_id UUID,
    rate DECIMAL(10, 2),
    
    FOREIGN KEY (property_id) REFERENCES properties(property_id) ON DELETE CASCADE,
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id) ON DELETE SET NULL,
    
    UNIQUE KEY unique_property_date (property_id, date),
    INDEX idx_date (date),
    INDEX idx_status (status)
);
```

## 4. Payment System

### 4.1 Overview
The Payment System handles all financial transactions including booking payments, refunds, and host payouts through integrated payment gateways.

### 4.2 Functional Requirements

**FR-PAY-001: Process Payment**
- **Description**: Process booking payments through payment gateway
- **Business Rules**:
  - Support multiple payment methods (Stripe, PayPal)
  - Authorization hold at booking time
  - Capture payment upon booking confirmation
  - Support 3D Secure authentication
  - Handle payment failures gracefully

**FR-PAY-002: Issue Refunds**
- **Description**: Process refunds according to cancellation policies
- **Business Rules**:
  - Automatic refund calculation based on cancellation timing
  - Partial refunds supported
  - Refund processing within 5-7 business days
  - Refund status tracking

**FR-PAY-003: Host Payouts**
- **Description**: Transfer funds to host accounts after stay completion
- **Business Rules**:
  - Payouts processed 24 hours after check-out
  - Minimum payout amount: $25
  - Support multiple payout methods (bank transfer, PayPal)
  - Payout fee deduction support

**FR-PAY-004: Payment Security**
- **Description**: Ensure secure payment processing
- **Business Rules**:
  - PCI DSS compliance
  - No sensitive payment data stored
  - Tokenization for payment methods
  - Fraud detection and prevention

### 4.3 API Endpoints

**4.3.1 POST /api/v1/payments/create-intent**
```json
{
  "booking_id": "uuid (required)",
  "payment_method": "enum (required: 'card' | 'paypal')",
  "return_url": "string (required for 3D Secure)"
}
```

**4.3.2 POST /api/v1/payments/confirm**
```json
{
  "payment_intent_id": "string (required)",
  "payment_method_id": "string (required)"
}
```

**4.3.3 POST /api/v1/payments/refund**
```json
{
  "booking_id": "uuid (required)",
  "amount": "number (optional)",
  "reason": "string (required)"
}
```

**4.3.4 GET /api/v1/payments/payouts**
- **Query Parameters**: status, date_from, date_to, page
- **Authorization**: Host can only see their own payouts

### 4.4 Data Models

**Payments Table**
```sql
CREATE TABLE payments (
    payment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    booking_id UUID NOT NULL,
    guest_id UUID NOT NULL,
    
    -- Payment Details
    amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    payment_method ENUM('card', 'paypal', 'bank_transfer') NOT NULL,
    payment_status ENUM('pending', 'processing', 'completed', 'failed', 'refunded') NOT NULL,
    
    -- Gateway Information
    gateway_payment_id VARCHAR(255),
    gateway_customer_id VARCHAR(255),
    payment_intent_id VARCHAR(255),
    
    -- Security
    last4 CHAR(4),
    brand VARCHAR(50),
    country VARCHAR(2),
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    
    -- Foreign keys
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id) ON DELETE CASCADE,
    FOREIGN KEY (guest_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    -- Indexes
    INDEX idx_booking_id (booking_id),
    INDEX idx_guest_id (guest_id),
    INDEX idx_status (payment_status),
    INDEX idx_created_at (created_at)
);
```

**Payouts Table**
```sql
CREATE TABLE payouts (
    payout_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_id UUID NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    status ENUM('pending', 'processing', 'completed', 'failed') NOT NULL,
    payout_method ENUM('bank_transfer', 'paypal') NOT NULL,
    gateway_payout_id VARCHAR(255),
    failure_reason TEXT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    processed_at TIMESTAMP,
    completed_at TIMESTAMP,
    
    FOREIGN KEY (host_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    INDEX idx_host_id (host_id),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
);
```

## 5. Notification System

### 5.1 Overview
The Notification System handles all communication with users including email notifications, in-app alerts, and push notifications.

### 5.2 Integration Points

**Email Service Integration** (SendGrid/Mailgun)
- Registration verification emails
- Booking confirmation notifications
- Payment receipts
- Host booking requests
- Cancellation notifications

**In-App Notifications**
- Booking status updates
- Host responses
- Payment confirmations
- System announcements

### 5.3 Notification Types

**Email Templates:**
- `WELCOME_EMAIL` - New user registration
- `EMAIL_VERIFICATION` - Email verification link
- `BOOKING_REQUEST` - New booking request for host
- `BOOKING_CONFIRMED` - Booking confirmation for guest
- `BOOKING_CANCELLED` - Cancellation notification
- `PAYMENT_RECEIPT` - Payment confirmation
- `HOST_PAYOUT` - Payout notification for hosts

## 6. Admin Management System

### 6.1 Overview
The Admin Management System provides administrative capabilities for system monitoring, user management, and content moderation.

### 6.2 Functional Requirements

**FR-ADMIN-001: User Management**
- View and manage user accounts
- Approve/reject host registrations
- Suspend problematic accounts
- View user activity logs

**FR-ADMIN-002: Property Moderation**
- Approve/reject property listings
- Review reported properties
- Manage property categories and amenities
- Bulk operations on properties

**FR-ADMIN-003: Booking Monitoring**
- View all bookings in the system
- Resolve booking disputes
- Manual booking adjustments
- Financial reporting

**FR-ADMIN-004: System Analytics**
- Dashboard with key metrics
- Revenue reports
- User growth analytics
- Property performance metrics

### 6.3 API Endpoints (Admin Only)

**6.3.1 GET /api/v1/admin/users**
- **Query Parameters**: role, status, date_from, date_to, page

**6.3.2 PUT /api/v1/admin/users/{user_id}/status**
```json
{
  "account_status": "enum (required: 'active' | 'suspended')",
  "reason": "string (required for suspension)"
}
```

**6.3.3 GET /api/v1/admin/properties/pending**
- **Query Parameters**: page, page_size

**6.3.4 POST /api/v1/admin/properties/{property_id}/approve**
```json
{
  "approval_notes": "string (optional)"
}
```

**6.3.5 POST /api/v1/admin/properties/{property_id}/reject**
```json
{
  "rejection_reason": "string (required)"
}
```

## 7. System Integration & Workflows

### 7.1 Booking Workflow

```
1. Guest searches for properties
2. Guest selects dates and creates booking request
3. System checks availability and calculates price
4. For instant book: Payment authorized immediately
5. For regular book: Host receives notification
6. Host approves/rejects within 24 hours
7. If approved: Payment captured, confirmation sent
8. If rejected: Authorization released, notification sent
9. Post-stay: Review period opens, host payout processed
```

### 7.2 Payment Flow

```
1. Payment intent created with booking
2. Guest provides payment method
3. Authorization hold placed
4. Upon confirmation: Payment captured
5. Upon cancellation: Refund processed based on policy
6. 24h after check-out: Host payout initiated
7. Payout processed to host's preferred method
```

### 7.3 Notification Flow

```
User Action → Event Trigger → Notification Service → 
    ↓
Email Service / In-App Notification / Push Notification
    ↓
User Receives Notification
```

## 8. Security & Compliance

### 8.1 Security Measures

**Data Protection:**
- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- Secure password hashing (bcrypt)
- Payment data tokenization

**Access Control:**
- Role-based access control (RBAC)
- JWT token authentication
- API rate limiting
- CORS configuration

**Monitoring & Logging:**
- Comprehensive audit logging
- Security event monitoring
- Failed login tracking
- Suspicious activity detection

### 8.2 Compliance Requirements

**GDPR Compliance:**
- User data access/deletion requests
- Data processing agreements
- Privacy policy compliance
- Cookie consent management

**PCI DSS Compliance:**
- No sensitive card data storage
- Secure payment processing
- Regular security assessments
- Vulnerability management

## 9. Deployment & Infrastructure

### 9.1 Technology Stack

**Backend:**
- Node.js with TypeScript
- Express.js framework
- PostgreSQL database
- Redis for caching
- JWT for authentication

**Frontend:**
- React with TypeScript
- Tailwind CSS for styling
- React Query for state management
- Axios for API calls

**Infrastructure:**
- Docker containerization
- AWS/Google Cloud deployment
- CDN for static assets
- Load balancers
- Database replication

### 9.2 Monitoring & Analytics

**Application Monitoring:**
- Application performance monitoring (APM)
- Error tracking and reporting
- User behavior analytics
- Performance metrics

**Business Analytics:**
- Booking conversion rates
- Revenue tracking
- User acquisition metrics
- Property performance

This comprehensive documentation provides the foundation for implementing the ARBAR clone system with all necessary technical specifications, API definitions, and system workflows.