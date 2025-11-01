## Level 0 Context Diagram
```
flowchart TD
    %% External Entities
    Guest[Guest User]
    Host[Property Host]
    Admin[System Admin]
    PaymentGateway[Payment Gateway<br/>Stripe/PayPal]
    EmailService[Email Service<br/>SendGrid/Mailgun]
    
    %% System Boundary
    subgraph ARBAR_SYSTEM [ARBAR Property Booking Platform]
        direction TB
        UserManagement[User Management]
        PropertyManagement[Property Management]
        BookingManagement[Booking Management]
        PaymentProcessing[Payment Processing]
        NotificationService[Notification Service]
    end
    
    %% Data Flows - Guest
    Guest -->|Registration Data| UserManagement
    Guest -->|Login Credentials| UserManagement
    Guest -->|Property Search Criteria| PropertyManagement
    Guest -->|Booking Requests| BookingManagement
    Guest -->|Payment Information| PaymentProcessing
    Guest -->|Review Data| PropertyManagement
    
    PropertyManagement -->|Property Listings| Guest
    BookingManagement -->|Booking Confirmations| Guest
    PaymentProcessing -->|Payment Receipts| Guest
    NotificationService -->|Notifications| Guest
    
    %% Data Flows - Host
    Host -->|Property Listing Data| PropertyManagement
    Host -->|Booking Approvals/Rejections| BookingManagement
    Host -->|Availability Updates| PropertyManagement
    Host -->|Payout Preferences| PaymentProcessing
    
    BookingManagement -->|Booking Requests| Host
    PaymentProcessing -->|Payout Notifications| Host
    PropertyManagement -->|Performance Analytics| Host
    
    %% Data Flows - Admin
    Admin -->|User Management Commands| UserManagement
    Admin -->|Property Moderation Decisions| PropertyManagement
    Admin -->|System Configuration| ARBAR_SYSTEM
    
    ARBAR_SYSTEM -->|System Reports| Admin
    ARBAR_SYSTEM -->|Alert Notifications| Admin
    
    %% External Service Flows
    PaymentProcessing -->|Payment Authorization| PaymentGateway
    PaymentGateway -->|Payment Confirmation| PaymentProcessing
    PaymentProcessing -->|Payout Requests| PaymentGateway
    
    NotificationService -->|Email Sending Requests| EmailService
    EmailService -->|Delivery Status| NotificationService
```


## Level 1 DFD - Detailed Processes
```
flowchart TD
    %% External Entities
    Guest[Guest User]
    Host[Property Host]
    Admin[System Admin]
    PaymentGateway[Payment Gateway]
    EmailService[Email Service]
    
    %% Process 1: User Management
    subgraph P1 [User Management]
        P1_1[P1.1: Register User]
        P1_2[P1.2: Authenticate User]
        P1_3[P1.3: Manage Profile]
        P1_4[P1.4: Email Verification]
    end
    
    %% Process 2: Property Management
    subgraph P2 [Property Management]
        P2_1[P2.1: Create/Update Property]
        P2_2[P2.2: Manage Images]
        P2_3[P2.3: Search Properties]
        P2_4[P2.4: Moderate Properties]
    end
    
    %% Process 3: Booking Management
    subgraph P3 [Booking Management]
        P3_1[P3.1: Create Booking]
        P3_2[P3.2: Check Availability]
        P3_3[P3.3: Manage Status]
        P3_4[P3.4: Handle Cancellations]
    end
    
    %% Process 4: Payment Processing
    subgraph P4 [Payment Processing]
        P4_1[P4.1: Process Payments]
        P4_2[P4.2: Handle Refunds]
        P4_3[P4.3: Manage Payouts]
        P4_4[P4.4: Calculate Fees]
    end
    
    %% Process 5: Notification Service
    subgraph P5 [Notification Service]
        P5_1[P5.1: Send Emails]
        P5_2[P5.2: In-App Notifications]
        P5_3[P5.3: Manage Templates]
    end
    
    %% Data Stores
    D1[\[D1\]<br/>Users]
    D2[\[D2\]<br/>Verification Tokens]
    D3[\[D3\]<br/>User Sessions]
    D4[\[D4\]<br/>Properties]
    D5[\[D5\]<br/>Property Images]
    D6[\[D6\]<br/>Amenities]
    D7[\[D7\]<br/>Locations]
    D8[\[D8\]<br/>Bookings]
    D9[\[D9\]<br/>Booking Calendar]
    D10[\[D10\]<br/>Cancellation Policies]
    D11[\[D11\]<br/>Payments]
    D12[\[D12\]<br/>Payouts]
    D13[\[D13\]<br/>Transaction Records]
    D14[\[D14\]<br/>Notifications]
    D15[\[D15\]<br/>Email Templates]
    D16[\[D16\]<br/>Notification Logs]
    
    %% User Management Flows
    Guest -->|Registration Data| P1_1
    Host -->|Registration Data| P1_1
    Guest -->|Login Credentials| P1_2
    P1_1 -->|User Data| D1
    P1_1 -->|Verification Token| D2
    P1_2 -->|Session Data| D3
    P1_3 -->|Profile Updates| D1
    P1_4 -->|Verification Status| D1
    P1_4 -->|Email Request| P5_1
    
    %% Property Management Flows
    Host -->|Property Data| P2_1
    Guest -->|Search Criteria| P2_3
    Admin -->|Moderation Decisions| P2_4
    P2_1 -->|Property Info| D4
    P2_1 -->|Image Data| D5
    P2_2 -->|Image Updates| D5
    P2_3 -->|Search Query| D4
    P2_3 -->|Location Query| D7
    P2_4 -->|Approval Status| D4
    
    %% Booking Management Flows
    Guest -->|Booking Request| P3_1
    Host -->|Booking Decision| P3_3
    P3_1 -->|Availability Check| P3_2
    P3_2 -->|Calendar Query| D9
    P3_1 -->|Booking Data| D8
    P3_3 -->|Status Updates| D8
    P3_4 -->|Cancellation Data| D8
    P3_4 -->|Policy Check| D10
    P3_1 -->|Payment Request| P4_1
    P3_3 -->|Confirmation| P5_1
    
    %% Payment Processing Flows
    P4_1 -->|Payment Data| D11
    P4_2 -->|Refund Data| D11
    P4_3 -->|Payout Data| D12
    P4_4 -->|Fee Calculations| D13
    P4_1 -->|Gateway Request| PaymentGateway
    PaymentGateway -->|Confirmation| P4_1
    P4_3 -->|Payout Request| PaymentGateway
    
    %% Notification Service Flows
    P5_1 -->|Email Data| EmailService
    P5_2 -->|Notification Data| D14
    P5_3 -->|Template Data| D15
    P5_1 -->|Send Log| D16
    EmailService -->|Delivery Status| D16
    
    %% Cross-process flows
    P1_2 -->|Auth Status| P3_1
    P2_3 -->|Search Results| P3_1
    P4_1 -->|Payment Status| P3_3
    P3_3 -->|Booking Events| P5_1
    P4_1 -->|Payment Events| P5_1
```

