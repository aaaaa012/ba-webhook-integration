# Partner Webhook Integration Project

[![Role](https://img.shields.io/badge/Role-Business%20Analyst-blue)](https://github.com/topics/business-analyst)
[![Type](https://img.shields.io/badge/Architecture-Event%20Driven-purple)](https://github.com/topics/event-driven)
[![Status](https://img.shields.io/badge/Status-Production%20Live-success)](https://github.com/topics/status)

## 📌 Executive Summary

This repository documents the **Event-Driven Notification System** I designed as a Business Analyst. The system provides real-time updates to partners regarding their remittance transactions, replacing legacy polling mechanisms with a robust **Webhook Push Model**.

**Key Features:**
- 🔔 **Real-Time Alerts**: Instant notifications for `COMPLETED`, `FAILED`, and `COMPLIANCE` status changes.
- 🔐 **Secure Design**: Implemented HMAC-SHA256 signature verification to ensure data integrity.
- 🔁 **Resilience**: Automated exponential backoff retry policy for up to 24 hours of partner downtime.

---

## 📂 Repository Structure

| Directory | Content Description |
|-----------|-------------------|
| **[Requirements](./documentation/Requirements/)** | Business Requirements (BRD), Security Standards, and Retry Policies. |
| **[Process Flows](./documentation/Process-Flows/)** | Mermaid-based Sequence Diagrams handling Happy Paths and Edge Cases. |
| **[API Specification](./documentation/API/)** | Payload definitions and header specifications for the webhook events. |
| **[Examples](./examples/payloads/)** | JSON samples for different event types. |

---

## 🔄 Integration Landscape

The integration connects our **Transaction Core** with External Partners. Instead of partners asking *"Is it done yet?"* every minute (Polling), we tell them *"It's done!"* immediately (Push).

### High-Level Flow
1. **Partner** initiates a transaction via Send API.
2. **Transaction Core** processes the request asynchronously.
3. **Webhook Engine** listens for state changes.
4. **Notifier** pushes a JSON payload to the Partner's registered URL.

---

## ⚙️ Core Business Logic

### Trigger Events
| Event Status | Trigger Condition | BA Note |
|:---|:---|:---|
| `PAID` | Beneficiary successfully credited. | Final success state. |
| `CANCELLED` | Transaction rejected by internal logic or bank. | Final failure state. |
| `COMPLIANCE` | Held for AML/PEP review. | **Critical:** Requires manual intervention or further docs. |

### Compliance Handling
A key part of this project was defining the **Compliance Workflow**. When a transaction hits a PEP (Politically Exposed Person) match:
1. Status updates to `COMPLIANCE`.
2. Webhook sent with `subReason: "PEP_REVIEW"`.
3. Partner is notified to expect delays (SLA stops).

*See [Compliance Rules](./documentation/Requirements/Compliance_Rules.md) for details.*

---

## 🛡️ Security & Reliability

- **Authentication**: All webhooks are signed with a shared secret. Partners verify `X-Webhook-Signature`.
- **Reliability**: If a partner's server returns `500` or times out, we retry:
    - Interval: `2^n` seconds (Exponential Backoff).
    - Max Retries: 5 times over 24 hours.

*See [Security Guide](./documentation/Requirements/Security_Guide.md) and [Retry Policy](./documentation/Requirements/Retry_Policy.md).*

---

## 📝 Artifacts Overview

- **[Business Requirement Document (BRD)](./documentation/Requirements/Webhook_BRD.md)**
- **[Webhook API Spec](./documentation/API/Webhook_Specification.md)**
- **[Event Workflows](./documentation/Process-Flows/Event_Workflows.md)**

---

## ⚠️ Disclaimer
*This repository contains generic and redacted documentation. All secrets, endpoints, and partner names are synthetic.*

