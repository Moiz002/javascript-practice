# Serviqo System Design: Sequence Diagram - Job Tracking, Review & Payment

This document outlines the final phase of the service lifecycle, including real-time status updates, simulated payment handling, and the feedback mechanism.

```mermaid
sequenceDiagram
    autonumber
    
    actor Worker
    actor Customer
    participant UI as Frontend (Web/Mobile)
    participant API as Job/Payment Controller
    participant Svc as Backend Service
    participant DB as MongoDB
    participant Notif as Notification Service

    %% --- Phase 1: Job Execution & Real-Time Tracking ---
    rect rgb(240, 248, 255)
        Note over Worker, Customer: Phase 1: Job Progress Tracking
        
        Worker->>UI: Start Job (Mark "In Progress")
        activate UI
        UI->>API: PATCH /api/jobs/:id/status (in-progress)
        activate API
        API->>Svc: Update Job State
        activate Svc
        Svc->>DB: updateOne({ status: 'in-progress' })
        activate DB
        DB-->>Svc: Success
        deactivate DB
        Svc->>Notif: Notify Customer: "Worker has started!"
        activate Notif
        Notif-->>Customer: Push Alert
        deactivate Notif
        Svc-->>API: Status Updated
        deactivate Svc
        API-->>UI: 200 OK
        deactivate API
        UI-->>Worker: Show "Job Active"
        deactivate UI
    end

    %% --- Phase 2: Completion & Verification ---
    rect rgb(245, 245, 245)
        Note over Worker, Customer: Phase 2: Completion Flow
        
        Worker->>UI: Mark as "Completed"
        activate UI
        UI->>API: PATCH /api/jobs/:id/status (completed)
        activate API
        API-->>UI: 200 OK (Pending Verification)
        deactivate API
        UI-->>Worker: Show "Awaiting Customer Confirmation"
        deactivate UI

        Customer->>UI: Review Work & Confirm Completion
        activate UI
        UI->>API: POST /api/jobs/:id/verify
        activate API
        API->>Svc: Set Final Completion State
        Svc->>DB: updateOne({ status: 'verified' })
        API-->>UI: 200 OK (Proceed to Payment)
        deactivate API
        UI-->>Customer: Load Payment Interface
        deactivate UI
    end

    %% --- Phase 3: Simulated Payment & Commission ---
    rect rgb(240, 255, 240)
        Note over Customer, DB: Phase 3: Simulated Payment Processing
        
        Customer->>UI: Authorize Payment (Digital Wallet/Card Sim)
        activate UI
        UI->>API: POST /api/payments/process
        activate API
        
        API->>Svc: Calculate Commission & Net Pay
        activate Svc
        Svc->>Svc: Amount - 10% Platform Fee
        
        Svc->>DB: Create Transaction Record (JobID, Total, Fee, Net)
        activate DB
        DB-->>Svc: TransactionID
        deactivate DB
        
        Svc->>DB: Update Worker Balance & Job Status (paid)
        activate DB
        DB-->>Svc: Success
        deactivate DB
        
        Svc->>Notif: Send Payment Confirmation to Worker
        activate Notif
        Notif-->>Worker: Push: "Payment Received!"
        deactivate Notif
        
        Svc-->>API: Transaction Success
        deactivate Svc
        
        API-->>UI: 200 OK (Generate Receipt)
        deactivate API
        UI-->>Customer: Show Success & "Rate your Worker"
        deactivate UI
    end

    %% --- Phase 4: Review & Rating ---
    rect rgb(255, 250, 240)
        Note over Customer, Worker: Phase 4: Feedback Loop
        
        Customer->>UI: Submit Review (Rating: 1-5, Comment)
        activate UI
        UI->>API: POST /api/reviews/submit
        activate API
        
        API->>Svc: Process Feedback
        activate Svc
        Svc->>DB: Save Review Object
        activate DB
        DB-->>Svc: Success
        deactivate DB
        
        Svc->>DB: Recalculate Worker Average Rating
        activate DB
        DB-->>Svc: New Average Updated
        deactivate DB
        
        Svc-->>API: Review Processed
        deactivate Svc
        
        API-->>UI: 200 OK
        deactivate API
        UI-->>Customer: Return to Dashboard
        deactivate UI
    end
```
