# Webhook Security Guide

## 1. Signature Verification
To ensure that the webhook events are genuinely from our system and haven't been tampered with, we sign every payload using **HMAC-SHA256**.

### How to Verify
1. **Secret Key**: You will be provided with a `webhook_secret` (starts with `whsec_...`) upon integration.
2. **Header**: Extract the signature from the `X-Webhook-Signature` header.
3. **Compute**: Create a SHA256 HMAC of the raw request body using your secret.
4. **Compare**: If your computed hash matches the header, the request is valid.

**Python Example:**
```python
import hmac
import hashlib

def verify_signature(payload, signature, secret):
    computed = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()
    return hmac.compare_digest(computed, signature)
```

## 2. IP Whitelisting
For additional security, you can whitelist our outbound IP addresses.
- `192.168.1.50`
- `192.168.1.51`

## 3. Replay Attacks
Every webhook includes an `eventId` and `timestamp`.
- **Deduplication**: Store `eventId` to prevent processing the same event twice.
- **Freshness**: Reject signatures where `timestamp` is older than 5 minutes.
