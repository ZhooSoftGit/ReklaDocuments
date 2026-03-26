# 📄 Business Requirement Document (BRD)

## Unified Support Ticketing System (Rider, Driver & Internal Operations)

---

## 1. 🎯 Purpose

To establish a **structured, scalable, and accountable support system** that:

* Resolves Rider and Driver issues efficiently
* Enables Admin teams to manage operations transparently
* Supports internal coordination through structured ticketing
* Ensures high customer satisfaction and operational control

---

## 2. 🌐 Scope

This system will cover:

### External Support

* Rider support (ride, payment, safety, app issues)
* Driver support (earnings, account, operations)

### Internal Support

* Admin-to-admin tickets for operational workflows
* Cross-team issue handling (Ops, Finance, Tech)

---

## 3. 👥 Stakeholders

### External

* Riders (Customers)
* Drivers

### Internal

* Support Agents
* Support Managers
* Operations Team
* Finance Team
* Compliance / Safety Team

---

## 4. 🧩 Business Objectives

* Reduce issue resolution time
* Improve customer and driver trust
* Provide clear accountability for every issue
* Enable structured internal communication
* Maintain audit and compliance visibility

---

## 5. 📌 Ticket Classification

### 5.1 Rider Issues

* Ride Experience (driver behavior, route issues)
* Payment & Refunds
* Cancellation Disputes
* Safety Concerns
* App/Technical Problems

### 5.2 Driver Issues

* Earnings & Incentives
* Payout Delays
* Ride Allocation Issues
* Account Suspension / Appeals
* Document Verification

### 5.3 Internal Tickets

* Driver onboarding follow-ups
* Payment reconciliation
* Fraud investigation
* System issues / escalations
* Vendor or partner coordination

---

## 6. 🔄 Ticket Lifecycle (Business View)

Every ticket must follow a **standard lifecycle**:

1. **Created** - Issue is raised
2. **Assigned** - Ownership is defined
3. **In Progress** - Investigation started
4. **Waiting** - Awaiting input (user or internal team)
5. **Resolved** - Solution provided
6. **Closed** - Final confirmation
7. **Escalated** - Moved to higher authority if needed

👉 No ticket should remain without an owner at any stage.

---

## 7. 🧭 Ticket Ownership & Responsibility

| Stage       | Owner                             |
| ----------- | --------------------------------- |
| Created     | System                            |
| Assigned    | Support Manager / Auto Allocation |
| In Progress | Support Agent                     |
| Escalated   | Senior Team / Specialist          |
| Closed      | Support Agent / System            |

---

## 8. ⚡ Priority & SLA (Service Commitment)

| Priority | Use Case                              | Expected Resolution          |
| -------- | ------------------------------------- | ---------------------------- |
| Critical | Safety, fraud, major payment issues   | Immediate (within 1-2 hours) |
| High     | Ride disputes, driver earnings issues | Within 6 hours               |
| Medium   | General issues                        | Within 24 hours              |
| Low      | Informational queries                 | Within 48 hours              |

👉 SLA adherence must be tracked and reported.

---

## 9. 🏢 Admin Handling Process

### Step 1: Ticket Intake

* All tickets appear in a centralized queue
* Categorized automatically or manually

### Step 2: Assignment

* Assigned based on:

  * Issue type
  * Team responsibility
  * Workload balancing

### Step 3: Investigation

* Agent reviews details
* May contact user or internal teams
* May request additional information

### Step 4: Action

Examples:

* Refund initiation
* Warning to driver
* Account review
* Technical fix request

### Step 5: Resolution

* Clear response provided to user
* Action documented

### Step 6: Closure

* Ticket closed after confirmation or timeout

---

## 10. 🔁 Escalation Management

Escalation should happen when:

* SLA breach risk
* High-value customer impact
* Safety or compliance issues
* Repeated complaints

Escalation Levels:

* Level 1: Support Agent
* Level 2: Support Manager
* Level 3: Specialist Team (Finance/Compliance)
* Level 4: Leadership (if critical)

---

## 11. 💬 Communication Guidelines

* All communication must be:

  * Clear
  * Professional
  * Time-bound

Channels:

* In-app communication (primary)
* Notifications (status updates)
* Optional: call support for critical issues

---

## 12. 🧾 Internal Ticketing (Operational Backbone)

