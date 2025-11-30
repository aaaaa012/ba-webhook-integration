# Business Analyst – Partner Webhook Integration

## Overview
This repository contains **documentation and artefacts** for the **Partner Webhook Integration** project you worked on as a Business Analyst. It captures the functional flow, requirements, and key decisions around the webhook that notifies the partner about transaction‑status changes.

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
   ├─ overview.md           # Detailed functional overview (includes Mermaid diagram)
   └─ flow_diagram.png      # (no longer needed – diagram now in Mermaid)
```

## How to Use
1. **Clone** the repo (once created on GitHub) to your local machine.
   ```bash
   git clone https://github.com/your-org/ba-webhook-integration.git
   cd ba-webhook-integration
   ```
2. Open `docs/overview.md` for a deep‑dive into the business rules.
3. Share the repo with developers, QA, product owners, and other analysts as the single source of truth for the integration.

---

*Created by Antigravity – Business Analyst assistant*
