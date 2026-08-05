# EAMO Operator Interface (OI) Technical Documentation Index

**Project Name:** EAMO (Enterprise Asset & Maintenance Operations Platform)  
**Target Directory:** `.github/docs/oi/`  

---

## 📌 Technical Workflow Documentation Master Index

This directory contains executive-level technical workflow specifications and architectural flowcharts for the EAMO Mobile Operator Interface (OI) portal.

| Module / Feature | Route URL | Workflow File | Description |
| :--- | :--- | :--- | :--- |
| **Incident Reporting** | `http://localhost:5173/portal/incident-report` | [`incident-report-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/oi/incident-report-workflow.md) | Rapid shop-floor breakdown reporting via QR camera scan & error category selection. |
| **Equipment Error Handling** | `http://localhost:5173/portal/error-handling` | [`error-handling-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/oi/error-handling-workflow.md) | Resolution workflow for maintenance technicians to resolve open machine breakdowns. |
| **Equipment Parameter Logging** | `http://localhost:5173/portal/equipment-log` | [`equipment-log-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/oi/equipment-log-workflow.md) | Operational telemetry logging (Temp, Pressure, RPM) with automated non-empty field filtering. |

---

## 🛠 Architectural Principles & Design System

1. **Mobile-First Operator UX**: Designed specifically for handheld devices and ruggedized industrial tablets on the factory floor.
2. **Camera-Native QR Scanning**: Integrated HTML5 video camera stream with fallback matching against system master data.
3. **Data Integrity & Non-Empty Filtering**: Form submissions strictly filter out un-entered fields to prevent empty database writes.
4. **Bilingual Support**: Fully internationalized supporting English (`en-US`) and Vietnamese (`zh-CN` / `vi`).
