# Partner Webhook Integration Project

## Executive Summary

This repository documents the **Event-Driven Notification System** I led as a  Business Analyst. The system provides real-time updates to partners regarding their remittance transactions, replacing legacy polling mechanisms with a robust **Webhook Push Model**.

This project significantly improved partner experience by reducing latency and ensuring strict compliance adherence through automated triggers.

---

## 1. Business Problem

Before this integration, our external partners relied on a **polling mechanism** to check the status of remittance transactions. This approach created several operational inefficiencies:
*   **High Latency:** Partners often experienced delays of 15-30 minutes in receiving status updates, leading to a poor end-user experience.
*   **Server Load:** Constant polling (thousands of requests per minute) caused unnecessary strain on our API gateway and backend infrastructure, even when no status changes occurred.
*   **Manual Intervention:** `COMPLIANCE` and `FAILED` states often went unnoticed until manual reconciliation, causing funding delays and partner dissatisfaction.
*   **Lack of Auditability:** There was no standardized way to prove *when* a partner was notified of a status change, leading to disputes over SLAs.

## 2. Business Objectives & Value

The primary goal was to shift from a "Pull" to a "Push" model to achieve real-time synchronization.

*   **Real-Time Capabilities:** Reduce status update latency from ~15 mins to **< 2 seconds**.
*   **Operational Efficiency:** Eliminate 90% of polling traffic, freeing up infrastructure resources.
*   **Enhanced Reporting:** Provide partners with detailed fields (e.g., `subReason` for failures) to automate their own support tickets.
*   **Regulatory Compliance:** Ensure immediate notification of AML/PEP blocks (`COMPLIANCE` status) so partners can freeze customer funds instantly.

## 3. My Role (Business Analyst Perspective)

As the **Lead Business Analyst**, I owned the end-to-end delivery of this feature, bridging the gap between business needs and engineering implementation.

*   **Requirement Gathering:** Conducted workshops with the Product Team and Key Partners to define the payload structure and trigger logic.
*   **Solution Design:** Collaborated with Solutions Architects to define the retry logic, security signature (HMAC), and idempotency rules.
*   **Documentation:** Authored the BRD, API Specifications, and Integration Guide. Created Mermaid sequence diagrams to visualize the flow.
*   **Testing & Quality:** Defined UAT scenarios and validated webhooks using sites like `webhook.site` and internal logs.
*   **Stakeholder Management:** Managed expectations across Compliance, Ops, and External Partners during the rollout.

## 4. Stakeholders & Collaboration

| Stakeholder | Collaboration Details |
| :--- | :--- |
| **Product Manager** | Aligned on the roadmap and prioritized the "Compliance Hold" notifications as a P0 requirement. |
| **Engineering Team** | Worked daily with backend devs to clarify edge cases (e.g., "What happens if a partner returns 404 vs 500?"). |
| **Compliance Team** | Defined the specific trigger rules for AML/PEP checks to ensure we weren't leaking sensitive data. |
| **External Partners** | Conducted integration sessions, provided API docs, and troubleshooted their signature verification issues. |

## 5. Functional & Non-Functional Requirements

### Functional Requirements
*   **Event Triggers:** System must trigger webhooks for `PAID`, `CREDITED`, `FAILED`, and `COMPLIANCE` (See *Core Business Logic* below).
*   **Payload Structure:** Must include `transactionId`, `status`, `timestamp`, and `signature`.
*   **Retries:** System must retry delivery if the partner server returns a non-200 status code.
*   **Security:** All payloads must be signed using `HMAC-SHA256` with a shared secret.

### Non-Functional Requirements (NFRs)
*   **Performance:** Webhook must be dispatched within 2 seconds of the internal state change.
*   **Reliability:** 99.9% delivery success rate (using retries).
*   **Scalability:** Support up to 100 webhook events per second.
*   **Auditability:** All delivery attempts (success/fail) must be logged for 30 days.

#### Core Business Logic (Trigger Events)
| Event Status | Trigger Condition | BA Note |
|:---|:---|:---|
| `PAID` | Beneficiary successfully credited. | Final success state. Customer is happy. |
| `CANCELLED` | Transaction rejected by internal logic or bank. | Final failure state. Triggers auto-refund logic. |
| `COMPLIANCE` | Held for AML/PEP review. | **Critical:** Requires manual intervention. Partner must notify sender. |

## 6. User Stories & Acceptance Criteria

**Story 1: Successful Payment Notification**
> "As a **Partner System**, I want to receive a webhook when a transaction is PAID, so that I can notify my customer immediately."
*   **AC1:** Webhook receives `200 OK` from partner.
*   **AC2:** Payload contains correct `transactionId` and status `PAID`.
*   **AC3:** Signature header matches the payload content.

