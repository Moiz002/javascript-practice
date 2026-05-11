# Serviqo System Design: Sequence Diagram - Bidding & Job Assignment

This document details the process of bid submission, proposal review, and the final job assignment flow between customers and workers.

```mermaid
sequenceDiagram
    autonumber
    
    actor Worker
    actor Customer
    participant UI as Frontend (Web/Mobile)
    participant API as Bid/Job Controller
    participant Svc as Backend Service
    participant DB as MongoDB
    participant Notif as Notification Service

    %% --- Phase 1: Bid Submission ---
    rect rgb(240, 248, 255)
        Note over Worker, DB: Phase 1: Bid Submission
        
        Worker->>UI: Submit Bid (Price, Duration, Proposal)
        activate UI
        UI->>API: POST /api/bids/submit
        activate API
        
        API->>Svc: Validate Bid Data & Worker Status
        activate Svc
        Svc->>DB: Check for Existing Bid
        activate DB
        DB-->>Svc: No Existing Bid
        deactivate DB
        
        Svc->>DB: Save Bid (Status: 'pending')
        activate DB
        DB-->>Svc: Success (BidID)
        deactivate DB
        
        Svc->>Notif: Notify Customer of New Bid
        activate Notif
        Notif-->>Customer: Push: "New Bid Received"
        deactivate Notif
        
        Svc-->>API: Bid Processed
        deactivate Svc
        
        API-->>UI: 201 Created
        deactivate API
        UI-->>Worker: Show "Bid Submitted"
        deactivate UI
    end

    %% --- Phase 2: Proposal Review ---
    rect rgb(245, 245, 245)
        Note over Customer, DB: Phase 2: Reviewing Proposals
        
        Customer->>UI: View Job Bids
        activate UI
        UI->>API: GET /api/jobs/:id/bids
        activate API
        API->>DB: Fetch Bids with Worker Profiles
        activate DB
        DB-->>API: Bids Array
        deactivate DB
        API-->>UI: Display Bids List
        deactivate API
        UI-->>Customer: Show Bids + Worker Ratings
        deactivate UI
    end

    %% --- Phase 3: Job Assignment ---
    rect rgb(240, 255, 240)
        Note over Customer, DB: Phase 3: Selection & Assignment
        
        Customer->>UI: Select Worker & Accept Bid
        activate UI
        UI->>API: POST /api/jobs/:id/assign
        activate API
        
        API->>Svc: Execute Assignment Transaction
        activate Svc
        
        Svc->>DB: updateJob(status: 'assigned', worker: workerID)
        activate DB
        DB-->>Svc: Success
        deactivate DB
        
        Svc->>DB: updateBid(status: 'accepted')
        activate DB
        DB-->>Svc: Success
        deactivate DB
        
        Svc->>DB: rejectOtherBids(jobID)
        activate DB
        DB-->>Svc: Success
        deactivate DB
        
        Svc->>DB: createConversation(customer, worker)
        activate DB
        DB-->>Svc: ConversationID
        deactivate DB
        
        Svc-->>API: Assignment Complete
        deactivate Svc
        
        API-->>UI: 200 OK
        deactivate API
        UI-->>Customer: Redirect to Messaging/Active Job
        deactivate UI
    end

    %% --- Phase 4: Confirmation Notifications ---
    rect rgb(255, 250, 240)
        Note over Worker, Notif: Phase 4: Final Alerts
        
        API->>Notif: Send Acceptance Alert to Selected Worker
        activate Notif
        Notif-->>Worker: Push: "Job Assigned!"
        deactivate Notif
        
        API->>Notif: Send Rejection Alerts to Others
        activate Notif
        Notif-->>Worker: Push: "Job Closed"
        deactivate Notif
    end
```
