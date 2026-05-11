# State Diagram 1: User Account Lifecycle

This state diagram outlines the lifecycle of a user account within the Serviqo platform, from initial registration through verification, onboarding, active usage, and eventual deactivation.

```mermaid
stateDiagram-v2
    direction TB

    %% --- State Definitions ---
    state "PROVISIONED" as S1
    state "CREDENTIAL_VERIFICATION" as S2
    state "IDENTITY_ONBOARDING" as S3
    state "ACTIVE_OPERATIONAL" as S4
    state "ADMINISTRATIVE_HOLD" as S5
    state "DEACTIVATED" as S6

    %% --- Main Flow ---
    [*] --> S1 : "onSignUpSubmit"
    
    S1 --> S2 : "createAccountRecord"
    note right of S1
        Initial DB entry created
        Role assigned (Worker/Customer)
    end note

    state S2 {
        direction LR
        [*] --> Awaiting_OTP
        Awaiting_OTP --> Awaiting_OTP : "resendOTP [attempts < 3]"
    }
    
    S2 --> S3 : "onOTPValidation [OTP_Valid]"
    
    state S3 {
        direction TB
        state "DATA_COLLECTION" as DC
        state "PENDING_VERIFICATION" as PV
        
        [*] --> DC : "entry / initializeOnboarding"
        DC --> PV : "submitWorkerKYC [Role == 'worker']"
        PV --> DC : "onAdminRejection [reasonProvided]"
        PV --> Onboarded : "onAdminApproval"
        DC --> Onboarded : "submitCustomerProfile [Role == 'customer']"
    }

    S3 --> S4 : "onOnboardingCompletion"
    
    state S4 {
        [*] --> Normal_Usage
        Normal_Usage --> Normal_Usage : "updateProfileDetails"
    }

    %% --- Exceptional & Termination Flows ---
    S4 --> S5 : "onSecurityViolation [Admin]"
    S5 --> S4 : "onComplianceResolution"
    
    S4 --> S6 : "onUserDeletionRequest"
    S5 --> S6 : "onPermanentBan [Admin]"
    
    S6 --> [*] : "purgeRecord / archiveData"

    %% --- Documentation Notes ---
    note left of S4
        Full access to:
        - Job Posting (Customer)
        - Bidding (Worker)
        - Messaging
    end note
```

### Key States Description
*   **PROVISIONED:** The account exists in the database with basic credentials but is not yet usable.
*   **CREDENTIAL_VERIFICATION:** A security phase where the user must prove ownership of their phone/email via OTP.
*   **IDENTITY_ONBOARDING:** A composite state where workers complete a 5-step setup (Portfolio, CNIC) and customers provide basic details. Workers remain in `PENDING_VERIFICATION` until manual admin approval.
*   **ACTIVE_OPERATIONAL:** The standard state for verified users with full platform features.
*   **ADMINISTRATIVE_HOLD:** A restricted state triggered by an Admin for suspicious activity.
*   **DEACTIVATED:** The final state before data archival or deletion.
