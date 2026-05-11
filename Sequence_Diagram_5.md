# Serviqo System Design: Sequence Diagram - Admin Operations & System Analytics

This document outlines the administrative lifecycle for worker verification and the generation of demand-based market analytics.

```mermaid
sequenceDiagram
    autonumber
    
    actor Admin
    participant UI as Admin Dashboard (Web)
    participant API as Admin Controller
    participant Svc as Backend Service
    participant DB as MongoDB
    participant Notif as Notification Service

    %% --- Phase 1: Worker Verification & Approval ---
    rect rgb(240, 248, 255)
        Note over Admin, DB: Phase 1: Worker Verification Lifecycle
        
        Admin->>UI: Open "Pending Verification" List
        activate UI
        UI->>API: GET /api/admin/workers/pending
        activate API
        API->>DB: Query Workers (status: 'pending')
        activate DB
        DB-->>API: List of Workers + Documents (CNIC/Portfolio)
        deactivate DB
        API-->>UI: Display Worker Verification Cards
        deactivate API
        
        Admin->>UI: Review Documents & Click "Approve"
        UI->>API: PATCH /api/admin/workers/:id/verify
        activate API
        
        API->>Svc: Process Approval
        activate Svc
        Svc->>DB: updateOne({ status: 'active', verifiedAt: now })
        activate DB
        DB-->>Svc: Success
        deactivate DB
        
        Svc->>Notif: Send Welcome Notification to Worker
        activate Notif
        Notif-->>Svc: Notification Queued
        deactivate Notif
        
        Svc-->>API: Verification Complete
        deactivate Svc
        
        API-->>UI: 200 OK (Worker Verified)
        deactivate API
        UI-->>Admin: Show "Worker Successfully Approved"
        deactivate UI
    end

    %% --- Phase 2: Demand Analysis & Marketplace Analytics ---
    rect rgb(245, 245, 245)
        Note over Admin, DB: Phase 2: Demand Intelligence (BO-6)
        
        Admin->>UI: Request "Demand Trends" Report
        activate UI
        UI->>API: GET /api/admin/analytics/demand
        activate API
        
        API->>Svc: Aggregate Marketplace Data
        activate Svc
        Svc->>DB: aggregate({ jobs by Category & Location })
        activate DB
        DB-->>Svc: Dataset (e.g., Plumbing: +20% growth)
        deactivate DB
        
        Svc->>Svc: Calculate High-Demand Zones & Pricing Trends
        
        Svc-->>API: Processed Analytics Report
        deactivate Svc
        
        API-->>UI: 200 OK (JSON Chart Data)
        deactivate API
        UI-->>Admin: Render Charts (Heatmaps, Bar Graphs)
        deactivate UI
    end

    %% --- Phase 3: Platform Monitoring ---
    rect rgb(240, 255, 240)
        Note over Admin, DB: Phase 3: Active Job Monitoring
        
        Admin->>UI: View "All Active Jobs"
        activate UI
        UI->>API: GET /api/admin/jobs/active
        activate API
        API->>DB: Fetch Ongoing Transactions
        activate DB
        DB-->>API: Real-time Job List
        deactivate DB
        API-->>UI: Update Live Dashboard
        deactivate API
        deactivate UI
    end
```
