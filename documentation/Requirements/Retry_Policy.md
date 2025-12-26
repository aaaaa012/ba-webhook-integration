# Webhook Retry Policy

## 1. Delivery Attempts
We strive for **Reliability**. If your server does not acknowledge the webhook with a `2xx` status code, we will retry.

### Schedule (Exponential Backoff)
| Attempt | Delay | Cumulative Time |
|:---|:---|:---|
| 1 | Instant | 0s |
| 2 | 5s | 5s |
| 3 | 30s | 35s |
| 4 | 5 mins | ~5.5 mins |
| 5 | 30 mins | ~35 mins |
| 6 | 4 hours | ~4.5 hours |
| 7 | 24 hours | ~1 day |

## 2. Timeout
We wait **5 seconds** for a response. If your server takes longer to process the logic, please:
1. Accept the webhook immediately (`200 OK`).
2. Process the business logic asynchronously (Queue/Worker).

## 3. Disabling Webhooks
If an endpoint fails **100 consecutive times** over 48 hours, the webhook subscription will be automatically **disabled**. You will be notified via email to reactivate it.
