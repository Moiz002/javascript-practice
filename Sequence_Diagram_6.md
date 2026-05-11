# Serviqo System Design: Sequence Diagram - Communication & Dispute Management

This document details the real-time interaction between users via messaging and the formal process for raising and resolving job disputes.

```mermaid
sequenceDiagram
    autonumber
    
    actor Customer
    actor Worker
    actor Admin
    participant UI as Frontend (Web/Mobile)
    participant Socket as Socket.io Server
    participant API as Message/Dispute Controller
    participant DB as MongoDB
    participant Notif as Notification Service

    %% --- Phase 1: Real-time In-App Messaging ---
    rect rgb(240, 248, 255)
        Note over Customer, Worker: Phase 1: Real-time Communication (Module 5)
        
        Customer->>UI: Open Message Interface
        UI->>Socket: Join Room (conversationID)
        activate Socket
        Worker->>UI: Open Message Interface
        UI->>Socket: Join Room (conversationID)
        
        Customer->>UI: Type & Send Message
        UI->>Socket: emit("send_message", payload)
        
        par Real-time Relay & Persistence
            Socket->>UI: broadcast("receive_message", payload)
            UI-->>Worker: Display Message Instantly
            
            Socket->>API: Persist Message
            activate API
            API->>DB: saveMessage({ sender, text, timestamp })
            deactivate API
        end
        deactivate Socket
    end

    %% --- Phase 2: Raising a Dispute ---
    rect rgb(255, 240, 245)
        Note over Customer, DB: Phase 2: Dispute Initiation
        
        Customer->>UI: Click "Raise Dispute" on Job
        activate UI
        UI-->>Customer: Show Dispute Form (Reason, Evidence)
        Customer->>UI: Submit Dispute
        UI->>API: POST /api/disputes/create
        activate API
        
        API->>DB: Create Dispute Record (Status: 'open')
        activate DB
        DB-->>API: DisputeID
        deactivate DB
        
        API->>Notif: Alert Admin of New Complaint
        activate Notif
        Notif-->>Admin: "A new dispute requires review."
        deactivate Notif
        
        API-->>UI: 201 Created
        deactivate API
        UI-->>Customer: Show "Dispute Filed - Under Review"
        deactivate UI
    end

    %% --- Phase 3: Admin Resolution Flow ---
    rect rgb(240, 255, 240)
        Note over Admin, DB: Phase 3: Dispute Resolution (Module 8)
        
        Admin->>UI: Open Dispute Details
        activate UI
        UI->>API: GET /api/admin/disputes/:id
        activate API
        API->>DB: Fetch Job History & Messages
        activate DB
        DB-->>API: Full Context Data
        deactivate DB
        API-->>UI: Display Case Evidence
        deactivate API
        
        Admin->>UI: Review Evidence & Select Resolution (Refund/Pay Worker)
        UI->>API: PATCH /api/admin/disputes/:id/resolve
        activate API
        
        API->>DB: updateDispute(status: 'resolved', decision: '...')
        activate DB
        DB-->>API: Success
        deactivate DB
        
        par Multi-party Notification
            API->>Notif: Notify Customer of Decision
            API->>Notif: Notify Worker of Decision
        end
        
        API-->>UI: 200 OK (Case Closed)
        deactivate API
        UI-->>Admin: Show "Dispute Resolved"
        deactivate UI
    end
```
