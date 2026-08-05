# EAMO Technical Specification: Equipment Parameter Logging Workflow

**Project Name:** EAMO (Enterprise Asset & Maintenance Operations Platform)  
**Module Name:** Operator Interface (OI) - Mobile Portal Equipment Parameter Logging  
**Routes:**  
- Step 1 (Scanner): `http://localhost:5173/portal/equipment-log`  
- Step 2 (Parameter Matrix Form): `http://localhost:5173/portal/equipment-log/:equipmentId`  
**Target View Components:**  
- [`src/views/mobile/portal/equipment-log/index.vue`](file:///c:/Users/khanh/Projects/eamo/frontend/src/views/mobile/portal/equipment-log/index.vue)  
- [`src/views/mobile/portal/equipment-log/handle.vue`](file:///c:/Users/khanh/Projects/eamo/frontend/src/views/mobile/portal/equipment-log/handle.vue)

---

## 1. Executive Summary

The **Equipment Parameter Logging** workflow in the EAMO Mobile Operator Interface (OI) empowers shop-floor operators to record operational telemetry (e.g. Temperature, Pressure, Vibration, RPM) directly at the physical machine. Following a 2-step QR scan workflow, operators are presented with a dynamic input list of all parameters configured for that specific equipment. **Crucially, any un-entered or blank parameter fields are automatically excluded from the batch persistence payload, saving system storage and preventing empty records**.

---

## 2. Architecture & System Flowchart

### 2.1 High-Level Operational Flowchart

```mermaid
graph TD
    A["Operator Accesses Step 1: /portal/equipment-log"] --> B["Real-time Camera QR Scanner (QrCameraScanner)"]
    B -->|"Scan QR Code"| C["Capture equipmentId & scan_time (getVNNowString)"]
    C --> D["Navigate to Step 2: /portal/equipment-log/:equipmentId?scan_time=..."]
    D --> E["Load Machine & Parameter Metadata"]
    E --> F["1. GET /v1/units (Units of Measurement Master Data)"]
    E --> G["2. GET /v1/equipment/:equipmentId (Includes equipment_parameters)"]
    F --> H["Hydrate Parameter Form Array: parameters = [{ id, name, code, unit_id, min_value, max_value, value: '' }]"]
    G --> H
    H --> I["Render Parameter Entry Matrix (Showing Code, Standard Range, Unit Badge, Input Field)"]
    I --> J["Operator Inputs Values for Measured Parameters (Leaves Non-measured Blank)"]
    J --> K["Operator Clicks 'Confirm Save Parameters'"]
    K --> L["Client-side Filtering Logic: Filter parameters where value.trim() !== ''"]
    L --> M{"Are any Valid Non-Empty Parameters Present?"}
    M -->|"No (validItems.length === 0)"| N["Display Warning Toast: Enter at least 1 parameter"]
    M -->|"Yes (validItems.length > 0)"| O["Construct BatchSavePayload Matrix"]
    O --> P["Dispatch POST /v1/equipment/equipment-parameter/logs/save"]
    P -->|"HTTP 200 Success"| Q["Display Success Toast & Redirect to /portal"]
    P -->|"API Error"| R["Display Error Toast & Retain Form Data"]
```

### 2.2 End-to-End Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor Operator as Shop-floor Operator
    participant Step1 as Mobile View: Step 1 (index.vue)
    participant Step2 as Mobile View: Step 2 (handle.vue)
    participant API as EAMO REST API Service
    participant DB as System Database

    Operator->>Step1: Open /portal/equipment-log & Scan Equipment QR Code
    Step1->>Step2: Router Push /portal/equipment-log/:equipmentId?scan_time=...

    par Fetch Measurement Units
        Step2->>API: GET /v1/units
        API-->>Step2: Return Measurement Units List (ºC, Bar, RPM, etc.)
    and Fetch Machine & Configured Parameters
        Step2->>API: GET /v1/equipment/:equipmentId
        API-->>Step2: Return Machine Data with equipment_parameters Array
    end

    Step2->>Step2: Initialize parameters list with value = '' for each item
    Operator->>Step2: Input values for measured parameters (e.g. Temp: 75.5, Press: 2.1)
    Operator->>Step2: Click "Confirm Save Parameters"
    
    Note over Step2: Filter: validItems = parameters.filter(p => p.value.trim() !== '')
    
    Step2->>API: POST /v1/equipment/equipment-parameter/logs/save
    Note over API,DB: Execute batch insert for valid non-empty parameter logs
    API-->>Step2: HTTP 200 OK (Array of created ParameterLogItems)
    Step2-->>Operator: Display Success Toast & Navigate to /portal
```

---

## 3. Detailed Step-by-Step Technical Execution

### Step 1: Scanner Route (`/portal/equipment-log`)
- **QR Scanning**: Uses `QrCameraScanner` to scan physical equipment labels.
- **Timestamp Capture**: Stores exact local timestamp (`getVNNowString()`).
- **Navigation**: Redirects to Step 2 with `:equipmentId` route parameter and `scan_time` query string.

### Step 2: Parameter Matrix Entry Route (`/portal/equipment-log/:equipmentId`)
- **Metadata Hydration (`loadData`)**:
  - Fetches measurement units via `GET /v1/units` (e.g., ºC, Bar, RPM, Hz).
  - Fetches machine metadata and its specific configured parameters array (`equipment_parameters`) via `GET /v1/equipment/:equipmentId`.
  - Initializes each parameter reactive model with `value: ''`.
- **Dynamic Parameter Matrix UI**:
  - Renders parameter name and code tag.
  - Formats standard min/max thresholds (`formatRangeText`).
  - Displays unit badge (`getUnitName`).
  - Provides clean input field for telemetry entry.
- **Client-Side Non-Empty Filtering (`handleSubmit`)**:
  ```ts
  const validItems = parameters.value.filter(
    (p) => p.value !== undefined && p.value !== null && String(p.value).trim() !== ''
  );
  ```
  - If `validItems.length === 0`, execution halts with a warning message.
  - Constructs `BatchSavePayload` containing **only** valid items.
  - Executes batch save API call `POST /v1/equipment/equipment-parameter/logs/save`.

---

## 4. API Specification

### Endpoint: Batch Save Equipment Parameter Logs
- **HTTP Method**: `POST`
- **URL Path**: `${API_BASE_URL}/v1/equipment/equipment-parameter/logs/save`
- **Authentication**: `Authorization: Bearer <accessToken>`

#### Request Payload Schema
```json
{
  "equipment_id": "018e4b3c-9a1d-7212-a123-456789abcdef",
  "recorded_at": "2026-08-05 11:45:00",
  "parameters": [
    {
      "equipment_parameter_id": "018e4b3c-9b2e-7323-b234-567890bcdef1",
      "unit_id": "018e4b3c-9c3f-7434-c345-678901cdef23",
      "value": "75.5",
      "recorded_at": "2026-08-05 11:45:00"
    },
    {
      "equipment_parameter_id": "018e4b3c-9b2e-7323-b234-567890bcdef2",
      "unit_id": "018e4b3c-9c3f-7434-c345-678901cdef24",
      "value": "2.1",
      "recorded_at": "2026-08-05 11:45:00"
    }
  ]
}
```

#### Response Schema (HTTP 200 OK)
```json
{
  "status": "success",
  "message": "Batch parameter logs saved successfully",
  "data": [
    {
      "id": "018e4b3c-9d4a-7545-d456-789012def345",
      "equipment_id": "018e4b3c-9a1d-7212-a123-456789abcdef",
      "equipment_parameter_id": "018e4b3c-9b2e-7323-b234-567890bcdef1",
      "unit_id": "018e4b3c-9c3f-7434-c345-678901cdef23",
      "value": "75.5",
      "recorded_at": "2026-08-05 11:45:00",
      "created_at": "2026-08-05T11:45:01Z"
    },
    {
      "id": "018e4b3c-9d4a-7545-d456-789012def346",
      "equipment_id": "018e4b3c-9a1d-7212-a123-456789abcdef",
      "equipment_parameter_id": "018e4b3c-9b2e-7323-b234-567890bcdef2",
      "unit_id": "018e4b3c-9c3f-7434-c345-678901cdef24",
      "value": "2.1",
      "recorded_at": "2026-08-05 11:45:00",
      "created_at": "2026-08-05T11:45:01Z"
    }
  ]
}
```
