# System Architecture: WooCommerce Order Management

This document provides a technical deep dive into the n8n workflows, data structures, and operational logic that power the WooCommerce Order Management system.

---

## 🏗️ High-Level System Overview

The system is designed as a modular, event-driven architecture that sits between WooCommerce and back-office tools (Google Sheets, Slack, Gmail).

```mermaid
graph TD
    subgraph WooCommerce
        W1[Order Created Webhook]
        W2[Order Updated Webhook]
        W3[Order API]
    end

    subgraph n8n_Core_Logic
        N1[Intake & Risk Workflow]
        N2[Fulfillment Sync Workflow]
        N3[Problem Handler Workflow]
        N4[Daily Audit Scheduler]
    end

    subgraph Data_Storage
        S1[(Orders Master)]
        S2[(Inventory Sheet)]
        S3[(Problem Log)]
        S4[(Error Log)]
    end

    subgraph Communication_Ops
        C1[Slack Notifications]
        C2[Gmail Customer Updates]
        C3[Gmail Internal Alerts]
    end

    W1 --> N1
    W2 --> N2
    N1 --> S1 & S2
    N1 --> C1 & C2
    N2 --> S1 & S3
    N3 --> S2 & C1
    N4 --> S1 & C1 & C3
    N1 -.-> W3
```

---

## 🔄 Workflow Breakdown

### 1. Order Intake & Risk Assessment (Flow 1)
Triggered by a `POST` request from WooCommerce when a new order is created.

```mermaid
sequenceDiagram
    participant Woo as WooCommerce
    participant n8n as n8n (Intake Flow)
    participant Sheets as Google Sheets
    participant Slack as Slack (#order-review)

    Woo->>n8n: POST /woo-order-created
    n8n->>n8n: Validate & Parse Payload
    n8n->>Sheets: Read Recent Orders & Inventory
    n8n->>n8n: Run Fraud Detection Logic
    n8n->>Sheets: Log Order to Master
    alt Is Flagged?
        n8n->>Slack: Send Flag Alert + Review Buttons
        Slack->>n8n: User clicks Approve/Hold/Cancel
        n8n->>Woo: Update Order Status (REST API)
        n8n->>Slack: Update Message (Confirmed)
    else No Flags
        n8n->>n8n: Send Confirmation Email (Gmail)
        n8n->>Slack: Log New Order in #orders
    end
```

**Fraud Logic Thresholds:**
*   **High Value:** Orders > $500.00.
*   **Velocity:** > 3 orders from same customer in 24h.
*   **Mismatches:** Billing country ≠ Shipping country.
*   **Guest Checkout:** Flagged for high-value orders with no registered customer account.

### 2. Fulfillment & Tracking (Flow 2)
Ensures the Master Sheet and customers are always in sync with WooCommerce status changes.

```mermaid
graph LR
    U[WooCommerce Update] --> P[Parse Status Change]
    P --> M[Update Orders Master]
    P --> L[Append Status Log]
    P --> C{Should Email Customer?}
    C -- Yes --> G[Send Status Gmail]
    C -- No --> S[Log to Slack #orders]
```

### 3. Problem Handling & Inventory Recovery (Flow 3)
Handles failed, cancelled, or refunded orders to prevent revenue loss and stock inaccuracies.

```mermaid
graph TD
    Trigger[Problem Webhook] --> Classify[Classify Problem & Severity]
    Classify --> Log[Log to Problem Orders Sheet]
    Classify --> Alerts[Slack #order-problems + Gmail]
    Alerts --> Action[Slack Interactive Buttons]
    Action -- Restock --> R[Read Inventory -> Add Back Qty -> Update Sheet]
    Action -- Contact --> G[Send Follow-up Gmail to Customer]
```

---

## 📊 Data Schema (Google Sheets)

### **Orders Master**
| Column | Description |
| :--- | :--- |
| `Order ID` | WooCommerce internal ID |
| `Status` | Current lifecycle state (Processing, Completed, etc.) |
| `Flagged` | YES/NO indicator from risk engine |
| `Logged At` | Timestamp of first intake |

### **Inventory**
| Column | Description |
| :--- | :--- |
| `SKU` | Unique product identifier |
| `Current Stock` | Real-time quantity available |
| `Reorder Threshold` | Minimum stock before alerting |

---

## 🚨 Error Handling Strategy

The system uses a **Global Error Trigger** linked to a secondary workflow.

1.  **Detection:** Every node in the main workflow is monitored.
2.  **Classification:**
    *   **CRITICAL:** Failures in Data Logging (Sheets) or Woo Updates.
    *   **WARNING:** Failures in Slack/Email notifications.
3.  **Response:**
    *   Logs the Error Stack Trace and Execution URL to the `Error Log`.
    *   Sends an urgent Slack message with a "View Execution" link for immediate debugging.
