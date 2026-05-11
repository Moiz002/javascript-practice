# State Diagram 6: Simulated Payment Transaction

This state diagram models the financial lifecycle of a job payment (simulated) from the moment a customer initiates payment through escrow management, platform commission deduction, and final worker payout.

```mermaid
stateDiagram-v2
    direction TB

    %% --- State Definitions ---
    state "PAYMENT_INITIATED" as S1
    state "ESCROW_HELD" as S2
    state "COMMISSION_DEDUCTION" as S3
    state "WORKER_PAYOUT_PENDING" as S4
    state "TRANSACTION_COMPLETED" as S5
    state "PAYMENT_REFUNDED" as S6

    %% --- Main Financial Flow ---
    [*] --> S1 : "onCheckoutSubmit"
    
    S1 --> S2 : "onFundAuthorization"
    note right of S1
        Funds are locked by the platform
        to ensure worker payment security
    end note

    S2 --> S3 : "onJobCompletionApproval [Customer Confirms]"
    
    S3 --> S4 : "calculatePlatformFee [5-10%]"
    note left of S4
        Net amount calculated
        after commission
    end note

    S4 --> S5 : "transferToWorkerWallet"
    
    S5 --> [*] : "generateReceipt / updateHistory"

    %% --- Reversion & Exceptional Paths ---
    S2 --> S6 : "onDisputeResolution [Refund_Decision]"
    S2 --> S6 : "onJobCancellation [Before Work Start]"
    
    S6 --> [*] : "revertSimulatedCredits"

    %% --- Documentation Notes ---
    note left of S2
        Monitored via Admin Dashboard
        for money laundering or
        suspicious payment patterns.
    end note

    note right of S5
        Transaction logged for
        Admin Analytics and
        Tax Reporting.
    end note
```

### Key States Description
*   **PAYMENT_INITIATED:** The user has triggered the payment process. The system validates the balance or simulated card details.
*   **ESCROW_HELD:** A critical trust state. Funds are "parked" by the platform. The worker can see that the money is secured, but they cannot access it until the job is done.
*   **COMMISSION_DEDUCTION:** Once the job is approved, the system calculates and subtracts the platform's service fee before finalizing the worker's share.
*   **WORKER_PAYOUT_PENDING:** The net funds are staged for transfer to the worker's internal wallet or simulated account.
*   **TRANSACTION_COMPLETED:** The successful terminal state. Funds have moved, and the ledger is balanced.
*   **PAYMENT_REFUNDED:** Triggered by cancellations or disputes. Funds are returned to the customer's original simulated source.
