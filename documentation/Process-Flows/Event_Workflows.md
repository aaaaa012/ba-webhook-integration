# Event Workflows

## 1. Happy Path: Transaction Paid

This flow illustrates the standard payout lifecycle where the transaction goes straight to completion.

```mermaid
sequenceDiagram
    autonumber
    participant Partner as Partner System
    participant Gateway as Send API
    participant Core as Transaction Core
    participant Webhook as Webhook Engine

    Partner->>Gateway: POST /send (Create TX)
    Gateway-->>Partner: 202 Accepted

    Core->>Core: Process Transaction
    Core->>Core: Credit Beneficiary (Success)
    
    Core->>Webhook: Event: STATUS_CHANGED (PAID)
    Webhook->>Webhook: Sign Payload (HMAC)
    Webhook->>Partner: POST /webhook/callback
    Partner-->>Webhook: 200 OK (Ack)
```

## 2. Compliance Hold Path

This flow shows what happens when a transaction is flagged for review.

```mermaid
sequenceDiagram
    autonumber
    participant Partner
    participant Core
    participant Compliance
    participant Webhook

    Partner->>Core: Create TX
    Core->>Compliance: Screen Sender/Receiver
    
    alt HIT FOUND (PEP/Sanction)
        Compliance-->>Core: Flag for Review
        Core->>Webhook: Event: STATUS_CHANGED (COMPLIANCE)
        Webhook->>Partner: POST /callback (Details: "Pending Review")
    end

    opt Manual Review (Later)
        Compliance->>Core: Approve TX
        Core->>Webhook: Event: STATUS_CHANGED (PAID)
        Webhook->>Partner: POST /callback
    end
```

## 3. Retry Logic (Resilience)

This flow demonstrates the exponential backoff mechanism when the partner's server is down.

```mermaid
sequenceDiagram
    participant Webhook
    participant Partner

    Webhook->>Partner: POST /callback (Attempt 1)
    Partner-->>Webhook: 500 Internal Server Error
    
    Note over Webhook: Wait 5 seconds
    
    Webhook->>Partner: POST /callback (Attempt 2)
    Partner-->>Webhook: 504 Gateway Timeout

    Note over Webhook: Wait 30 seconds
    
    Webhook->>Partner: POST /callback (Attempt 3)
    Partner-->>Webhook: 200 OK
    
    Note over Webhook: Retry Queue Cleared
```
