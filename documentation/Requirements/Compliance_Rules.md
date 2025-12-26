# Compliance Rules for Webhooks

## 1. The "Compliance" State
A transaction enters the `COMPLIANCE` state when it triggers an internal risk rule. This is **not a rejection**, but a **hold**.

### Common Triggers
| Rule ID | Name | Description | Response Action |
|:---|:---|:---|:---|
| `CMP_001` | **PEP Match** | Sender/Receiver name matches a Politically Exposed Person list. | Manual Review required. SLA paused. |
| `CMP_002` | **Sanctions Hit** | Match against OFAC/UN sanctions list. | Review required. High probability of rejection. |
| `CMP_003` | **Velocity Limit** | Sender has exceeded the daily limit (e.g., > $10k/day). | Escalation to Source of Funds check. |

## 2. Partner Action Required
When you receive a webhook with `status: "COMPLIANCE"`:
1. **Notify User**: "Your transaction is under review."
2. **Do Not Refund**: Funds are held, not returned yet.
3. **Wait**: Listen for a subsequent `PAID` or `CANCELLED` webhook (usually within 2-4 hours).
