# Webhook API Specification

**Version:** 1.0  
**Method:** `POST`  
**Content-Type:** `application/json`

---

## 1. Request Structure

### 1.1 Headers
| Header | Description | Example |
|:---|:---|:---|
| `Content-Type` | Payload format. | `application/json` |
| `X-Webhook-ID` | Unique ID for idempotency handling. | `evt_123456789` |
| `X-Webhook-Signature` | HMAC-SHA256 signature for verification. | `sha256=a1b2c3d4...` |
| `X-Event-Type` | The type of event triggering the hook. | `transaction.updated` |

### 1.2 Payload Body (Standard)

```json
{
  "eventId": "evt_550e8400-e29b-41d4-a716-446655440000",
  "eventType": "transaction.updated",
  "timestamp": "2023-10-27T10:00:00Z",
  "data": {
    "transactionId": "TX_987654321",
    "partnerReference": "PARTNER_REF_001",
    "status": "PAID",
    "reason": "Success",
    "subReason": null,
    "amount": {
      "value": 100.00,
      "currency": "USD"
    },
    "metadata": {
      "customer_segment": "vip"
    }
  }
}
```

---

## 2. Event Types & Statuses

| Event Type | Status | Description |
|:---|:---|:---|
| `transaction.updated` | `PAID` | Transaction successfully credited to beneficiary. |
| `transaction.updated` | `CANCELLED` | Transaction rejected (bank error, insufficient funds). |
| `transaction.updated` | `COMPLIANCE` | Transaction held for review (PEP, Sanctions). |

---

## 3. Error Handling
If the partner endpoint returns any status code **other than 2xx**, the system treats it as a failure and initiates the **Retry Policy**.

- **Success:** `200 OK`, `201 Created`, `202 Accepted`
- **Failure:** `400 Bad Request`, `401 Unauthorized`, `500 Server Error`, `504 Gateway Timeout`