Internal tickets are used for:

* Cross-team dependencies
* Task tracking
* Operational follow-ups

### Examples:

* Finance verifying payout mismatch
* Ops verifying driver documents
* Tech resolving system bug

👉 Internal tickets must have:

* Owner
* Deadline
* Status tracking

---

## 13. 📊 Monitoring & Reporting

The system must provide visibility on:

### Key Metrics

* Total tickets raised
* Resolution time
* SLA compliance rate
* Escalation rate
* Repeat issues
* Agent productivity

### Insights

* Frequent problem areas
* Driver-related issues trends
* Payment-related complaints

---

## 14. ⭐ Feedback & Quality Control

After resolution:

* Users can rate support experience
* Feedback helps:

  * Improve service quality
  * Evaluate agent performance

---

## 15. 🔔 Notifications & Alerts

* Ticket created -> acknowledgement
* Status changes -> user notified
* Agent response -> user notified
* SLA breach risk -> admin alerted

---

## 16. 🏢 Multi-Company (White-Label) Consideration

Each business/client should have flexibility to:

* Define ticket categories
* Set SLA rules
* Configure escalation paths
* Manage their own support team

👉 Ensures scalability across multiple deployments

---

## 17. ⚠️ Business Rules

* Every ticket must have an owner
* No ticket should exceed SLA without escalation
* All actions must be logged
* Sensitive issues (safety/fraud) must be prioritized
* Duplicate tickets should be minimized or merged

---

## 18. 🚀 Success Criteria

The system is successful if:

* Issues are resolved faster
* Customer and driver satisfaction improves
* Operational workload is structured
* Internal coordination becomes seamless
* Management gets clear visibility

---

## 19. 🔍 Gap Analysis (What Was Missing for Full E2E Coverage)

The existing BRD had strong business intent, but complete execution readiness requires the following additional clarity:

* Defined ticket intake fields (mandatory metadata for routing and audit)
* Lifecycle entry/exit criteria (when a ticket can move to next stage)
* SLA timer behavior (start, pause, resume, breach rules)
* Reopen, duplicate merge, and parent-child ticket handling
* Cross-team handoff rules with ownership transfer accountability
* End-to-end scenario flows by major issue types
* Measurable acceptance criteria to confirm business success in production

---

## 20. 🧱 Detailed Ticket Data Model (Business Mandatory Fields)

Every ticket must store the following minimum business data:

* Ticket ID (unique, immutable)
* Tenant/Company ID (for white-label segregation)
* Ticket Type (Rider / Driver / Internal)
* Category and Subcategory
* Priority
* Current Status
* Requester identity (Rider ID / Driver ID / Employee ID)
* Assigned Team and Assigned Agent
* Related entities (Ride ID, Payment ID, Driver Profile ID)
* Source channel (in-app, web portal, call center)
* Description and attachments
* Created timestamp, first response timestamp, resolution timestamp, closure timestamp
* SLA target and SLA breach flag
* Escalation level and escalation history
* Resolution summary and root cause
* Audit trail of all status and ownership changes

---

## 21. ⏱ SLA Clock & State Transition Rules

### 21.1 SLA Clock Rules

* SLA timer starts at ticket creation time.
* SLA timer pauses only when status is Waiting for Requester.
* SLA timer continues in Assigned, In Progress, and Escalated states.
* SLA breach warning is triggered at 80% of SLA target.
* SLA breach is recorded when target time is exceeded without resolution.

### 21.2 Lifecycle Transition Rules

* Created -> Assigned: Must have category and priority.
* Assigned -> In Progress: Agent must accept ownership.
* In Progress -> Waiting: Agent must log pending dependency (user/internal).
* Waiting -> In Progress: Dependency response received.
* In Progress -> Resolved: Mandatory resolution summary and action evidence.
* Resolved -> Closed: User confirmation or auto-close timeout.
* Any active state -> Escalated: Triggered by policy (SLA risk, safety, compliance).
* Closed -> Reopened: Allowed within defined reopen window (for example 7 days).

---

## 22. 🔁 Complete End-to-End Business Scenarios

### Scenario 1: Rider Refund Dispute (Standard Financial Case)

