# Serviqo System Design: Sequence Diagram - Job Posting & AI Recommendation

This diagram illustrates the lifecycle of a service request, from initial creation with AI pricing guidance to automated worker recommendation and the bidding process.

```mermaid
sequenceDiagram
    autonumber
    
    actor Customer
    participant UI as Frontend (Web/Mobile)
    participant API as Job Controller
    participant PriceAI as Pricing Advisor Agent
    participant RecruitAI as Recruiter AI Agent
    participant DB as MongoDB
    participant WorkerUI as Worker App
    participant Notif as Notification Service

    %% --- Phase 1: Job Creation with Pricing Guidance ---
    rect rgb(240, 248, 255)
        Note over Customer, PriceAI: Phase 1: Job Initiation & Pricing Guidance
        
        Customer->>UI: Select Service Category (e.g., Plumbing)
        activate UI
        UI->>API: GET /api/jobs/suggested-price?category=Plumbing
        activate API
        
        API->>PriceAI: Request Price Prediction
        activate PriceAI
        PriceAI->>DB: Query Historical Bid Data
        activate DB
        DB-->>PriceAI: Aggregate Data
        deactivate DB
        PriceAI-->>API: Predicted Price Range (e.g., $50 - $80)
        deactivate PriceAI
        
        API-->>UI: Return Budget Suggestions
        deactivate API
        UI-->>Customer: Display "Smart Budget" Recommendations
        deactivate UI
    end

    %% --- Phase 2: Final Submission & Storage ---
    rect rgb(245, 245, 245)
        Note over Customer, DB: Phase 2: Job Submission
        
        Customer->>UI: Input Job Details (Desc, Location, Budget, Images)
        activate UI
        UI->>API: POST /api/jobs/create
        activate API
        
        API->>DB: Save Job (Status: 'open')
        activate DB
        DB-->>API: JobID Created
        deactivate DB
        
        API-->>UI: 201 Created (Success Message)
        deactivate API
        UI-->>Customer: Redirect to "Track Job" Page
        deactivate UI
    end

    %% --- Phase 3: AI Worker Recommendation (Recruiter Agent) ---
    rect rgb(240, 255, 240)
        Note over API, RecruitAI: Phase 3: Recruiter AI Analysis
        
        API->>RecruitAI: Trigger Recommendation Engine (JobID)
        activate RecruitAI
        
        RecruitAI->>DB: Query Workers (Skills, Location, Rating)
        activate DB
        DB-->>RecruitAI: List of Potential Matches
        deactivate DB
        
        RecruitAI->>RecruitAI: Score & Rank Workers (ML Model)
        
        RecruitAI->>DB: Store Recommendations for Job
        activate DB
        DB-->>RecruitAI: Success
        deactivate DB
        
        RecruitAI-->>API: Top Matches Identified
        deactivate RecruitAI
    end

    %% --- Phase 4: Notification & Bidding ---
    rect rgb(255, 250, 240)
        Note over API, WorkerUI: Phase 4: Job Distribution & Bidding
        
        API->>Notif: Send Push Notifications to Recommended Workers
        activate Notif
        Notif->>WorkerUI: New Job Opportunity!
        deactivate Notif
        
        WorkerUI->>Customer: Browse Job Feed
        
        Customer->>WorkerUI: Open Job Details
        activate WorkerUI
        WorkerUI->>API: GET /api/jobs/:id
        activate API
        API-->>WorkerUI: Full Job Context
        deactivate API
        
        Customer->>WorkerUI: Submit Bid (Price, ETA, Proposal)
        WorkerUI->>API: POST /api/bids/submit
        activate API
        API->>DB: Save Bid & Notify Customer
        activate DB
        DB-->>API: Success
        deactivate DB
        API-->>WorkerUI: Bid Received
        deactivate API
        deactivate WorkerUI
    end

    %% --- Phase 5: Customer Comparison ---
    rect rgb(245, 245, 255)
        Note over Customer, DB: Phase 5: Proposal Review
        
        Customer->>UI: View Bids for Job
        activate UI
        UI->>API: GET /api/jobs/:id/bids
        activate API
        API->>DB: Fetch Bids + Worker AI Scores
        activate DB
        DB-->>API: Bids Data
        deactivate DB
        API-->>UI: Display Bids with "Top Match" Badges
        deactivate API
        UI-->>Customer: Show Comparison View
        deactivate UI
    end
```
