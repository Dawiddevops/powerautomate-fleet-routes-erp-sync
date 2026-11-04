# Fleet Route Dispatch & ERP Sync Automation

[![Power Automate](https://img.shields.io/badge/Power_Automate-Cloud_Flow-0066FF?logo=microsoft-power-automate&logoColor=white)](https://powerautomate.microsoft.com/)
[![Microsoft Forms](https://img.shields.io/badge/Microsoft_Forms-Mobile_Dispatch-008272?logo=microsoft&logoColor=white)](https://forms.office.com/)
[![ERP Integration](https://img.shields.io/badge/ERP-GBS_Logistics_CSV-green)](#architecture)
[![Status](https://img.shields.io/badge/Status-Production_Ready-brightgreen)](#)

> **Enterprise Case Study**: An automated pre-dispatch route reporting and ERP data pipeline for commercial distribution fleets. Replaces manual driver phone calls and Excel re-keying with a mobile-optimized intake form and an automated Power Automate Cloud Flow that validates catalog indexes, checks daily contract pricing, generates 18-column ERP-compliant CSV import batches, and routes dispatch notifications to logistics planners.

---

## Business Impact & Metrics

* **Time Saved per Morning Dispatch**: Reduced route preparation and dispatch lag from **90 minutes to under 5 minutes** per driver.
* **Zero Format Errors**: 100% elimination of delimiter, quoting, and illegal character corruption in ERP import files.
* **Real-time Price Sync**: Automates differential contract price lookups across 3,000+ customer accounts with daily scheduled updates.
* **Volume**: Successfully handling morning route dispatches across 420+ active delivery points and hundreds of cylinder units daily.

---

## Architecture & Data Flow

`mermaid
flowchart TD
    Driver([Driver on Mobile]) -->|Submits 1-15 Route Stops| Forms[Microsoft Forms Intake]
    Forms -->|Triggers on New Response| PA[Power Automate Cloud Flow Engine]
    
    subgraph Validation & Lookups
        PA -->|1. Parse Stops| StopParser[Dynamic Route Stop Parser]
        StopParser -->|2. Map Customer| CustMap[(Client Reference Map: 420+ Locations)]
        StopParser -->|3. Catalog Verification| CatalogCheck{Valid Product SKU?}
        PA -->|4. Contract Pricing| PriceCache[(Synced Pricing JSON: 3,000+ Accounts)]
    end
    
    CatalogCheck -->|Invalid SKU| AlertEmail[Logistics Warning Flag]
    CatalogCheck -->|Valid| CSVBuilder[18-Column ERP CSV Generator]
    PriceCache --> CSVBuilder
    
    subgraph Enterprise ERP Integration
        CSVBuilder -->|Archive Copy| SPDoc[SharePoint Audit Document Library]
        CSVBuilder -->|Automated Attachment| Mailbox[Central Logistics Tablet Dispatch Mailbox]
        Mailbox -->|Direct Ingestion| ERP[(Enterprise Logistics ERP / GBS)]
    end
`

---

## Key System Components

### 1. Driver Intake Interface (Microsoft Forms)
- **Constraint Handling**: Solved Microsoft Forms dropdown option limits (6,800–10,000 character limit per question) by building optimized option formatting algorithms combining customer identifier and town without exceeding payload boundaries.
- **Dynamic Route Structure**: Accommodates 1 to 15 stops per trip, capturing driver metadata, vehicle ID, date overrides, and up to 4 distinct cylinder SKU quantities per stop.

### 2. Power Automate Cloud Flow Engine (25+ Structured Actions)
- **JSON Matrix Lookup**: Embeds fast client reference lookup tables into Flow Compose actions for instantaneous (1)$ client ID resolution.
- **Product SKU Validation**: Verifies driver-entered cylinder categories against the customer's authorized product profile, preventing dispatch rejections at loading bays.
- **ERP CSV Generation**: Builds strict RFC 4180-compliant CSV files with exact column ordering required by legacy ERP ingestion scripts (ORDER_TYPE, EXT_DELIVERY_NUM, DRIVER, VEHICLE_NUM, DELIVERY_DATE, REFERENCE, PRO9, PRO11, PRO33, etc.).
- **Differential Daily Pricing Lookup**: Connects with an automated background price extractor that scans contract rate changes and populates DEL_REMARK pricing codes dynamically.

### 3. Error Handling & Auditing
- **Automated Anomaly Detection**: If a driver selects a discontinued or unmapped product code, the flow flags the row in an HTML alert table and routes priority warnings to the planning team.
- **Non-blocking Execution**: If optional price lookup files are temporarily undergoing sync, the dispatch flow gracefully proceeds using baseline contract configurations, ensuring zero disruption to morning physical departure schedules.

---

## Technical Specifications

| Parameter | Specification |
|:---|:---|
| **Platform** | Microsoft Power Automate Cloud Flow, Microsoft Forms |
| **Data Targets** | SharePoint Document Libraries, Central Logistics Mailbox, ERP Ingestion Batch |
| **Import Format** | Fixed 18-Column Semicolon/Comma-delimited CSV, UTF-8 / Windows-1250 |
| **Capacity** | 420+ client stops per route network, 15 sequential stops per batch |
| **Execution Speed** | ~8-12 seconds from form submission to ERP CSV delivery |

---

## Repository Structure

`	ext
├── README.md                                  # Enterprise Architecture & Operations Guide
├── schema/
│   ├── erp-import-format.json                 # 18-column ERP logistics batch schema
│   └── client-reference-sample.json          # Sanitized customer reference mapping
└── templates/
    └── workflow-definition-spec.json          # Logic Apps / Flow definition skeleton
`

---

## Sanitization Notice
*All customer identifiers, proprietary ERP endpoint addresses, and personnel contact information have been fully sanitized with enterprise placeholders in accordance with standard data privacy guidelines.*