1. Rider raises ticket from ride history with Ride ID and payment reference.
2. System auto-tags category as Payment & Refunds and sets priority High.
3. Ticket enters centralized queue and is auto-assigned to Payments Support.
4. Agent validates fare, promotions, wallet debit, and gateway transaction.
5. If mismatch confirmed, agent initiates refund action and records transaction ID.
6. Rider receives status notification with expected settlement timeline.
7. Ticket moves to Resolved after refund initiation evidence is attached.
8. Ticket moves to Closed after rider confirmation or auto-close timeout.

Expected Outcome:

* Financial correction completed
* Full audit trail available
* SLA compliance tracked

### Scenario 2: Driver Payout Delay (Cross-Team Dependency)

1. Driver raises payout delay complaint in driver app.
2. System classifies as Driver -> Payout Delays with High priority.
3. Support agent verifies payout cycle status and bank transfer logs.
4. If unresolved, agent creates linked internal Finance ticket.
5. Ownership transfers to Finance while parent ticket remains visible to support.
6. Finance posts reconciliation outcome (failed transfer, bank rejection, etc.).
7. Support communicates final update to driver and confirms next payout action.
8. Parent and child tickets are resolved and closed with linked references.

Expected Outcome:

* Clear inter-team accountability
* No ownership ambiguity during handoff
* Traceable parent-child closure

### Scenario 3: Rider Safety Incident (Critical Escalation Path)

1. Rider submits safety issue during or immediately after trip.
2. System marks ticket as Critical and auto-escalates to Safety queue.
3. Level 2 manager and Compliance team receive instant alerts.
4. Agent attempts immediate rider contact and logs safety checklist actions.
5. Driver account may be temporarily restricted as per policy.
6. Compliance investigation runs with evidence collection and risk classification.
7. Decision is documented (warning, suspension, law enforcement support, closure).
8. Ticket closes only after mandatory safety closure checklist is completed.

Expected Outcome:

* Fast critical response
* Policy-compliant handling
* Complete legal/audit trace

### Scenario 4: Internal Technical Escalation (Support to Engineering)

1. Multiple users report app crash while booking.
2. Support tags incidents under App/Technical Problems and links duplicates.
3. Incident threshold triggers internal Engineering escalation.
4. Engineering receives consolidated internal ticket with logs and impact scope.
5. Workaround or fix ETA is shared back to Support.
6. Support broadcasts proactive communication to impacted users.
7. Once production fix is deployed, related tickets are batch resolved.
8. Post-incident root cause and preventive action are documented.

Expected Outcome:

* Faster incident containment
* Reduced duplicate effort
* Reusable incident knowledge for future prevention

### Scenario 5: Ticket Reopen and Quality Recovery

1. Ticket was marked Resolved but issue persists.
2. User reopens within reopen window.
3. System restores ticket to In Progress and increases priority if repeat issue.
4. Case is routed to higher-skilled queue for second-level review.
5. Corrective action is executed and verified with user evidence.
6. Ticket is resolved again with updated root cause and corrective notes.
7. Quality team flags case for coaching if first resolution was weak.

Expected Outcome:

* Reduced false closures
* Improved first-contact resolution quality over time

---

## 23. 🧭 Operational Controls for End-to-End Integrity

* Auto-assignment must use skill-based routing plus live workload balancing.
* Duplicate detection must suggest merge by same requester + same ride/payment context.
* Each escalation must record trigger reason, owner, and timestamp.
* All user-facing responses must use approved communication templates.
* Sensitive tickets (safety/fraud) require restricted access and masked data exposure.
* Closed tickets must remain searchable for audit and compliance retention period.

---

## 24. ✅ Acceptance Criteria (Business Sign-Off)

This BRD is considered complete and implementation-ready when all conditions below are met:

* 100% tickets are created with mandatory fields.
* 100% tickets have explicit owner in every active state.
* SLA dashboard shows real-time compliance and breach alerts.
* Critical safety tickets are acknowledged within 15 minutes.
* Reopen rate is tracked and reported by category and agent.
* Parent-child internal dependencies are traceable end-to-end.
* All status, owner, and action changes are auditable.
* Tenant-level configuration supports category, SLA, and escalation customization.

---

# ✅ Final Note

This support system is not just a feature - it is the **trust engine of your platform**.

A strong ticketing system ensures:

* Better retention (drivers + riders)
* Operational control
* Scalable growth across multiple companies

---
