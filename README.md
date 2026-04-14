# WooCommerce Order Management & Automation (n8n)

A robust, enterprise-grade order management and automation system built with n8n. This project bridges the gap between a WooCommerce storefront and back-office operations, providing automated fraud detection, inventory recovery, Slack-based interactive reviews, and detailed operational auditing.

## 🚀 Key Features

*   **Intelligent Order Intake:** Automatically processes new orders, validates payloads, and logs them to a Master Google Sheet.
*   **Fraud & Risk Engine:** Custom logic flags high-value orders, shipping/billing mismatches, and high-frequency customers for manual review.
*   **Interactive Slack Operations:** Approve, Hold, or Cancel flagged orders directly from Slack via interactive webhooks.
*   **Inventory Recovery:** One-click "Restock" functionality for cancelled or failed orders to ensure inventory accuracy.
*   **Automated Customer Communication:** Sends personalized Gmail updates for status changes (Processing, Shipped, On Hold).
*   **Daily Operational Audit:** A scheduled 8 AM digest reporting revenue, SLA breaches (stuck orders), and unresolved issues.
*   **Global Error Handling:** Dedicated error workflow that classifies failures by severity and alerts the team via Slack and Email.

## 🛠️ Tech Stack

*   **Automation:** [n8n](https://n8n.io/) (Low-code workflow automation)
*   **E-commerce:** WooCommerce (via Webhooks & API)
*   **Database:** Google Sheets (Master Orders, Inventory, Status Logs, Error Logs)
*   **Notifications:** Slack (Operations & Alerts)
*   **Email:** Gmail (Customer updates & Critical alerts)

## 📋 Prerequisites

*   An n8n instance (self-hosted or Cloud).
*   A WooCommerce store with REST API access.
*   A Google Cloud Project with Sheets and Gmail APIs enabled.
*   A Slack App with incoming webhook or bot tokens.

## ⚙️ Setup Instructions

1.  **n8n Import:**
    *   Import `Woocommerce Order Management.json` into a new n8n workflow.
    *   Import `Woocommerce Order Management - Error Workflow.json` into a second workflow.
2.  **Credentials:**
    *   Configure your WooCommerce, Google Sheets, Gmail, and Slack credentials in n8n.
3.  **WooCommerce Webhooks:**
    *   Set up a webhook in WooCommerce (`Settings > Advanced > Webhooks`) for `order.created` and `order.updated` pointing to your n8n Production Webhook URLs.
4.  **Google Sheets Setup:**
    *   Create a spreadsheet with tabs for: `Orders Master`, `Inventory`, `Order Status Log`, `Problem Orders`, `Daily Summary`, and `Error Log`.
5.  **Environment Variables:**
    *   Update the `n8n-nodes-base.code` nodes with your specific thresholds (e.g., `HIGH_VALUE_THRESHOLD`, `SLA_PROCESSING_HOURS`).

## 📂 Project Structure

```text
├── workflow/
│   ├── Woocommerce Order Management.json          # Main logic (Intake, Updates, Problems)
│   └── Woocommerce Order Management - Error Workflow.json # Global error handler
├── data/
│   └── Woocommerce Order Management.xlsx          # Reference sheet structure
└── architecture.md                                # Deep dive into the system logic
```

## ⚖️ License

Distributed under the MIT License. See `LICENSE` for more information.