## Detailed Booking Flow Sequence
```
sequenceDiagram
    participant G as Guest
    participant P2 as Property Search
    participant P3 as Booking Management
    participant P4 as Payment Processing
    participant P5 as Notification Service
    participant H as Host
    participant PG as Payment Gateway
    
    G->>P2: Search Properties (dates, location)
    P2->>P2: Check availability & pricing
    P2-->>G: Return property results
    
    G->>P3: Create Booking Request
    P3->>P3: Validate dates & availability
    P3->>P4: Calculate total amount
    P4-->>P3: Return calculated total
    P3-->>G: Return booking summary
    
    G->>P4: Submit payment method
    P4->>PG: Create payment intent
    PG-->>P4: Return payment authorization
    P4->>P3: Confirm payment ready
    P3->>H: Notify of booking request
    P3-->>G: Booking status: Pending Approval
    
    alt Host Approves
        H->>P3: Approve booking
        P3->>P4: Capture payment
        P4->>PG: Process payment capture
        PG-->>P4: Payment confirmed
        P4->>P3: Update payment status
        P3->>P5: Send confirmation emails
        P5->>G: Booking confirmed notification
        P5->>H: Booking confirmed notification
        P3-->>G: Booking status: Confirmed
    else Host Rejects
        H->>P3: Reject booking
        P3->>P4: Release authorization
        P4->>PG: Cancel payment intent
        P3->>P5: Send rejection notices
        P5->>G: Booking rejected notification
        P3-->>G: Booking status: Rejected
    end
```

## Payment Processing Flow
```
flowchart TD
    Start[Booking Created] --> CheckType{Booking Type}
    
    CheckType -->|Instant Book| InstantBook[Instant Book Process]
    CheckType -->|Host Approval| HostApproval[Host Approval Process]
    
    subgraph InstantBook [Instant Book Payment Flow]
        IB1[Authorize Payment] --> IB2[Capture Payment Immediately]
        IB2 --> IB3[Confirm Booking Automatically]
        IB3 --> IB4[Send Instant Confirmation]
    end
    
    subgraph HostApproval [Host Approval Payment Flow]
        HA1[Authorize Payment Hold] --> HA2[Wait for Host Decision]
        HA2 --> Decision{Host Decision}
        Decision -->|Approve| HA3[Capture Payment]
        Decision -->|Reject| HA4[Release Authorization]
        HA3 --> HA5[Confirm Booking]
        HA4 --> HA6[Notify Guest of Rejection]
        HA5 --> HA7[Send Confirmation Notifications]
    end
    
    InstantBook --> Payout[Schedule Host Payout]
    HostApproval --> Payout
    
    Payout --> PostStay[Post-Stay Process]
    
    subgraph PostStay [After Stay Completion]
        PS1[24-hour waiting period] --> PS2[Process Host Payout]
        PS2 --> PS3[Transfer to Host Account]
        PS3 --> PS4[Send Payout Notification]
    end
    
    PostStay --> End[Process Complete]
```

## Notification Flow Diagram
```
flowchart LR
    subgraph EventSources [Event Sources]
        A[User Registration]
        B[Booking Created]
        C[Payment Processed]
        D[Booking Status Change]
        E[Host Payout]
    end
    
    subgraph NotificationEngine [Notification Engine]
        F[Event Processor]
        G[Template Renderer]
        H[Channel Router]
    end
    
    subgraph DeliveryChannels [Delivery Channels]
        I[Email Service]
        J[In-App Notifications]
        K[Push Notifications]
    end
    
    subgraph DataStores [Data Stores]
        L[Notification Templates]
        M[Notification Logs]
        N[User Preferences]
    end
    
    EventSources --> F
    F --> G
    G --> L
    L --> G
    G --> H
    H --> N
    N --> H
    H --> I
    H --> J
    H --> K
    I --> M
    J --> M
    K --> M

```