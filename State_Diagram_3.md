# State Diagram 3: Bid Lifecycle

This state diagram defines the lifecycle of a worker's proposal (Bid) from submission through evaluation, customer decision, and final status synchronization with the parent Job.

```mermaid
stateDiagram-v2
    direction TB

    %% --- State Definitions ---
    state "SUBMITTED_PENDING" as S1
    state "SHORTLISTED" as S2
    state "EVALUATED_REJECTED" as S3
    state "ACCEPTED_CONTRACTED" as S4
    state "WITHDRAWN" as S5
    state "VOIDED" as S6

    %% --- Main Process Flow ---
    [*] --> S1 : onProposalSubmit
    note right of S1: Pricing Advisor AI validates\nbid against market range

    S1 --> S2 : onCustomerInterest [Add to Shortlist]
    
    state S2 {
        direction LR
        [*] --> Reviewing
        Reviewing --> Interviewing : onMessageInitiated
    }

    S2 --> S4 : onCustomerSelection [Hire Worker]
    note left of S4: Triggers Job state change\nto WORKER_ASSIGNED

    S4 --> [*] : onJobCompletion

    %% --- Alternative & Terminal Paths ---
    S1 --> S3 : onCustomerRejection
    S2 --> S3 : onComparisonFailure
    
    S1 --> S5 : onWorkerRetraction [Before Acceptance]
    S2 --> S5 : onAvailabilityChange
    
    S1 --> S6 : onJobCancellation [External Event]
    S2 --> S6 : onJobExpiry
    S4 --> S6 : onContractTermination [Dispute/Abandonment]

    S3 --> [*] : logForAnalytics
    S5 --> [*] : notifyCustomer
    S6 --> [*] : archiveProposal

    %% --- Documentation Notes ---
    note left of S1
        Initial analysis by:
        - Pricing Advisor AI
        - Historical Rating Check
    end note

    note right of S4
        Legally binding 
        agreement within
        the platform scope.
    end note
```

### Key States Description
*   **SUBMITTED_PENDING:** The initial state where the bid is visible to the customer. The Pricing Advisor AI may tag the bid as "Fair," "High," or "Competitive" during this phase.
*   **SHORTLISTED:** A composite state where the customer has shown specific interest, potentially leading to further communication (Interviewing) via the in-app messaging system.
*   **ACCEPTED_CONTRACTED:** The winning bid. This transition is synchronous with the Job moving to the `WORKER_ASSIGNED` state.
*   **EVALUATED_REJECTED:** The customer has explicitly declined the proposal.
*   **WITHDRAWN:** The worker has removed their bid before it was accepted, often due to a change in availability.
*   **VOIDED:** An "automatic" terminal state triggered when the parent Job is cancelled, expires, or is closed without this specific bid being selected.
