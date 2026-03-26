# Business Requirements Document (BRD) - Rating & Feedback System

| Field | Details |
|------|---------|
| Document ID | BRD-REKLA-RATING-FEEDBACK-007 |
| Version | 1.1 |
| Status | Draft (Business Complete for Architecture Handover) |
| Date | March 25, 2026 |
| Prepared By | ZhooSoft Team |
| Parent BRDs | [01_BRD_Rekla_Ride_App.md](01_BRD_Rekla_Ride_App.md), [02_BRD_Rider_App.md](02_BRD_Rider_App.md), [03_BRD_Driver_App.md](03_BRD_Driver_App.md), [04_BRD_Admin_Portal.md](04_BRD_Admin_Portal.md) |

---

## 1. Purpose

This BRD defines the business requirements for a complete rating and feedback system covering rider-to-driver and driver-to-rider feedback, quality monitoring, low-rating intervention, abuse handling, and transparent operational governance.

The goal is to ensure trust, service quality improvement, and fair accountability while remaining scalable for multi-company deployments.

---

## 2. Scope

### In Scope

- Post-ride rating capture for riders and drivers.
- Star ratings and optional structured feedback reasons.
- Rating and feedback lifecycle, validation, and moderation policy.
- Low-rating interventions and escalation management.
- Visibility rules for riders, drivers, support, and admins.
- Dispute handling and correction governance.
- KPI reporting and business acceptance criteria.

### Out of Scope

- Technical design of scoring engines.
- UI visual design details.
- Legal policy drafting beyond business rule references.

---

## 3. Business Objectives

- Improve rider and driver trust through fair feedback loops.
- Detect and correct service-quality risks early.
- Reduce repeated poor experience incidents.
- Provide auditable and non-manipulative rating governance.
- Enable support and operations teams to act using consistent policy.

---

## 4. Stakeholders

### External

- Riders
- Drivers

### Internal

- Product Team
- Support Team
- Operations Team
- Compliance and Risk Team
- Admin Team

---

## 5. Rating Model & Participation Rules

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RF01 | Rating request shall be triggered only for successfully completed rides. | Must |
| FR-RF02 | Rider shall be allowed to rate driver. | Must |
| FR-RF03 | Driver shall be allowed to rate rider where policy enables bilateral rating. | Should |
| FR-RF04 | Rating scale shall be 1 to 5 stars. | Must |
| FR-RF05 | Rating submission window shall be business configurable (for example 24 hours). | Must |
| FR-RF06 | If rating window expires without submission, rating event shall close without synthetic score injection unless policy explicitly allows. | Must |
| FR-RF07 | One party can submit at most one rating per role per ride. | Must |

---

## 6. Feedback Content Rules

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RF08 | Users may provide optional free-text feedback with rating submission. | Must |
| FR-RF09 | System shall provide predefined feedback reasons to standardize classification. | Must |
| FR-RF10 | Feedback reason catalog shall be role-based (rider reasons and driver reasons). | Must |
| FR-RF11 | Inappropriate content policy shall apply to free-text feedback with moderation workflow. | Must |
| FR-RF12 | Sensitive or safety-related feedback reasons shall trigger priority review routing. | Must |

Example rider reasons:

- Driving behavior
- Route handling
- Vehicle condition
- Timeliness
- Professional conduct

Example driver reasons:

- Rider behavior
- Pickup delay
- Payment issue at trip completion
- Safety concern

---

## 7. Rating Lifecycle & State Management

Each rating interaction must follow controlled lifecycle states:

1. Eligible (ride completed, within rating window)
2. Submitted
3. Validated
4. Published (visible in allowed views)
5. Under Review (if disputed/flagged)
6. Confirmed, Corrected, or Removed (policy-governed outcome)

Business rule:

- Every review action must preserve audit history and reason code.

---

## 8. Validation, Integrity & Abuse Controls

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RF13 | Duplicate ratings for the same ride and same role shall be blocked. | Must |
| FR-RF14 | Self-rating or collusive manipulation patterns shall be detected and flagged. | Must |
| FR-RF15 | Ratings from invalid or reversed ride outcomes shall follow invalidation policy. | Must |
| FR-RF16 | Suspicious rating velocity or abnormal feedback patterns shall support risk review. | Must |
| FR-RF17 | Manual rating removal must be role-restricted and auditable with reason and approver. | Must |

---

## 9. Score Calculation & Aggregation Rules

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RF18 | Driver score shall be calculated using configurable aggregation policy (for example last N rated rides). | Must |
| FR-RF19 | Rider score policy, where enabled, shall use separate configurable aggregation logic. | Should |
| FR-RF20 | Score presentation shall include both average rating and rating count to avoid misleading interpretation. | Must |
| FR-RF21 | New users with insufficient ratings shall use explicit low-confidence display treatment as defined by policy. | Must |
| FR-RF22 | Any weighting policy changes shall apply prospectively and be policy-announced. | Must |

---

## 10. Visibility & Communication Rules

Rider-facing visibility:

