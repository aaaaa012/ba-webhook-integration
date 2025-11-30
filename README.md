# Business Analyst – Partner Webhook Integration

## Overview
This repository contains **documentation and artefacts** for the **Partner Webhook Integration** project i have worked on as a Business Analyst. It captures the functional flow, requirements, and key decisions around the webhook that notifies the partner about transaction‑status changes.

- **Partner** calls our **Send API** to create a payout transaction.
- Our system processes the transaction and updates its status (`cancelled`, `compliance`, `paid`).
- Whenever the status changes, we **POST** a webhook payload back to the partner.
- Special handling is required when a transaction gets stuck in **compliance** (e.g., politically exposed persons, duplicate transactions, etc.).

## Repository Structure
```
ba_webhook_integration/
├─ README.md                # This file
├─ .gitignore               # Standard ignore file
└─ docs/
   ├─ overview.md           # Detailed functional overview
   └─ flow_diagram.png      
```

