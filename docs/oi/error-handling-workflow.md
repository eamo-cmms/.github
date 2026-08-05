# EAMO Technical Specification: Equipment Error Handling & Resolution Workflow

**Project Name:** EAMO (Enterprise Asset & Maintenance Operations Platform)  
**Module Name:** Operator Interface (OI) - Mobile Portal Equipment Error Handling  
**Routes:**  
- Step 1 (Scanner): `http://localhost:5173/portal/error-handling`  
- Step 2 (Resolution Form): `http://localhost:5173/portal/error-handling/:equipmentId`  
**Target View Components:**  
- [`src/views/mobile/portal/error-handling/index.vue`](file:///c:/Users/khanh/Projects/eamo/frontend/src/views/mobile/portal/error-handling/index.vue)  
- [`src/views/mobile/portal/error-handling/handle.vue`](file:///c:/Users/khanh/Projects/eamo/frontend/src/views/mobile/portal/error-handling/handle.vue)

---

## 1. Executive Summary

The **Equipment Error Handling** workflow enables maintenance technicians and shop-floor engineers to resolve active machine breakdowns. By scanning an equipment QR code, the system cross-references active open error logs against configured machine errors. Technicians select the resolved issue, and the system either updates the existing active incident log to `is_handled: true` or records a new completed resolution log.

---

## 2. Architecture & System Flowchart

### 2.1 High-Level Operational Flowchart

```mermaid
graph TD
    A["Technician Accesses Step 1: /portal/error-handling"] --> B["Camera QR Scanner (QrCameraScanner)"]
    B -->|"Scan QR Code"| C["Capture equipmentId & scan_time (getVNNowString)"]
    C --> D["Navigate to Step 2: /portal/error-handling/:equipmentId?scan_time=..."]
    D --> E["Load Data via Concurrent REST API Calls"]
    E --> F["1. GET /v1/equipment/:equipmentId"]
    E --> G["2. GET /v1/equipment/error-monitoring/equipment-error-logs"]
    E --> H["3. GET /v1/equipment-errors?equipment_id=:equipmentId"]
    F --> I["Hydrate Equipment Details & Active Open Logs Map (activeLogMap)"]
    G --> I
    H --> I
    I --> J["Display Scanned Machine Header & Error Resolution Cards"]
    J --> K["Technician Selects Resolved Error Card (selectedErrorId)"]
    K --> L["Technician Clicks 'Confirm Error Handled'"]
    L --> M{"Does an Active Open Log Exist for this Error? (matchedError.log_id)"}
    M -->|"Yes (Open Incident Found)"| N["UPDATE Log Endpoint: PUT /v1/equipment/error-monitoring/equipment-error-logs/:log_id"]
    M -->|"No (Resolved On-site Directly)"| O["CREATE Log Endpoint: POST /v1/equipment/error-monitoring/equipment-error-logs"]
    N --> P["Payload: { is_handled: true, handled_at: timeStr, restarted_at: timeStr, handler_ids: [...] }"]
    O --> Q["Payload: { equipment_id, equipment_error_id, occurred_at, handled_at, restarted_at, is_handled: true }"]
    P --> R["HTTP Success Response"]
    Q --> R
    R --> S["Show Success Toast Notification & Redirect to /portal"]
```

### 2.2 End-to-End Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor Tech as Maintenance Technician
    participant Step1 as Mobile View: Step 1 (index.vue)
    participant Step2 as Mobile View: Step 2 (handle.vue)
    participant API as EAMO REST API Service
    participant DB as System Database

    Tech->>Step1: Open /portal/error-handling & Scan QR
    Step1->>Step2: Router Push /portal/error-handling/:equipmentId?scan_time=...
    
    par Fetch Machine Data
        Step2->>API: GET /v1/equipment/:equipmentId
        API-->>Step2: Return Equipment Metadata
    and Fetch Open Error Logs
        Step2->>API: GET /v1/equipment/error-monitoring/equipment-error-logs
        API-->>Step2: Return Active Error Logs (Filter !is_handled & !handled_at)
    and Fetch Equipment Errors Master
        Step2->>API: GET /v1/equipment-errors?equipment_id=:equipmentId
        API-->>Step2: Return Allowed Equipment Errors List
    end

    Step2->>Step2: Map Active Logs: equipment_error_id -> log_id
    Tech->>Step2: Select Resolved Error Card (selectedErrorId)
    Tech->>Step2: Click "Confirm Error Handled"

    alt Active Open Log Exists (log_id present)
        Step2->>API: PUT /v1/equipment/error-monitoring/equipment-error-logs/:log_id
        Note over API,DB: Update log: is_handled = true, handled_at, restarted_at
    else Direct On-site Resolution (no prior log_id)
        Step2->>API: POST /v1/equipment/error-monitoring/equipment-error-logs
        Note over API,DB: Insert new log: is_handled = true, handled_at, restarted_at
    end

    API-->>Step2: HTTP 200 OK Response
    Step2-->>Tech: Display Success Toast & Navigate to /portal
```

---

## 3. Detailed Step-by-Step Technical Execution

### Step 1: Scanner Route (`/portal/error-handling`)
- **QR Code Scanning**: Utilizes `QrCameraScanner` component to scan machine barcode/QR tag.
- **Timestamp Binding**: Obtains timestamp of resolution start using `getVNNowString()`.
- **Navigation**: Navigates to Step 2 with URL parameter `:equipmentId` and query `scan_time`.

### Step 2: Resolution Route (`/portal/error-handling/:equipmentId`)
- **Data Hydration**: On component mount (`onMounted`):
  1. Fetches machine details via `GET /v1/equipment/:equipmentId`.
  2. Queries active open logs (`GET /v1/equipment/error-monitoring/equipment-error-logs`) and filters unresolved logs (`!handled_at && !is_handled && !deleted_at`). Builds `activeLogMap` mapping `equipment_error_id` ➔ `log_id`.
  3. Loads error categories specific to this machine via `GET /v1/equipment-errors`.
- **Single-Choice Card Selection**: Renders errors as selectable cards (`filteredMasterErrors`).
- **Resolution Execution (`handleSubmit`)**:
  - Validates `selectedErrorId`.
  - Determines resolution strategy:
    - **Existing Log Resolution**: Executes `PUT` request to update existing `log_id` setting `is_handled = true`, `handled_at`, `restarted_at`, and `handler_ids`.
    - **Immediate Log Creation**: Executes `POST` request creating a new resolved error log entry.
  - Redirects to `/portal` upon completion.

---

## 4. API Specifications

### Endpoint A: Update Existing Error Log (Resolve Incident)
- **HTTP Method**: `PUT`
- **URL Path**: `${API_BASE_URL}/v1/equipment/error-monitoring/equipment-error-logs/{log_id}`
- **Authentication**: `Authorization: Bearer <accessToken>`

#### Request Payload Schema
```json
{
  "handled_at": "2026-08-05 11:45:00",
  "restarted_at": "2026-08-05 11:45:00",
  "is_handled": true,
  "notes": "Bearing Alignment & Lubrication Done",
  "handler_ids": ["018e4b3c-0000-7000-a000-000000000001"]
}
```

### Endpoint B: Create Resolved Error Log (Direct On-site Resolution)
- **HTTP Method**: `POST`
- **URL Path**: `${API_BASE_URL}/v1/equipment/error-monitoring/equipment-error-logs`
- **Authentication**: `Authorization: Bearer <accessToken>`

#### Request Payload Schema
```json
{
  "equipment_id": "018e4b3c-9a1d-7212-a123-456789abcdef",
  "equipment_error_id": "018e4b3c-9b2e-7323-b234-567890bcdef1",
  "occurred_at": "2026-08-05 11:45:00",
  "handled_at": "2026-08-05 11:45:00",
  "restarted_at": "2026-08-05 11:45:00",
  "is_handled": true,
  "notes": "Bearing Alignment & Lubrication Done",
  "handler_ids": ["018e4b3c-0000-7000-a000-000000000001"]
}
```
