# EAMO Technical Specification: Incident Reporting Workflow

**Project Name:** EAMO (Enterprise Asset & Maintenance Operations Platform)  
**Module Name:** Operator Interface (OI) - Mobile Portal Incident Reporting  
**Route:** `http://localhost:5173/portal/incident-report`  
**Target View Component:** [`src/views/mobile/portal/incident-report/index.vue`](file:///c:/Users/khanh/Projects/eamo/frontend/src/views/mobile/portal/incident-report/index.vue)

---

## 1. Executive Summary

The **Incident Reporting** workflow in the EAMO Mobile Operator Interface (OI) is designed for shop-floor operators to immediately log operational breakdowns, mechanical failures, or unexpected machine stops. Utilizing a 2-step mobile-first wizard, operators scan an equipment QR code, select a pre-configured breakdown category, and record an active incident log in real-time.

---

## 2. Architecture & System Flowchart

### 2.1 High-Level Operational Flowchart

```mermaid
graph TD
    A["Operator Accesses /portal/incident-report"] --> B["Step 1: Real-time Camera QR Scanner (QrCameraScanner)"]
    B -->|"Scan QR Code"| C{"Equipment Recognized in Master Data?"}
    C -->|"Yes"| D["Hydrate Equipment Details (id, code, name)"]
    C -->|"No"| E["Generate Fallback Equipment Entity [Code: rawText]"]
    D --> F["Capture Timestamp (getVNNowString) & Advance to Step 2"]
    E --> F
    F --> G["Step 2: Breakdown Category Selection (Single Choice Chips)"]
    G --> H["Operator Selects Incident Category (selected_error_id)"]
    H --> I["Operator Clicks 'Submit Incident Log'"]
    I --> J{"Validation: Equipment & Category Selected?"}
    J -->|"No"| K["Display Warning Toast Notification"]
    J -->|"Yes"| L["Construct Payload (is_handled: false)"]
    L --> M["Dispatch POST /v1/equipment/error-monitoring/equipment-error-logs"]
    M -->|"HTTP 200/201 Success"| N["Display Success Toast & Redirect to /portal"]
    M -->|"API Error"| O["Display Error Toast & Maintain Form State"]
```

### 2.2 End-to-End Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor Operator as Operator (Mobile OI)
    participant UI as Mobile Portal (Incident View)
    participant Cam as QrCameraScanner Module
    participant API as EAMO REST API Service
    participant DB as System Database

    Operator->>UI: Open /portal/incident-report
    UI->>API: GET /v1/equipment & GET /v1/equipment-errors
    API-->>UI: Return Equipment & Error Master Data
    UI->>Cam: Initialize HTML5 QR Camera Stream
    Operator->>Cam: Present Physical Machine QR Code
    Cam-->>UI: Event @scanned({ rawText, matchedEquipment })
    UI->>UI: Bind scanTimestamp = getVNNowString(), Set Step = 2
    Operator->>UI: Select Error Category Card (formState.selected_error_id)
    Operator->>UI: Click "Submit Incident Log"
    UI->>API: POST /v1/equipment/error-monitoring/equipment-error-logs
    Note over API,DB: Persist active incident log (is_handled = false)
    API-->>UI: HTTP 200/201 Created Response
    UI-->>Operator: Success Message & Router Push /portal
```

---

## 3. Detailed Step-by-Step Technical Execution

### Step 1: Identification & QR Camera Scanning
- **Camera Initialization**: Loads `QrCameraScanner` leveraging HTML5 QR code scanning engine.
- **Master Data Preloading**: On mount (`onMounted`), issues concurrent GET requests to fetch master equipment lists and error types (`/v1/equipment` and `/v1/equipment-errors`).
- **QR Code Recognition**: When a QR code is detected:
  - Matches scanned string against preloaded `equipments` by `id` or `code`.
  - Fallback logic generates a dynamic equipment placeholder if unknown.
  - Captures exact local timestamp via `getVNNowString()`.
  - Automatically transitions state from `step = 1` to `step = 2`.

### Step 2: Breakdown Information & Form Submission
- **Single-Choice Card Matrix**: Displays error categories as interactive chip cards (`filteredMasterErrors`).
- **Live Search Filter**: Offers search capability when master error list exceeds 4 items.
- **Form State Binding**: Binds selected error ID (`formState.selected_error_id`).
- **Submit Handling (`handleSubmit`)**:
  - Validates selected equipment and error category.
  - Constructs `payload` with `is_handled: false` to represent an unresolved open incident.
  - Dispatches `POST` request to the error monitoring backend service.

---

## 4. API Specification

### Endpoint: Create Equipment Error Log
- **HTTP Method**: `POST`
- **URL Path**: `${API_BASE_URL}/v1/equipment/error-monitoring/equipment-error-logs`
- **Authentication**: `Authorization: Bearer <accessToken>`

#### Request Payload Schema
```json
{
  "equipment_id": "018e4b3c-9a1d-7212-a123-456789abcdef",
  "equipment_error_id": "018e4b3c-9b2e-7323-b234-567890bcdef1",
  "occurred_at": "2026-08-05 11:45:00",
  "is_handled": false,
  "notes": "Motor Overheating / High Temperature",
  "handler_ids": ["018e4b3c-0000-7000-a000-000000000001"]
}
```

#### Response Schema (HTTP 201 Created)
```json
{
  "status": "success",
  "message": "Equipment error log created successfully",
  "data": {
    "id": "018e4b3c-9c3f-7434-c345-678901cdef23",
    "equipment_id": "018e4b3c-9a1d-7212-a123-456789abcdef",
    "equipment_error_id": "018e4b3c-9b2e-7323-b234-567890bcdef1",
    "occurred_at": "2026-08-05 11:45:00",
    "handled_at": null,
    "is_handled": false,
    "notes": "Motor Overheating / High Temperature",
    "created_at": "2026-08-05T11:45:01Z"
  }
}
```
