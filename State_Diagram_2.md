# State Diagram 2: Job Lifecycle

This state diagram illustrates the operational flow of a service request (Job) from its creation by a customer to its final resolution, including standard and exceptional termination paths.

```mermaid
stateDiagram-v2
    direction TB

    %% --- State Definitions ---
    state "POSTED_OPEN" as S1
    state "WORKER_ASSIGNED" as S2
    state "EXECUTION_IN_PROGRESS" as S3
    state "WORK_REVIEW_PENDING" as S4
    state "COMPLETED_CLOSED" as S5
    state "CANCELLED" as S6
    state "DISPUTE_RESOLUTION" as S7

    %% --- Main Success Path ---
    [*] --> S1 : onJobSubmission
    note right of S1: Job is visible in worker feed\nAccepting bids

    S1 --> S2 : onBidAcceptance [Customer selects Worker]
    
    S2 --> S3 : onJobStart [Worker arrival/commence]
    
    S3 --> S4 : onWorkSubmission [Worker marks as 'Done']
    note left of S4: Customer reviews quality\nVerifies completion

    S4 --> S5 : onCustomerApproval [Finalizing payment]
    
    S5 --> [*] : archiveRecord

    %% --- Branching & Exceptional Paths ---
    S1 --> S6 : onUserCancellation [No bids accepted]
    S2 --> S6 : onJobAbandonment [Worker/Customer withdrawal]
    
    S3 --> S7 : onConflictDetection [Dispute raised]
    S4 --> S7 : onQualityObjection [Customer rejects work]
    
    state S7 {
        direction LR
        [*] --> Under_Investigation
        Under_Investigation --> Resolution_Proposed : onAdminIntervention
        Resolution_Proposed --> [*] : onAgreement
    }

    S7 --> S5 : onResolved_Success
    S7 --> S6 : onResolved_Failure [Refund triggered]

    S6 --> [*] : logCancellationReason

    %% --- Documentation Notes ---
    note left of S3
        Real-time tracking active.
        Messaging enabled between
        Customer and Assigned Worker.
    end note

    note right of S7
        Guardian AI monitors 
        for suspicious activity
        during this phase.
    end note
```

### Key States Description
*   **POSTED_OPEN:** The initial state where the job is live and the Recruiter AI Agent begins inviting suitable workers.
*   **WORKER_ASSIGNED:** A contract is formed. The job is no longer visible to other workers.
*   **EXECUTION_IN_PROGRESS:** The worker is physically or digitally performing the service. Real-time status updates are expected.
*   **WORK_REVIEW_PENDING:** The worker has claimed completion. The system waits for the customer to verify the result before releasing funds.
*   **COMPLETED_CLOSED:** The "Happy Path" termination. Feedback/Reviews are exchanged, and payment is finalized.
*   **DISPUTE_RESOLUTION:** A non-linear state where an Administrator or system logic must mediate a conflict between the parties.
*   **CANCELLED:** The job is terminated without completion. Reasons are logged for platform analytics.
