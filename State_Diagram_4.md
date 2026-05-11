# State Diagram 4: Worker Verification Workflow

This state diagram outlines the multi-stage trust and safety process that service providers must undergo to gain "Verified" status on the Serviqo platform.

```mermaid
stateDiagram-v2
    direction TB

    %% --- State Definitions ---
    state "PROFILE_INCOMPLETE" as S1
    state "PENDING_ADMIN_REVIEW" as S2
    state "UNDER_MANUAL_INVESTIGATION" as S3
    state "VERIFIED_ACTIVE" as S4
    state "REVISION_REQUIRED" as S5
    state "PERMANENTLY_REJECTED" as S6

    %% --- Main Verification Path ---
    [*] --> S1 : "onWorkerRegistration"
    
    S1 --> S2 : "submitVerificationDocs [CNIC_Front and CNIC_Back]"
    note right of S1
        Worker provides identity documents
        and selects primary category
    end note

    S2 --> S3 : "onAdminFlag [SuspiciousDataDetected]"
    S3 --> S2 : "onInitialClearance [FalsePositive]"

    S2 --> S4 : "onAdminApproval [IdentityConfirmed]"
    note left of S4
        Unlocks:
        - Bid Submission
        - Premium Badges
        - Public Portfolio
    end note

    %% --- Rejection & Revision Loops ---
    S2 --> S5 : "onAdminRejection [BlurryImages / Mismatch]"
    S5 --> S2 : "resubmitDocs [FixedData]"

    S3 --> S6 : "onFraudConfirmation [FakeIdentity / Blacklisted]"
    S2 --> S6 : "onPolicyViolation"

    S4 --> S3 : "onSecurityAudit [PeriodicCheck]"
    
    S6 --> [*] : "logForCompliance"

    %% --- Documentation Notes ---
    note left of S2
        SLA: 24-48 Hours
        Admin dashboard tracks
        document quality and
        expiry dates.
    end note

    note right of S5
        Feedback provided to 
        worker via notifications
        with specific reasons.
    end note
```

### Key States Description
*   **PROFILE_INCOMPLETE:** The worker has an account but has not yet submitted the required government-issued identity documents (CNIC).
*   **PENDING_ADMIN_REVIEW:** Documents have been uploaded. The worker appears in the Admin Dashboard verification queue.
*   **UNDER_MANUAL_INVESTIGATION:** A high-scrutiny state triggered if an administrator identifies potential risks or anomalies requiring deeper background checks or secondary verification.
*   **VERIFIED_ACTIVE:** The "Gold Standard" state. The worker is fully authorized to participate in the marketplace and use all agentic AI features.
*   **REVISION_REQUIRED:** Documents were rejected for fixable reasons (e.g., poor lighting, expired ID). The worker is prompted to update their submission.
*   **PERMANENTLY_REJECTED:** The identity was flagged as fraudulent or in violation of platform safety policies. This state is terminal to prevent platform abuse.
