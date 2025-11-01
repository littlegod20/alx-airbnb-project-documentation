# Use Case Diagram - Airbnb Clone Backend

## Actors

### Primary Actors
1. **Guest** - End user who searches and books properties
2. **Host** - Property owner who lists and manages properties
3. **Admin** - System administrator who manages the platform

### Secondary Actors (External Systems)
4. **Payment Gateway** - Stripe/PayPal for payment processing
5. **Email Service** - SendGrid/Mailgun for notifications

---

## Use Cases by Actor

### GUEST Use Cases

#### Authentication & Profile
- **UC1: Register Account**
  - Guest creates new account
  - Includes: Email verification
  - Precondition: None
  - Postcondition: Account created, verification email sent

- **UC2: Login/Logout**
  - Guest authenticates with credentials or OAuth
  - Precondition: Valid account exists
  - Postcondition: Session established/terminated

- **UC3: Manage Profile**
  - Guest updates personal information, preferences
  - Precondition: Logged in
  - Postcondition: Profile updated

- **UC4: Reset Password**
  - Guest requests password reset
  - Includes: Email service
  - Precondition: Valid email exists
  - Postcondition: Reset link sent

#### Property Discovery
- **UC5: Search Properties**
  - Guest searches by location, dates, price, amenities
  - Precondition: None (can be guest or visitor)
  - Postcondition: Filtered results displayed

- **UC6: View Property Details**
  - Guest views full property information
  - Precondition: None
  - Postcondition: Property details displayed

#### Booking
- **UC11: Create Booking**
  - Guest books property for specific dates
  - Includes: Check availability, Process payment
  - Precondition: Logged in, property available
  - Postcondition: Booking created, payment processed

- **UC12: View Bookings**
  - Guest views their booking history
  - Precondition: Logged in
  - Postcondition: Booking list displayed

- **UC13: Cancel Booking**
  - Guest cancels existing booking
  - Extends: Issue refund
  - Precondition: Active booking exists
  - Postcondition: Booking canceled, refund initiated

- **UC15: Check Availability**
  - System validates property availability
  - Precondition: Property and dates selected
  - Postcondition: Availability confirmed/denied

#### Payment
- **UC16: Process Payment**
  - Guest pays for booking
  - Actor: Payment Gateway
  - Precondition: Valid payment method
  - Postcondition: Payment completed

- **UC17: View Transaction History**
  - Guest views payment records
  - Precondition: Logged in
  - Postcondition: Transaction list displayed

#### Reviews
- **UC20: Write Review**
  - Guest reviews property after stay
  - Includes: Rate property
  - Precondition: Completed booking
  - Postcondition: Review published

- **UC21: View Reviews**
  - Guest reads property reviews
  - Precondition: None
  - Postcondition: Reviews displayed

- **UC23: Rate Property**
  - Guest gives 1-5 star rating
  - Precondition: Completed booking
  - Postcondition: Rating recorded

#### Communication
- **UC24: Send Message**
  - Guest messages host
  - Precondition: Logged in
  - Postcondition: Message sent

- **UC25: Receive Notifications**
  - Guest receives booking/payment updates
  - Actor: Email Service
  - Precondition: Valid email
  - Postcondition: Notification delivered

---

### HOST Use Cases

#### Property Management
- **UC7: Add Property Listing**
  - Host creates new property listing
  - Includes: Manage availability
  - Precondition: Logged in as host
  - Postcondition: Property listed

- **UC8: Edit Property Listing**
  - Host updates property details
  - Precondition: Property exists, owner logged in
  - Postcondition: Property updated

- **UC9: Delete Property Listing**
  - Host removes property from platform
  - Precondition: Property exists, no active bookings
  - Postcondition: Property deleted

- **UC10: Manage Availability**
  - Host sets available/blocked dates
  - Precondition: Property exists
  - Postcondition: Calendar updated

#### Booking Management
- **UC12: View Bookings**
  - Host views all property bookings
  - Precondition: Logged in as host
  - Postcondition: Booking list displayed

- **UC14: Approve/Reject Booking**
  - Host confirms or declines booking request
  - Includes: Send notification
  - Precondition: Pending booking exists
  - Postcondition: Booking status updated

#### Financial
- **UC19: Receive Payout**
  - Host receives payment after booking completion
  - Actor: Payment Gateway
  - Precondition: Completed booking
  - Postcondition: Funds transferred

- **UC17: View Transaction History**
  - Host views earnings and payouts
  - Precondition: Logged in
  - Postcondition: Financial records displayed

#### Reviews
- **UC22: Respond to Review**
  - Host replies to guest review
  - Precondition: Review exists
  - Postcondition: Response published

---

### ADMIN Use Cases

#### User Management
- **UC27: Manage Users**
  - Admin views, suspends, or deletes user accounts
  - Precondition: Logged in as admin
  - Postcondition: User status updated

#### Listing Management
- **UC28: Monitor Listings**
  - Admin reviews all property listings
  - Precondition: Logged in as admin
  - Postcondition: Listing status reviewed

- **UC31: Approve Listings**
  - Admin approves/rejects new listings
  - Precondition: Pending listing exists
  - Postcondition: Listing status updated

#### Operations
- **UC29: Handle Disputes**
  - Admin mediates conflicts between users
  - Precondition: Dispute reported
  - Postcondition: Resolution recorded

- **UC30: Generate Reports**
  - Admin creates analytics and financial reports
  - Precondition: Data exists
  - Postcondition: Report generated

- **UC32: Manage Payments**
  - Admin oversees transactions and issues refunds
  - Precondition: Transaction exists
  - Postcondition: Payment adjusted

- **UC18: Issue Refund**
  - Admin manually processes refund
  - Actor: Payment Gateway
  - Extends: Cancel booking
  - Precondition: Valid refund reason
  - Postcondition: Refund processed

---

## Relationships

### Include Relationships (mandatory)
- Register Account **includes** Verify Email
- Create Booking **includes** Check Availability
- Create Booking **includes** Process Payment
- Write Review **includes** Rate Property
- Approve Booking **includes** Send Notification

### Extend Relationships (optional)
- Cancel Booking **extends** Issue Refund
- Process Payment **extends** Receive Payout

### Associations
- All use cases interact with external systems (Payment Gateway, Email Service) as needed

---

## System Boundary
All use cases occur within the **Airbnb Clone System** boundary, with external actors (Payment Gateway, Email Service) interacting from outside.


![alt text](<use-case.png>)