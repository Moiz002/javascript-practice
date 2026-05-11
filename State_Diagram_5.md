# State Diagram 5: Dispute Resolution Lifecycle

This state diagram illustrates the procedural lifecycle of a formal dispute within the Serviqo platform, detailing how conflicts between customers and workers are managed and resolved by administrators.

```mermaid
stateDiagram-v2
    direction TB

    %% --- State Definitions ---
    state "DISPUTE_RAISED" as S1
    state "TRIAGE_IN_PROGRESS" as S2
    state "ACTIVE_INVESTIGATION" as S3
    state "MEDIATION_PROPOSED" as S4
    state "RESOLVED_SETTLED" as S5
    state "DISMISSED_UNFOUNDED" as S6

    %% --- Main Resolution Flow ---
    [*] --> S1 : "onDisputeSubmission [Triggered by User]"
    note right of S1
        Job state moves to DISPUTE_RESOLUTION
        Payments are frozen in escrow
    end note

    S1 --> S2 : "assignAdminHandler"
    
    S2 --> S3 : "onEvidenceReview [Evidence_Sufficient]"
    note left of S3
        Admin reviews:
        - Chat logs
        - Job images
        - Bid terms
    end note

    S3 --> S4 : "proposeResolution [Refund/Partial_Payment]"
    
    S4 --> S5 : "onMutualAcceptance [Agreement_Reached]"
    
    S5 --> [*] : "releaseFunds / updateRatings"

    %% --- Alternative & Terminal Paths ---
    S2 --> S6 : "onInvalidClaim [Lack_of_Evidence]"
    S3 --> S6 : "onPolicyViolation [Reporting_Party_at_Fault]"
    
    S4 --> S3 : "onProposalRejection [Negotiation_Continues]"
    
    S6 --> [*] : "resumeJob_or_Archive"

    %% --- Documentation Notes ---
    note left of S1
        Monitored via Admin Dashboard
        to detect recurring
        conflict patterns.
    end note

    note right of S4
        Resolution types:
        - Full Refund
        - Partial Payout
        - Redo Work
    end note
```

### Key States Description
*   **DISPUTE_RAISED:** The formal entry point. Triggered when either party indicates a failure in service delivery or contract terms.
*   **TRIAGE_IN_PROGRESS:** An initial administrative phase where the dispute is assigned to a human moderator and checked for basic validity.
*   **ACTIVE_INVESTIGATION:** The core analytical phase. The Administrator examines all platform-recorded evidence (messages, timestamps, image uploads) to determine the root cause.
*   **MEDIATION_PROPOSED:** A state where the Admin offers a compromise or a specific ruling (e.g., 50% payment) to both parties.
*   **RESOLVED_SETTLED:** The successful terminal state. Both parties have accepted the resolution, and final accounting (simulated payments) is executed.
*   **DISMISSED_UNFOUNDED:** The dispute is closed without action, typically because the claim was deemed invalid or the reporter violated platform terms.
