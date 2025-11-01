# UML Use Case Diagram - Airbnb Clone System

## Actors

### Primary Actors (Left Side)
1. **Guest** - End user who books properties
2. **Host** - Property owner who lists properties
3. **Admin** - System administrator

### External Actors (Right Side)
4. **Payment Gateway** - Stripe/PayPal for processing payments
5. **Email Service** - SendGrid/Mailgun for notifications

---

## Use Cases

### 1. User Registration
- **Register as Guest** (Guest)
- **Register as Host** (Host)
- **Verify Email** (System) - Included in both registrations
- **Approve Account** (Admin) - Verifies new registrations

### 2. Property Booking
- **Search Properties** (Guest)
- **Create Booking** (Guest)
- **Approve Booking** (Host)
- **View Bookings** (Guest, Host)
- **Cancel Booking** (Guest, Host)
- **Monitor Bookings** (Admin)

### 3. Payments
- **Process Payment** (Guest) - Included in Create Booking
- **Receive Payout** (Host)
- **View Transactions** (Guest, Host)
- **Issue Refund** (Admin) - Extended from Cancel Booking
- **Manage Payments** (Admin)

---

## Relationships

### Include (<<include>>)
- Register as Guest **includes** Verify Email
- Register as Host **includes** Verify Email
- Create Booking **includes** Process Payment

### Extend (<<extend>>)
- Cancel Booking **extends** Issue Refund (optional refund processing)

### Associations
- **Guest**: 7 direct connections
- **Host**: 5 direct connections
- **Admin**: 4 direct connections
- **Payment Gateway**: 3 connections (payment, payout, refund)
- **Email Service**: 3 connections (verify, approve, notify)

---

## Actor Actions Summary

| Use Case | Guest | Host | Admin |
|----------|-------|------|-------|
| Register as Guest | ✓ | | |
| Register as Host | | ✓ | |
| Verify Email | (included) | (included) | |
| Approve Account | | | ✓ |
| Search Properties | ✓ | | |
| Create Booking | ✓ | | |
| Approve Booking | | ✓ | |
| View Bookings | ✓ | ✓ | |
| Cancel Booking | ✓ | ✓ | |
| Monitor Bookings | | | ✓ |
| Process Payment | ✓ | | |
| Receive Payout | | ✓ | |
| View Transactions | ✓ | ✓ | |
| Issue Refund | | | ✓ |
| Manage Payments | | | ✓ |