**Story 2: Retry Logic for Downtime**
> "As a **System Administrator**, I want the system to retry notifications if the partner's server is down, so that no data is lost."
*   **AC1:** If partner returns `500`, system waits and retries.
*   **AC2:** Retry intervals follow exponential backoff.
*   **AC3:** Stop retrying after 5 failed attempts and log an alert.

## 7. Process / System Flow Explanation

The integration follows an **Event-Driven Architecture**.

1.  **Event Generation:** A transaction changes status in our Core Banking Engine (e.g., Bank confirms credit).
2.  **Detection:** The Webhook Service detects this state change via a queue listener.
3.  **Construction:** The service constructs the JSON payload and signs it using the partner's specific Secret Key.
4.  **Dispatch:** The system sends a `POST` request to the Partner's registered Callback URL.
5.  **Acknowledgment:**
    *   **Success:** Partner returns `200 OK`.
    *   **Failure:** Partner returns `4xx/5xx` or times out -> **Retry Mechanism** activates.

## 8. API / Integration Examples

### Example Payload: Transaction Paid
```json
{
  "eventId": "evt_8843920194",
  "eventType": "TRANSACTION_UPDATE",
  "timestamp": "2024-02-01T14:30:00Z",
  "data": {
    "transactionId": "TXN_123456789",
    "referenceId": "REF_ABC_999",
    "status": "PAID",
    "subStatus": "CREDITED_TO_BENEFICIARY",
    "currency": "NPR",
    "amount": 100.00,
    "updatedAt": "2024-02-01T14:29:55Z"
  }
}
```

### Security: Signature Verification
Partners must verify the `X-Signature` header.
```http
POST /webhook/listener HTTP/1.1
Host: partner-api.com
X-Signature: sha256=5b34... (HMAC of payload)
Content-Type: application/json
```

## 9. Risks, Edge Cases & Mitigations

| Risk / Scenario | Impact | Mitigation Strategy |
| :--- | :--- | :--- |
| **Partner Downtime** | Webhooks fail to deliver, status sync breaks. | **Exponential Backoff:** Retry 5 times over 24 hours. Provide a manual "Re-trigger" API for partners. |
| **Replay Attacks** | Malicious actor resends an old payload. | **Timestamp Check:** Partners should reject requests older than 5 minutes. |
| **Out-of-Order Delivery** | `COMPLIANCE` arrives *after* `Use` (rare). | **Version/Timestamping:** Payload includes `updatedAt`. Partners must compare with their local state. |
| **Malformed Payload** | Partner cannot parse the JSON. | **Strict Schema Validation:** We use JSON Schema validation before sending any webhook. |

## 10. Success Metrics / KPIs

Post-launch, we tracked the following metrics to validate the project's success:
*   **Latency:** Reduced average notification time from **12 minutes** (polling) to **1.5 seconds**.
*   **System Load:** 40% reduction in API Gateway traffic due to the removal of polling.
*   **Partner Satisfaction:** Positive feedback from 3 major partners who integrated the push notifications within 2 weeks.
*   **Data Integrity:** 100% of "Compliance Hold" events were successfully delivered and acknowledged.

## 11. Deliverables Produced

As the BA, I produced and maintained the following artifacts (archived in this repo):

| Directory | Content Description |
|-----------|-------------------|
| **[Requirements](./documentation/Requirements/)** | BRD, Security Standards, and Retry Policies. |
| **[Process Flows](./documentation/Process-Flows/)** | Mermaid-based Sequence Diagrams handling Happy Paths and Edge Cases. |
| **[API Specification](./documentation/API/)** | Payload definitions and header specifications. |
| **[Examples](./examples/payloads/)** | JSON samples for different event types. |

*See [Compliance Rules](./documentation/Requirements/Compliance_Rules.md), [Security Guide](./documentation/Requirements/Security_Guide.md), and [Retry Policy](./documentation/Requirements/Retry_Policy.md).*

## 12. Key Learnings

*   **Idempotency is King:** Partners will inevitably receive duplicate webhooks (network jitters). Designing for idempotency (using `eventId` or `transactionId` as a unique key) was crucial.
*   **Clear Error Codes Matter:** differentiating between a "Temporary Failure" (try again later) and "Permanent Failure" (bad request) in our retry logic saved days of debugging.
*   **Stakeholder Empathy:** Partners have different tech stacks. Providing code samples in Python, Node, and Java helped speed up their integration time significantly.

---

