# EAMO Maintenance Planning & Operations Documentation

**Project Name:** EAMO (Enterprise Asset & Maintenance Operations Platform)  
**Target Directory:** `.github/docs/maintain-plan/`  

---

## 📌 Maintenance Management Master Index

This documentation suite provides a business-process-driven overview of the maintenance operations, recurring schedule generation, judgment evaluation lifecycle, and operational output propagation within the EAMO platform.

| Module / Feature | Route URL | Documentation Reference | Operational Scope |
| :--- | :--- | :--- | :--- |
| **Maintenance Planning & Visual Calendar** | `http://localhost:5173/ops/maintenance-plans` | [`maintenance-judgment-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/maintain-plan/maintenance-judgment-workflow.md) | Establishing recurring preventative maintenance plans, configuring cycle intervals, and managing calendar task schedules. |
| **Task Execution & Judgment Lifecycle** | `http://localhost:5173/ops/maintenance-plans` | [`maintenance-judgment-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/maintain-plan/maintenance-judgment-workflow.md) | Evaluating performed maintenance tasks, submitting quality judgments, and recording work observations. |
| **Historical Maintenance Logs & Source Tracking** | `http://localhost:5173/maintenance/maintenance-logs` | [`maintenance-judgment-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/maintain-plan/maintenance-judgment-workflow.md) | Tracking historical maintenance records with automated classification between Scheduled and Direct/Unscheduled interventions. |
| **Executive Performance & Compliance Analytics** | `http://localhost:5173/dashboard/analytics` | [`maintenance-judgment-workflow.md`](file:///c:/Users/khanh/Projects/eamo/.github/docs/maintain-plan/maintenance-judgment-workflow.md) | Real-time monitoring of 30-day compliance rates, overdue task tracking, and technician performance rankings. |

---

## ⚙️ Core Operational Sourcing Framework

EAMO operates a dual-track maintenance architecture designed to capture both planned preventative activities and sudden breakdown interventions:

```
                           OPERATIONAL MAINTENANCE INITIATIVES
                                            │
         ┌──────────────────────────────────┴──────────────────────────────────┐
         ▼                                                                     ▼
[Track 1: Scheduled Maintenance]                              [Track 2: Direct / Unscheduled Maintenance]
• Pre-configured recurrence cycles                            • Immediate response to machine breakdowns
• Forward-looking calendar task creation                      • Direct on-site work logging
• Linked to a formal Maintenance Plan (MP-xxxx)               • No pre-existing schedule required
• Source Badge: "Scheduled (Theo kế hoạch)"                   • Source Badge: "Direct / Unscheduled (Đột xuất / Trực tiếp)"
```

---

## 🔄 5-Tier Judgment Output Ecosystem

When a technician or supervisor completes and judges a maintenance activity, the platform initiates a coordinated update across five key business tiers:

1. **Tier 1: Audit Log Generation:** Creates a permanent historical record capturing the asset, technician, completion date, judgment outcome (`Completed` or `Failed`), and work remarks.
2. **Tier 2: Schedule State Synchronization:** Updates the schedule date to reflect actual completion and marks the task as completed, removing it from pending and overdue backlogs.
3. **Tier 3: Asset Operating Baseline Reset:** Records the latest maintenance timestamp on the physical asset, resetting the accumulated operational hour counter for accurate predictive maintenance tracking.
4. **Tier 4: Executive Dashboard & KPI Recalculation:** Immediately recalculates 30-day compliance percentages, decrements overdue counts, and awards completed task points to assigned technicians on the performance leaderboard.
5. **Tier 5: Analytics & Warning Status Refresh:** Updates the maintenance type distribution charts and reassesses machine health, moving recovered assets back to the `Healthy` status category.
