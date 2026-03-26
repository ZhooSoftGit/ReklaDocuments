# Business Requirements Document (BRD) - Driver Pricing Model
## Rekla Ride App

| Field         | Details                                          |
|---------------|--------------------------------------------------|
| Document ID   | BRD-REKLA-DRIVER-PRICING-005                     |
| Version       | 1.1                                              |
| Status        | Draft (Business Complete for Architecture Handover) |
| Date          | March 25, 2026                                   |
| Prepared By   | ZhooSoft Team                                    |
| Parent BRDs   | [BRD-REKLA-MASTER-001](01_BRD_Rekla_Ride_App.md), [BRD-REKLA-DRIVER-003](03_BRD_Driver_App.md), [BRD-REKLA-ADMIN-004](04_BRD_Admin_Portal.md) |

---

## Table of Contents

1. [Purpose & Scope](#1-purpose--scope)
2. [Business Context & Pricing Principles](#2-business-context--pricing-principles)
3. [Functional Requirements](#3-functional-requirements)
   - [3.1 Pricing Model Summary](#31-pricing-model-summary)
   - [3.2 Model Availability & Default Assignment](#32-model-availability--default-assignment)
   - [3.3 Commission Model](#33-commission-model)
   - [3.4 Package Model](#34-package-model)
   - [3.5 Package Purchase & Payment Handling](#35-package-purchase--payment-handling)
   - [3.6 Driver Choice & Model Transition Rules](#36-driver-choice--model-transition-rules)
   - [3.7 Transparency & Driver Communication](#37-transparency--driver-communication)
   - [3.8 Admin Configuration & Operational Controls](#38-admin-configuration--operational-controls)
   - [3.9 Exceptions, Validation & Fallback Rules](#39-exceptions-validation--fallback-rules)
   - [3.10 Business Governance: Eligibility, Refunds, and Adjustments](#310-business-governance-eligibility-refunds-and-adjustments)
   - [3.11 End-to-End Business Scenarios](#311-end-to-end-business-scenarios)
   - [3.12 Revenue Protection, Compliance, and Misuse Controls](#312-revenue-protection-compliance-and-misuse-controls)
   - [3.13 Business Monitoring & Decision KPIs](#313-business-monitoring--decision-kpis)
   - [3.14 Rollout, Change Management & Communication Policy](#314-rollout-change-management--communication-policy)
4. [Technical Specification Readiness Inputs](#4-technical-specification-readiness-inputs)
   - [4.1 Pricing States](#41-pricing-states)
   - [4.2 Core Data Objects](#42-core-data-objects)
   - [4.3 Configurable Business Rules](#43-configurable-business-rules)
   - [4.4 Key Events & Notifications](#44-key-events--notifications)
5. [Driver Pricing Non-Functional Requirements](#5-driver-pricing-non-functional-requirements)
6. [Assumptions & Dependencies](#6-assumptions--dependencies)
7. [Business Acceptance Criteria](#7-business-acceptance-criteria)
8. [Approval](#8-approval)

---

## 1. Purpose & Scope

This document defines the business requirements for the **Driver Pricing Model** in the Rekla platform. It covers how the platform monetizes driver participation through either a **Commission Model** or a **Package Model**, how drivers view pricing before accepting rides, and how the system falls back safely when package conditions are not met.

**In Scope:** Driver-side pricing model selection, per-ride commission visibility, package subscription rules, package validity handling, fallback behavior, driver transparency requirements, and related admin configuration controls.
**Pricing Rule Precedence Note:** Where any older Driver App or Admin Portal wording conflicts with this document on driver monetization, this Driver Pricing BRD shall govern pricing-model behavior.
**Business Payment Boundary Note:** This document governs platform monetization from drivers only. Rider fare collection remains separate from package purchase and is not changed by this BRD.
**Wallet Boundary Note:** Driver wallet usage is limited to commission settlement in Commission Model flows unless future finance policy explicitly expands wallet scope.
**Out of Scope:** Rider fare rule design, promo engine design, gateway implementation detail, accounting journal logic, tax policy design, and UI visual design specification.

---

## 2. Business Context & Pricing Principles

The platform shall support a simple and transparent pricing experience for drivers with only two monetization choices:

1. **Pay per ride** through commission deduction.
2. **Pay once and ride unlimited** through a time-bound package.

The pricing model is designed around the following business principles:

- Drivers shall understand the active pricing rule before accepting work.
- Only one pricing model shall be active for a driver at a time.
- Package failures or expiry shall not block normal driver operations by default; the platform shall safely fallback to Commission Model.
- No hidden charges shall be applied.
- Pricing logic shall remain configurable for future business changes without redesigning the core flow.

### Pricing Model Comparison

| Attribute | Commission Model | Package Model |
|-----------|------------------|---------------|
| Default assignment | Yes | No |
| Availability at initial launch | Enabled | Disabled |
| Driver opt-in required | No | Yes |
| Charge timing | Per ride | Upfront package purchase |
| Per-ride commission | Applicable | Not applicable while package is valid |
| Wallet usage | Commission settlement only | Not used for package purchase |
| Expiry effect | Not applicable | Automatic fallback to Commission Model |

---

## 3. Functional Requirements

### 3.1 Pricing Model Summary

**Business Flow:**
```
Driver Onboarded -> Default Commission Model -> Driver views active pricing model
-> Driver may continue on Commission Model or opt into Package Model (when enabled)
-> System validates active model before each new ride acceptance
-> Package expiry or invalid condition triggers fallback to Commission Model
```

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP01  | The platform shall support exactly two driver pricing models: **Commission Model** and **Package Model**.      | Must |
| FR-DP02  | A driver shall have only one active pricing model at any given time.                                             | Must |
| FR-DP03  | Pricing model evaluation shall apply to new ride offers and new ride acceptances only; completed and active rides shall preserve their already-determined pricing outcome. | Must |
| FR-DP04  | The platform shall present pricing behavior to drivers in a simple form equivalent to **pay per ride** or **pay once and ride unlimited**. | Must |

---

### 3.2 Model Availability & Default Assignment

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP05  | By default, every newly onboarded and existing driver shall be assigned to **Commission Model**.               | Must |
| FR-DP06  | The **Package Model** feature shall be disabled at initial launch.                                              | Must |
| FR-DP07  | The system shall support enabling Package Model later through configuration without requiring redesign of driver onboarding flow. | Must |
| FR-DP08  | If Package Model is disabled by configuration, drivers shall continue to operate only under Commission Model.   | Must |
| FR-DP09  | Enabling Package Model shall not automatically migrate any driver from Commission Model; package activation shall require an explicit driver action or approved operational process. | Must |

---

### 3.3 Commission Model

**Commission Flow:**
```
Driver receives ride request -> System evaluates fare estimate
-> Platform fee is computed using commission rules
-> Driver sees fare, platform fee, and net earning before accepting
-> On ride completion, commission is settled per configured rule
```

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP10  | During the onboarding phase, the commission amount shall be **0** by default.                                   | Must |
| FR-DP11  | The onboarding-phase zero-commission rule shall be configurable so the business can later activate paid commission logic. | Must |
| FR-DP12  | The Commission Model shall support percentage-based commission calculation.                                      | Must |
| FR-DP13  | The Commission Model shall support a configurable minimum fee per ride.                                          | Must |
| FR-DP14  | The Commission Model shall support a configurable maximum fee per ride.                                          | Must |
| FR-DP15  | When commission is active, the system shall calculate per-ride platform fee using the configured percentage, subject to minimum and maximum limits. | Must |
| FR-DP16  | For every ride request shown to a driver under Commission Model, the system shall display the estimated fare, estimated platform fee, and estimated driver earning after fee. | Must |
| FR-DP17  | Commission visibility shall be shown before the driver accepts the ride request.                                | Must |
| FR-DP18  | The ride summary for Commission Model rides shall store and display the final applied commission amount and final net driver earning. | Must |

---

### 3.4 Package Model

**Package Flow:**
```
Package Model enabled by config -> Driver opens package options -> Driver selects package
-> Package purchase is completed successfully -> Package becomes active
-> New rides apply zero per-ride commission while package is valid
-> On expiry or invalid condition, pricing falls back to Commission Model
```

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP19  | Package Model shall be optional for drivers.                                                                     | Must |
| FR-DP20  | A driver shall manually opt into a package; package enrollment shall not be forced by default.                  | Must |
| FR-DP21  | The system shall support at minimum three package durations: **Daily**, **Weekly**, and **Monthly**.           | Must |
| FR-DP22  | Each package definition shall contain package name, validity period, price, and rules or conditions if applicable. | Must |
| FR-DP23  | When a valid package is active for a driver, no per-ride commission shall be charged for eligible rides during the package validity period. | Must |
| FR-DP24  | Package validity shall include a clear start time and expiry time.                                               | Must |
| FR-DP25  | When the package expires, the driver shall automatically fall back to Commission Model for new rides.           | Must |
| FR-DP26  | If multiple package offerings are available, the system shall allow the driver to select only one package activation path at a time unless future stacking rules are explicitly introduced. | Must |
| FR-DP27  | Package rules or conditions, if defined, shall be visible to the driver before purchase confirmation.          | Must |

---

### 3.5 Package Purchase & Payment Handling

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP28  | Package purchase shall be handled as a separate transaction flow from per-ride commission settlement.          | Must |
| FR-DP29  | Package amount shall not be deducted from the driver wallet.                                                     | Must |
| FR-DP30  | The driver wallet shall be used only for commission settlement under Commission Model unless future policy explicitly changes this rule. | Must |
| FR-DP31  | Successful package purchase shall activate Package Model according to the purchased package validity window.     | Must |
| FR-DP32  | Failed or incomplete package purchase shall not switch the driver away from Commission Model.                   | Must |
| FR-DP33  | Package purchase records shall remain auditable and separate from ride earning records.                         | Must |

---

### 3.6 Driver Choice & Model Transition Rules

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP34  | A driver shall be allowed to operate under either Commission Model or Package Model.                            | Must |
| FR-DP35  | Only one pricing model shall be active at any time for a driver.                                                | Must |
| FR-DP36  | A model change shall not affect any ride that is already active, accepted, or otherwise locked for execution.  | Must |
| FR-DP37  | A newly activated package shall apply to subsequent eligible rides only after activation is confirmed by the system. | Must |
| FR-DP38  | If a package expires during an active ride, that active ride shall continue under the pricing determination already applied at acceptance or trip lock time; fallback shall apply only to the next eligible ride cycle. | Must |
| FR-DP39  | If a driver does not choose a package, the driver shall continue under Commission Model without service interruption. | Must |

---

### 3.7 Transparency & Driver Communication

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP40  | The system shall clearly display the driver's currently active pricing model in the driver app.                | Must |
| FR-DP41  | The system shall clearly display applicable fees for the current ride context, if any.                         | Must |
| FR-DP42  | If the driver is subscribed to Package Model, the system shall display package validity details including active status and expiry information. | Must |
| FR-DP43  | The system shall not apply any hidden pricing deduction outside the visible and configured pricing model rules. | Must |
| FR-DP44  | Package and commission messaging shall use driver-friendly language that minimizes confusion about how charges are applied. | Must |
| FR-DP45  | Before ride acceptance, the driver shall be able to understand whether the ride is under commission deduction or package benefit. | Must |

---

### 3.8 Admin Configuration & Operational Controls

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP46  | Authorized admin roles shall be able to enable or disable Package Model through configuration.                 | Must |
| FR-DP47  | Authorized admin roles shall be able to configure commission percentage, minimum commission per ride, and maximum commission per ride. | Must |
| FR-DP48  | Authorized admin roles shall be able to define package duration, package price, package name, and package rules or conditions. | Must |
| FR-DP49  | Pricing configuration changes shall apply only to future package activations and future ride-pricing evaluations unless explicitly stated otherwise by policy. | Must |
| FR-DP50  | The system shall maintain an audit trail for driver pricing configuration changes, including actor, timestamp, old value, and new value. | Must |

---

### 3.9 Exceptions, Validation & Fallback Rules

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP51  | The system shall validate package expiry before applying Package Model benefits to a new ride.                 | Must |
| FR-DP52  | The system shall validate package eligibility conditions, if defined, before applying Package Model benefits.  | Must |
| FR-DP53  | If a package condition is not met, the system shall continue to allow normal driver operations.                | Must |
| FR-DP54  | If a package condition is not met, pricing shall fall back to Commission Model for the affected ride-pricing decision. | Must |
| FR-DP55  | If package data is unavailable, invalid, expired, or inconsistent at ride-pricing time, the system shall default safely to Commission Model rather than blocking driver operation by default. | Must |
| FR-DP56  | Fallback to Commission Model shall be logged so support and operations teams can review pricing state transitions. | Must |
| FR-DP57  | The driver shall be informed when Package Model is not applied and Commission Model fallback has been used for new rides. | Must |

---

### 3.10 Business Governance: Eligibility, Refunds, and Adjustments

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP58  | Package purchase eligibility shall be governed by business policy, including at minimum account status and compliance status checks. | Must |
| FR-DP59  | Drivers under suspension, fraud review, or mandatory compliance hold shall not be eligible for package activation until cleared. | Must |
| FR-DP60  | Package cancellation and refund policy shall be explicitly defined by business, including whether refund is allowed and under which approved reasons. | Must |
| FR-DP61  | If refund policy allows partial recovery, proration method and approval authority shall be documented and visible to support operations. | Should |
| FR-DP62  | Any goodwill or exception-based pricing adjustment shall require an auditable reason and authorized approver. | Must |
| FR-DP63  | Policy changes affecting driver charges must include an effective date and advance communication window. | Must |

---

### 3.11 End-to-End Business Scenarios

#### Scenario A: New Driver Onboarding to Commission

1. Driver is approved and activated.
2. System assigns Commission Model by default.
3. Driver sees pricing summary before first ride acceptance.
4. Zero-commission onboarding rule applies if enabled.
5. Driver completes rides with transparent pre-acceptance and post-ride pricing records.

Expected Business Outcome:

- New driver starts operations without pricing ambiguity.
- Platform monetization policy is transparent from day one.

#### Scenario B: Driver Moves from Commission to Package

1. Package Model is enabled by business configuration.
2. Driver reviews package offerings and terms.
3. Driver completes package purchase successfully.
4. Package status becomes active with clear validity window.
5. New eligible rides apply package benefit with no per-ride commission.

Expected Business Outcome:

- Driver has informed choice.
- Pricing transition is controlled, auditable, and non-disruptive.

#### Scenario C: Package Expiry and Safe Fallback

1. Active package reaches expiry.
2. Driver is notified before expiry and at expiry.
3. Next eligible ride is evaluated under Commission Model.
4. Driver continues service without downtime.

Expected Business Outcome:

- No ride flow interruption.
- Revenue policy continuity is maintained.

#### Scenario D: Package Purchase Failure

1. Driver attempts package purchase.
2. Purchase fails or remains incomplete.
3. Driver remains on Commission Model.
4. Driver receives clear message to retry or use alternate path.

Expected Business Outcome:

- No accidental unpaid package activation.
- Driver operations continue safely.

#### Scenario E: Eligibility Block and Reinstatement

1. Driver attempts package activation while on compliance hold.
2. Activation is blocked with policy reason.
3. Support resolves compliance case.
4. Driver becomes eligible and can activate package.

Expected Business Outcome:

- Compliance policy is enforced.
- Recovery path is clear for valid drivers.

---

### 3.12 Revenue Protection, Compliance, and Misuse Controls

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP64  | Business shall define and maintain anti-misuse rules for abnormal pricing benefit consumption patterns.        | Must |
| FR-DP65  | Repeated pricing disputes by the same actor shall be reviewable for possible abuse or training intervention.   | Must |
| FR-DP66  | Critical pricing policy breaches shall support escalation to Finance and Compliance operations.                 | Must |
| FR-DP67  | Pricing records used for disputes must remain retrievable for audit and regulatory review within retention policy. | Must |
| FR-DP68  | Any manual pricing override must be role-restricted and captured with reason, actor, timestamp, and approval chain. | Must |

---

### 3.13 Business Monitoring & Decision KPIs

The pricing program shall be evaluated using both operational and strategic business metrics.

| KPI Area | Minimum Metric Set |
|----------|--------------------|
| Adoption | Package opt-in rate, package renewal rate, package churn rate |
| Revenue | Effective take rate, average platform revenue per active driver, commission recovery reliability |
| Driver Health | Net earnings trend by model, earnings predictability sentiment, pricing-related support ticket rate |
| Quality | Pricing dispute rate, fallback frequency, policy exception volume |
| Governance | Unauthorized override attempts, SLA on pricing dispute closure, audit exception count |

Business Reporting Rules:

- KPI views shall be available by city/region, company/tenant, and driver segment.
- KPI trends shall be reviewed at a defined business cadence (weekly and monthly).
- Major policy decisions (price changes, package changes) shall reference KPI evidence.

---

### 3.14 Rollout, Change Management & Communication Policy

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-DP69  | Pricing policy rollout shall support phased enablement by business segment (for example city, cohort, or partner fleet). | Must |
| FR-DP70  | Business shall define rollback conditions for pricing policy rollouts if driver impact exceeds acceptable threshold. | Must |
| FR-DP71  | Driver communication for any pricing change shall be sent before policy effective date with clear impact statement. | Must |
| FR-DP72  | Support teams shall receive policy briefing and response guidance before pricing changes go live.              | Must |
| FR-DP73  | Post-change review shall be completed to confirm expected business outcomes and identify corrective actions.    | Must |

---

## 4. Technical Specification Readiness Inputs

### 4.1 Pricing States

| State | Description |
|-------|-------------|
| CommissionDefault | Driver is on Commission Model by default. |
| CommissionOnboardingFree | Driver is on Commission Model with zero commission during onboarding phase. |
| PackageDisabled | Package Model is not available for selection in the deployment. |
| PackageAvailable | Package Model is enabled for purchase and activation. |
| PackageActive | Driver has an active valid package and no per-ride commission applies for eligible rides. |
| PackageExpired | Driver's prior package validity has ended; new rides fall back to Commission Model. |
| PackageInvalidFallback | Driver had package intent or history, but package could not be applied due to invalid or unmet conditions, so Commission Model is used. |

---

### 4.2 Core Data Objects

| Object | Minimum Business Fields |
|--------|-------------------------|
| Driver Pricing Profile | Driver ID, active pricing model, pricing state, onboarding commission flag, effective-from timestamp |
| Commission Configuration | Deployment/company scope, percentage, minimum fee, maximum fee, effective-from, effective-to, status |
| Package Definition | Package ID, package name, duration type, duration value, price, active flag, rules/conditions, visibility flag |
| Driver Package Subscription | Driver ID, package ID, purchase reference, activation time, expiry time, status, fallback reason if any |
| Ride Pricing Snapshot | Ride ID, pricing model applied, estimated fare, estimated platform fee, estimated net earning, final platform fee, final net earning |
| Pricing Transition Log | Driver ID, previous model/state, new model/state, trigger reason, triggered at, actor/system source |

---

### 4.3 Configurable Business Rules

| Rule Area | Configurable Items |
|-----------|--------------------|
| Package availability | Global enable/disable flag, deployment scope, role permissions |
| Commission | Percentage, minimum fee, maximum fee, zero-commission onboarding window or eligibility |
| Package catalog | Daily/weekly/monthly offers, pricing, conditions, validity behavior |
| Visibility | Driver app labels, pre-acceptance fee messaging, package status display |
| Fallback | When to revert to Commission Model, fallback notifications, logging and audit behavior |

---

### 4.4 Key Events & Notifications

| Event | Expected Outcome |
|-------|------------------|
| Driver onboarded | Driver is assigned to Commission Model by default. |
| Package Model enabled | Eligible drivers can view and opt into packages. |
| Package purchased successfully | Driver package is activated and Package Model becomes effective for subsequent rides. |
| Package purchase failed | Driver remains on Commission Model. |
| Ride request generated | Driver sees active pricing model impact before acceptance. |
| Package expired | System falls back to Commission Model for subsequent rides and informs the driver. |
| Package condition failed | System keeps driver operational, applies Commission Model fallback, and logs the reason. |

---

## 5. Driver Pricing Non-Functional Requirements

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| NFR-DP01 | Pricing calculation for ride-request display shall be fast enough that pricing details are visible without noticeable delay in the normal matching flow. | Must |
| NFR-DP02 | Pricing rules shall be configurable without requiring mobile app re-release for normal business parameter changes. | Must |
| NFR-DP03 | Auditability shall be maintained for pricing configuration changes, package activations, and pricing fallbacks. | Must |
| NFR-DP04 | Pricing display language shall be consistent across driver app, admin portal, and support-facing operational views. | Must |
| NFR-DP05 | The pricing system shall favor safe fallback over ride blocking when package validation cannot be completed, unless a future policy explicitly introduces stricter enforcement. | Must |

---

## 6. Assumptions & Dependencies

1. Rider fare estimation already exists or will exist as a separate pricing input used for ride-request display.
2. Driver ride request cards already support showing estimated monetary values.
3. Package purchase collection may use an external payment flow or admin-assisted verification flow, but it remains separate from wallet deduction.
4. Admin configuration capability for pricing rules will be implemented in the Admin Portal or an equivalent secure configuration interface.
5. Any future tax, discount, promo, or refund behavior for packages shall be documented in supporting finance or admin BRDs without overriding the core rules in this document unless explicitly approved.

---

## 7. Business Acceptance Criteria

This BRD shall be treated as business-complete for architecture handover only when all conditions below are satisfied:

1. Both pricing models and transition rules are unambiguous for all new-ride decisions.
2. Eligibility, refund, adjustment, and override policies are approved by business owners.
3. End-to-end scenarios (onboarding, migration, expiry fallback, failure, eligibility block) are validated by Product and Operations.
4. KPI definitions and reporting ownership are confirmed.
5. Rollout and rollback governance is approved by Product, Operations, and Finance.
6. Driver communication obligations are documented with lead times.

---

## 8. Approval

| Role | Name | Status | Date |
|------|------|--------|------|
| Product Owner | TBD | Pending | TBD |
| Business Analyst | TBD | Pending | TBD |
| Engineering Lead | TBD | Pending | TBD |
| Operations Lead | TBD | Pending | TBD |

---

**Final Statement:**

The platform shall provide a flexible driver pricing system with two selectable models, **Commission Model** and **Package Model**, ensuring transparency, configurability, operational continuity, and ease of understanding for drivers while supporting future scalability.