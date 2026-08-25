# EAMO Technical Documentation & Feature Index

**Project Name:** EAMO (Enterprise Asset & Maintenance Operations Platform)  
**Target Directory:** `.github/docs/oi/`  

---

## 📌 Technical Workflow Documentation Master Index

This directory contains executive-level technical workflow specifications, architectural flowcharts, and security access control documentation for the EAMO platform.

| Module / Feature | Route URL | Workflow / Doc Reference | Description |
| :--- | :--- | :--- | :--- |
| **Dynamic Role-Based Access Control (RBAC)** | `http://localhost:5173/company/users` | [Dynamic Permissions Specification](#-dynamic-role-based-access-control-rbac) | Granular permission management for Manager and Engineer roles via Admin control modal. |
| **Incident Reporting** | `http://localhost:5173/portal/incident-report` | [`incident-report-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/oi/incident-report-workflow.md) | Rapid shop-floor breakdown reporting via QR camera scan & error category selection. |
| **Equipment Error Handling** | `http://localhost:5173/portal/error-handling` | [`error-handling-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/oi/error-handling-workflow.md) | Resolution workflow for maintenance technicians to resolve open machine breakdowns. |
| **Equipment Parameter Logging** | `http://localhost:5173/portal/equipment-log` | [`equipment-log-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/oi/equipment-log-workflow.md) | Operational telemetry logging (Temp, Pressure, RPM) with automated non-empty field filtering. |

---

## 🔐 Dynamic Role-Based Access Control (RBAC)

EAMO implements a comprehensive **Dynamic Permission Matrix & Policy Auto-Discovery System** that balances security boundaries with flexible shop-floor administration.

### UI Overview: User Dynamic Permissions Matrix
![User Dynamic Permissions](../../img/rbac.png)

### Key Architectural Characteristics:

1. **Strict Role Boundaries (Immutable Constraints):**
   - **Admin:** Possesses unrestricted access to all abilities across the entire platform via Laravel `Gate::before(fn($user) => $user->hasRole(UserRole::Admin) ? true : null)`.
   - **Guest:** Read-only access (`view`, `viewAny`) across all modules; 100% of data mutations (`create`, `update`, `delete`, `judge`, `complete`, `import`, `save`) are strictly forbidden.
   - **Engineer:** Hard-blocked at both Policy and FormRequest layers from Organization administration (`Company`, `Department`, `User` management). Granted granular technical permissions in Masterdata, Checklist, Maintenance, and Monitoring modules.
   - **Manager:** Authorized for both Organization and Technical operations, strictly governed by dynamic assigned permissions.

2. **5 Standardized Permission Groups:**
   - **Organization Management:** Company, Department, and User CRUD (*Manager & Admin only*).
   - **Equipment Masterdata:** Equipment, Categories, Parameters, Error Codes, States, and Units.
   - **Checklist Operations:** Session lifecycle, Result evaluation/judgment, and Schedule completion.
   - **Maintenance Operations:** Planning, Approvals/Judgment, Schedules, and Maintenance logs.
   - **Monitoring & Operating Logs:** Incident telemetry, Machine operating runtime hours, and Parameter logs.

3. **Administration Interface (`/company/users`):**
   - **Conditional Action Button:** "Permissions" button appears exclusively for authenticated Admins on `Manager` and `Engineer` rows (automatically hidden on `Admin`, `User`, and `Guest` rows).
   - **Minimalist 1200px Modal:** Features clean Ant Design `<Tabs>` categorization, live keyword search, Select All / Deselect Group toggles, and seamless batch synchronization via `PUT /api/users/{user}/permissions`.

---

## 🛠 Architectural Principles & Design System

1. **Zero-Config Policy Auto-Discovery**: Automatic namespace resolution mapping `[Namespace]\Models\[Model]` to `[Namespace]\Policies\[Model]Policy` across all modules.
2. **Mobile-First Operator UX**: Designed specifically for handheld devices and ruggedized industrial tablets on the factory floor.
3. **Camera-Native QR Scanning**: Integrated HTML5 video camera stream with fallback matching against system master data.
4. **Data Integrity & Non-Empty Filtering**: Form submissions strictly filter out un-entered fields to prevent empty database writes.
5. **Bilingual Support**: Fully internationalized supporting English (`en-US`) and Vietnamese (`zh-CN` / `vi`).