- Driver average rating and rating count before/at ride decision points as policy allows.
- Rating submission confirmation and submission window reminders.

Driver-facing visibility:

- Own average rating, rating count, trend indicator, and relevant feedback categories.
- Alerts on low-rating risk and required corrective actions.

Internal visibility:

- Support and admin roles can view detailed rating history, flags, and review actions per permissions.

Communication rule:

- All notifications must use clear, non-technical, and non-accusatory language.

---

## 11. Low Rating Handling & Escalation

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RF23 | Low-rating threshold must be configurable by policy (example baseline 3.5). | Must |
| FR-RF24 | Crossing threshold shall trigger staged intervention workflow (warning, coaching, review, restriction). | Must |
| FR-RF25 | Safety-related low-rating patterns shall be escalated immediately to Compliance and Risk teams. | Must |
| FR-RF26 | Driver restriction decisions must include reason, review owner, and recovery path criteria. | Must |
| FR-RF27 | Post-intervention performance shall be monitored over defined review window. | Must |

---

## 12. Dispute & Moderation Workflow

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RF28 | Users must have a defined process to dispute unfair or abusive ratings within a configured time window. | Must |
| FR-RF29 | Dispute cases shall include SLA, owner, and status visibility for support operations. | Must |
| FR-RF30 | Moderation outcomes shall be one of: retain, redact feedback text, remove rating, or escalate. | Must |
| FR-RF31 | Critical complaint-linked ratings shall be cross-linked to support tickets when applicable. | Must |
| FR-RF32 | All moderation decisions shall be auditable and policy-consistent. | Must |

---

## 13. Admin Governance & Policy Controls

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RF33 | Authorized admin roles shall configure rating windows, thresholds, and feedback reason catalogs. | Must |
| FR-RF34 | Policy changes shall have effective dates and shall not silently rewrite historical outcomes. | Must |
| FR-RF35 | Maker-checker approval shall be required for critical quality-policy changes. | Must |
| FR-RF36 | Admin actions impacting ratings must be logged with actor, timestamp, reason, and before-after state. | Must |

---

## 14. End-to-End Business Scenarios

### Scenario A: Standard Positive Feedback Loop

1. Ride completes successfully.
2. Rider submits 5-star rating with positive reason.
3. Rating is validated and published.
4. Driver score updates in allowed views.

Expected Outcome:

- Healthy feedback cycle supports trust and quality recognition.

### Scenario B: Low-Rating Intervention

1. Driver receives repeated low ratings within review window.
2. Threshold breach triggers warning and coaching workflow.
3. If trend persists, case escalates for operational review.
4. Restriction or recovery plan is applied as per policy.

Expected Outcome:

- Quality issues are detected and corrected before wider impact.

### Scenario C: Rating Dispute and Moderation

1. Driver disputes rating as abusive/unfair.
2. Support reviews ride context, flags, and feedback content.
3. Decision is recorded with reason and communicated.
4. Score impact is retained or corrected per policy.

Expected Outcome:

- Fairness and transparency maintained with auditable decisions.

### Scenario D: Safety-Sensitive Feedback

1. Rider submits low rating with safety-related reason.
2. Case is auto-routed for priority compliance review.
3. Interim risk action may be applied.
4. Final decision is documented and tracked.

Expected Outcome:

- Safety risks receive immediate and controlled attention.

---

## 15. Reporting & Business KPIs

The business must monitor:

- Average rating by city/tenant/time period.
- Rating submission rate after completed rides.
- Low-rating incidence and repeat-low-rating rate.
- Dispute rate, moderation turnaround time, and reversal rate.
- Coaching effectiveness and recovery success after intervention.
- Safety-linked feedback volume and closure outcomes.

Reporting dimensions:

- Tenant/company
- Geography
- Service type
- Time window
- Driver cohort

---

## 16. Multi-Company (White-Label) Considerations

Each tenant should be able to configure:

- Rating window duration.
- Low-rating thresholds and intervention policy.
- Feedback reason catalogs and language templates.
- Visibility policy for rating display.

Isolation rule:

- Ratings, moderation records, and policy settings must remain tenant-isolated.

---

## 17. Assumptions & Dependencies

1. Ride completion events are reliable and available for rating eligibility checks.
2. Support tooling exists or will exist for dispute handling and moderation actions.
3. Compliance policy defines handling standards for safety-related feedback.
4. Notification channels are available to communicate rating reminders and outcomes.

---

## 18. Business Acceptance Criteria

This BRD is business-complete for architecture handover when:

1. Rating, feedback, moderation, and escalation rules are unambiguous.
2. End-to-end scenarios cover normal, dispute, low-rating, and safety-sensitive paths.
3. Admin governance and audit requirements are formally documented.
4. KPI definitions and ownership are agreed by Product, Ops, and Support.
5. Multi-tenant configurability and isolation are clearly defined.

---

## 19. Final Statement

The platform shall implement a fair, auditable, and policy-governed rating and feedback system that improves trust, detects quality risks early, supports transparent intervention, and enables measurable service-quality improvements across riders and drivers.
