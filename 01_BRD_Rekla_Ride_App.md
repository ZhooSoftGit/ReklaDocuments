# Business Requirements Document (BRD) — Master
## Rekla Ride App

| Field         | Details                                                   |
|---------------|-----------------------------------------------------------|
| Document ID   | BRD-REKLA-MASTER-001                                      |
| Version       | 3.0                                                       |
| Status        | Draft                                                     |
| Date          | March 21, 2026                                            |
| Updated       | March 21, 2026 — v3.0: aligned with Rider, Driver, and Admin child BRDs |
| Prepared By   | ZhooSoft Team                                             |

---

## Document Structure

This is the parent business document for the Rekla platform. It gives business-level understanding of the product scope, deployment model, revenue model, app responsibilities, and shared rules. Detailed functional requirements are maintained in child BRDs.

| Document | Document ID | Purpose |
|----------|-------------|---------|
| This document | BRD-REKLA-MASTER-001 | Parent BRD for overall product scope, business model, app boundaries, and shared rules |
| Rider / User App BRD | BRD-REKLA-RIDER-002 | Detailed rider-facing requirements and rider technical-readiness inputs |
| Driver App BRD | BRD-REKLA-DRIVER-003 | Detailed driver onboarding, trip execution, earnings, and compliance requirements |
| Admin Portal BRD | BRD-REKLA-ADMIN-004 | Detailed operations, monitoring, configuration, support, reporting, and governance requirements |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Product Vision & White-Label Model](#2-product-vision--white-label-model)
3. [Business Objectives](#3-business-objectives)
4. [Platform Scope](#4-platform-scope)
5. [Application Scope & Responsibilities](#5-application-scope--responsibilities)
6. [Business Context & Operating Model](#6-business-context--operating-model)
	- [6.5 White-Label Delivery Flow](#65-white-label-delivery-flow)
7. [Core Platform Capabilities](#7-core-platform-capabilities)
	- [7.4 Operational Automation & AI-First Model](#74-operational-automation--ai-first-model)
8. [Shared Business Rules](#8-shared-business-rules)
9. [Stakeholders](#9-stakeholders)
10. [High-Level User Personas](#10-high-level-user-personas)
11. [Cross-Platform Non-Functional Requirements](#11-cross-platform-non-functional-requirements)
12. [Dependencies & Integrations](#12-dependencies--integrations)
13. [Assumptions & Constraints](#13-assumptions--constraints)
14. [Out of Scope](#14-out-of-scope)
15. [Success Metrics (KPIs)](#15-success-metrics-kpis)
16. [Risks](#16-risks)
17. [Referenced Child BRDs](#17-referenced-child-brds)
18. [Approval](#18-approval)

---

## 1. Executive Summary

Rekla is a ride-mobility platform consisting of three coordinated products:
- a Rider / User mobile app,
- a Driver mobile app,
- and an Admin web portal.

The platform is designed for local and regional mobility operations where reliability, operational control, and simple user experience are more important than marketplace complexity.

Rekla supports three ride models:
- Local Ride
- Package Ride
- Outstation Ride

The platform business model is based primarily on driver package sales and operational configuration, not on mandatory in-app ride-fare settlement. In the base deployment, the rider pays the driver directly using cash, direct UPI, or equivalent company-supported direct-payment method. The app may show fare reference, rules, and support information, but ride-fare settlement is not required to happen within the platform.

Rekla should also be understood as a reusable white-label product foundation that helps a client launch their own ride business in a shorter time span. The product provides the ready business structure, core modules, and configurable workflows, but each client deployment is intended to be provisioned as that client's own app environment.

This master document exists so any reader can quickly understand:
- what Rekla is,
- how the business operates,
- what each app is responsible for,
- what belongs to the parent scope versus child BRDs,
- and how the product is delivered as a white-label offering.

---

## 2. Product Vision & White-Label Model

### 2.1 Product Vision

Rekla aims to provide a dependable, easy-to-use, and operationally manageable mobility platform for companies serving city, semi-urban, and regional transport markets.

The platform is intended to create value for all three operating groups:
- Riders get a simple and trustworthy booking experience.
- Drivers get access to trip opportunities through a structured platform.
- Business owners and operations teams get real-time visibility, business controls, and support tools.

### 2.2 White-Label Product Model

Rekla is a white-label product.

This means:
- REKLA Ride is the proof/model reference product for the platform business.
- The product is used as a ready platform foundation so a client can build and launch their ride app in minimum practical time.
- The product can be delivered to different companies with company-specific branding.
- A global configuration schema is centrally maintained for all deployments; configuration keys and structure remain consistent across clients.
- Frontend/client-facing configuration is primarily for app identity and light UI behavior (for example app name, logo, theme, policy links, support contact display, and similar UI presentation settings).
- Backend business configuration controls operational behavior (for example base fare values, enabled vehicle types, enabled trip types, policy parameters, and location-specific operational rules).
- Configuration must remain runtime value-only and independent. Configuration data must not be implemented as relationally linked/FK-bound dependency tables against core operational entities.
- Only configuration values vary by client deployment; schema shape and key set are treated as product contract unless changed by platform version release.
- White-label adaptation does not mean full product redesign; core major functionality remains the Rekla baseline unless explicitly changed in signed scope.
- Optional modules can be enabled or disabled through plugin / plugout configuration.
- The client delivery is not intended to depend on one permanently shared live app/backend instance across all brands.

### 2.3 Deployment Model Clarification

The current white-label model is not a single shared live product where all customer brands run on the exact same deployed backend and app environment.

Instead, Rekla is delivered as company-specific deployment / duplication with company-level release configuration. This means the Rekla product acts as the base platform, but each client app is expected to have its own deployment setup for that brand.

The parent business document and child BRDs should therefore be read with the following assumptions:
- brand configuration is company-specific,
- each client deployment is a cloned and separately managed app/backend setup,
- each customer app deployment is separate for that customer,
- backend/services/environment provisioning may be created separately per customer deployment,
- optional features may vary by company,
- release configuration must hide disabled modules cleanly,
- and each deployed app should reflect only the approved scope of that company.

---

## 3. Business Objectives

| ID | Objective |
|----|-----------|
| BO-01 | Provide a simple and reliable mobility experience for riders across Local, Package, and Outstation booking use cases. |
| BO-02 | Provide drivers with a structured earning opportunity through platform access, ride assignment, and clear trip execution flows. |
| BO-03 | Provide business owners and operations teams with strong visibility, control, support tools, and business reporting. |
| BO-04 | Build a configurable white-label mobility product that can be adapted for different companies without re-defining the full business model each time. |
| BO-05 | Support operational launch in target markets with manageable onboarding, real-time ride monitoring, and scalable admin governance. |
| BO-06 | Increase rider retention by making the user app trustworthy, transparent, and operationally dependable enough to become the user's default ride app. |
| BO-07 | Enable monetization primarily through driver package subscription / package access models and related business plans. |
| BO-08 | Minimize manual operations by default and maximize programmatic and AI-assisted automation across onboarding, dispatch, support, monitoring, communication, and reporting workflows. |
| BO-09 | Use REKLA Ride as the baseline proof/model product and accelerate client go-live through clone-and-configure deployment strategy. |

---

## 4. Platform Scope

### 4.1 In Scope

- Rider / User mobile application for iOS and Android
- Driver mobile application for iOS and Android
- Admin web portal for operations and business management
- OTP-based authentication for rider and driver apps
- Driver onboarding and verification workflows
- Real-time ride request handling and live ride tracking
- Local, Package, and Outstation ride flows
- Admin live monitoring of ongoing rides
- Rider support, ticketing, and call-support access
- Driver support, policy alerts, and package visibility
- Admin dashboards, audit logs, reports, and quote desk workflows
- White-label brand and module configuration
- Optional plugin-based modules such as referral, loyalty points, promo management, and company-specific feature toggles
- Programmatic and AI-assisted automation features for repetitive operational tasks, with human review for high-impact decisions

### 4.2 Scope Boundaries

This parent document defines overall business boundaries and shared rules.

Detailed flows are intentionally delegated to the child BRDs:
- Rider details belong in [02_BRD_Rider_App.md](02_BRD_Rider_App.md)
- Driver details belong in [03_BRD_Driver_App.md](03_BRD_Driver_App.md)
- Admin details belong in [04_BRD_Admin_Portal.md](04_BRD_Admin_Portal.md)

---

## 5. Application Scope & Responsibilities

### 5.1 Product Boundary Summary

| Application | Primary User | Business Role |
|-------------|--------------|---------------|
| Rider / User App | Rider / Passenger | Discover service, book rides, track trip, manage profile, access support, and retain engagement |
| Driver App | Driver | Register, get approved, go online, receive trips, execute rides, and manage working profile |
| Admin Portal | Operations / Management Team | Run operations, monitor live business activity, manage users, configure business rules, support escalations, and review reports |

### 5.2 Rider / User App Responsibility

The Rider App is responsible for the customer journey.

Its business responsibilities include:
- rider onboarding through mobile OTP,
- pickup and drop selection,
- displaying ride options and reference fare,
- promo, referral, and loyalty experiences where enabled,
- booking Local / Package / Outstation rides,
- live tracking of assigned driver and active trip,
- cancellation requests within allowed policy,
- ride history, receipts, and issue reporting,
- settings, legal visibility, and support access,
- trust and retention features that keep the rider within the ecosystem.

### 5.3 Driver App Responsibility

The Driver App is responsible for the supply-side journey.

Its business responsibilities include:
- driver registration and document upload,
- admin-controlled approval and activation,
- package visibility and eligibility to receive trips,
- online / offline availability control,
- ride-category enablement,
- ride request acceptance / decline workflows,
- trip navigation and execution,
- ride-type-specific trip start and trip end validations,
- earnings visibility,
- shift history, trip history, support access, and compliance alerts.

### 5.4 Admin Portal Responsibility

The Admin Portal is the operational command center.

Its business responsibilities include:
- dashboard and real-time operational visibility,
- live ride monitoring with no manual refresh dependence,
- driver, rider, vehicle, and ride management,
- approvals and governance workflows,
- support and dispute handling,
- price, promo, and package-plan configuration,
- quote support for package and outstation requests,
- audit logging and role-based access management,
- analytics, report building, and business oversight.

### 5.5 Responsibility Matrix

| Capability | Rider App | Driver App | Admin Portal |
|-----------|-----------|------------|--------------|
| OTP authentication | Own rider login | Own driver login | Own admin authentication with company email + password + OTP |
| Booking request creation | Primary owner | Not applicable | Can assist via Book via Admin where authorized |
| Trip acceptance | View only | Primary owner | Monitor only |
| Ride execution | Track only | Primary owner | Monitor / review / intervene |
| Fare reference visibility | Primary owner | Partial visibility | Configuration and audit visibility |
| Direct ride-fare settlement | Informational only | Collection acknowledgment only | Policy / dispute visibility only |
| Driver onboarding | Not applicable | Primary owner | Approval and compliance owner |
| Rider engagement | Primary owner | Not applicable | Campaign configuration and communication owner |
| Support initiation | Rider can raise | Driver can raise | Support resolution owner |
| Reporting | Personal history only | Personal summaries only | Full business reporting owner |
| Feature configuration | Not applicable | Not applicable | Owner / manager controlled |

---

## 6. Business Context & Operating Model

### 6.1 Market Need

Many local and regional transport businesses need a mobility platform that is:
- simpler than large aggregator products,
- operationally manageable,
- suitable for local business practices,
- and flexible enough to be branded for individual companies.

### 6.2 Operating Model

Rekla works as a connected operating system for ride business operations:
- Riders request rides or scheduled trip types.
- Drivers receive eligible requests based on category, availability, package eligibility, and policy rules.
- Admin teams oversee onboarding, operations, support, exceptions, and business performance.

### 6.3 Revenue Model

The core revenue model assumed by the current BRDs is:
- drivers purchase platform packages / plans,
- platform monetization comes primarily from package sales and configured business operations,
- ride fare is collected directly from rider by driver in the base model,
- ride-fare payment gateway settlement is not mandatory for the core product model.

### 6.4 White-Label Commercial Flexibility

Per company deployment, the following may vary by signed scope:
- enabled ride types,
- brand assets and naming,
- deployment environment and service setup,
- referral reward rules,
- loyalty rules,
- promo rules,
- support contacts,
- communication templates,
- and selected optional features.

### 6.5 White-Label Delivery Flow

The following flow defines how a new client brand is delivered using the Rekla product foundation while keeping major core functionality stable.

1. **Scope and Commercial Sign-Off**
- Confirm signed scope for that client, including mandatory modules, optional modules, and exclusions.
- Confirm that white-label customization is for brand/policy/configuration and does not automatically imply major core workflow redesign.

2. **Brand and Policy Configuration**
- Apply client-specific app name, logo, theme, legal/policy links, support contact details, and communication templates.
- Configure policy values such as cancellation rules, reward rules, and support SLA settings where allowed.

3. **Business Configuration Setup**
- Configure trip types (Local/Package/Outstation) as enabled in signed scope.
- Configure vehicle types and category mappings for that client business.
- Configure base fare and related pricing inputs as approved for that deployment.

4. **Separate Client Deployment Provisioning**
- Provision separate app/backend/service environment for that client deployment.
- Apply client configuration package and secrets in that deployment environment.
- Validate that disabled modules remain hidden and inaccessible in all relevant flows.

5. **UAT, Go-Live, and Handover**
- Execute UAT with client-approved scenarios.
- Publish rider/driver apps and admin portal access for that client deployment.
- Complete operational handover and support-readiness checklist.

### 6.5.1 App-Wise Customization Scope

| App | Expected White-Label Customizations | Notes |
|-----|-------------------------------------|-------|
| Rider App | App name, logo, theme, legal/policy content, support contacts, UI labels/text assets, notifications/templates presentation | Frontend customization is UI-focused. Business behavior changes are controlled by individual backend configuration for that deployment. |
| Driver App | App name, logo, theme, legal/policy links, support contacts, UI labels/text assets, notifications/templates presentation | Frontend customization is UI-focused. Business behavior changes are controlled by individual backend configuration for that deployment. |
| Admin Portal | Portal name/branding, legal/policy links, support contacts, UI labels/text assets, dashboard/report visual presets | Frontend customization is UI-focused. Business behavior changes are controlled by individual backend configuration for that deployment. |

### 6.5.2 Backend Business Configuration Examples (Per Client)

- Base fare and pricing values
- Vehicle type list and eligibility mapping
- Trip type enablement (Local/Package/Outstation)
- Cancellation policy thresholds and fees
- Location-specific operational overrides
- Package plan rules and eligibility parameters

**Mandatory Configuration Registry Requirement:**
- The platform shall maintain a backend configuration registry that acts as the single source of truth for baseline business configuration.
- The registry shall include enum-type master values (for example ride types, booking source types, status codes, escalation levels, notification categories, and policy action codes).
- The registry shall include configurable key-value entries for deployment-specific behavior (for example pricing parameters, cancellation windows, plugin enablement, SLA settings, and alert thresholds).
- Enum catalogs and configuration values shall be versioned and auditable.
- Application services shall consume these values from backend configuration tables instead of hard-coded constants where policy/configuration behavior is expected to vary.

These are backend-managed client configurations and are not expected to be maintained as frontend config-file customizations.

**Scope Guardrail:** White-label delivery is designed to accelerate launch by reusing Rekla core capabilities. Major functionality change requests shall be treated as separate scoped enhancements and not assumed as default white-label customization.

---

## 7. Core Platform Capabilities

### 7.1 Shared Business Capability Set

| Capability Area | Platform Meaning |
|-----------------|------------------|
| Authentication | Secure OTP-based access for rider and driver; stronger admin authentication for operations users |
| Ride Discovery | User can set pickup/drop and select allowed ride type |
| Dispatch & Matching | Eligible drivers are matched based on availability, ride category, and business rules |
| Real-Time Tracking | Rider and admin can observe trip progression with live location updates |
| Trip Execution | Driver completes trip workflow according to ride-type-specific rules |
| Support & Safety | SOS, support tickets, support calling, escalation, and admin intervention support |
| Business Configuration | Pricing, packages, promos, ride categories, and policy values can be controlled by authorized admin roles |
| Reporting & Audit | Key business activities are reportable and sensitive actions are auditable |
| White-Label Control | Optional modules and company branding are configurable per deployment |

### 7.2 Ride Types Supported

| Ride Type | Business Meaning |
|-----------|------------------|
| Local Ride | Immediate or near-immediate intra-city ride request with live matching |
| Package Ride | Time / kilometer bundled ride product, often scheduled and operationally coordinated |
| Outstation Ride | Longer-distance inter-city or regional ride with scheduled confirmation model |

### 7.3 Trust and Retention Importance

The rider experience is strategically important. The product intent is not just to let a rider complete one trip, but to make the rider stay within the platform once they enter it.

Therefore, the platform prioritizes:
- clarity of fare reference,
- predictable trip status communication,
- accessible support,
- post-trip issue reporting,
- service trust through visibility,
- and low-friction reuse of the app.

### 7.4 Operational Automation & AI-First Model

Rekla is intended to operate as an automation-first business platform where repetitive manual work is reduced by programmatic rules and AI-assisted workflows.

| Automation Area | Expected Automation Capability | Human Role |
|-----------------|--------------------------------|------------|
| Driver onboarding checks | Auto-validation of document completeness, expiry checks, and profile data consistency | Manual review only for exceptions/high-risk cases |
| Ride dispatch operations | Automated matching and queue rules based on eligibility, trip type, and live status | Manual intervention for edge-case escalations |
| Support triage | AI/rule-based classification, priority tagging, and routing of tickets to correct queue | Manual handling for unresolved or sensitive tickets |
| Policy and SLA monitoring | Automated breach detection (confirmation delays, response delays, unresolved tickets) | Ops review and action closure |
| Communication workflows | Auto-campaign triggers for inactivity, expiry reminders, and operational alerts | Approval for campaign strategy and sensitive communication |
| Reporting and business insights | Scheduled report generation, anomaly alerts, trend summaries, and recommendation prompts | Management interpretation and final decisions |
| Fraud/risk flags | Rule-based and AI-assisted anomaly detection for suspicious patterns | Final investigation and enforcement by authorized roles |

**Automation Principle:** Automate-by-default for repetitive, low-risk actions; human-in-the-loop for policy-sensitive, legal, financial, and high-impact account actions.

---

## 8. Shared Business Rules

| ID | Rule |
|----|------|
| BR-01 | Rider and driver authentication shall use OTP-based verification. |
| BR-02 | Admin authentication shall use company email, password, and OTP-based second-factor verification. |
| BR-03 | A driver shall not become eligible for trip allocation until required onboarding and approval are complete. |
| BR-04 | Driver eligibility for ride allocation may be restricted by active package / plan validity according to configured business policy. |
| BR-05 | The platform shall support Local, Package, and Outstation rides, with company-level enablement rules. |
| BR-06 | In the base business model, the rider pays the driver directly; ride-fare in-app settlement is not mandatory. |
| BR-07 | Fare shown to rider may act as reference fare and must be transparent and policy-aligned. |
| BR-08 | Promo, referral, and loyalty features are optional business modules and may be enabled or disabled per company deployment. |
| BR-09 | Cancellation rules shall distinguish free cancellation window, post-accept cancellation, and post-arrival cancellation. |
| BR-10 | Cancellation through the app shall not be allowed once the trip has started. |
| BR-11 | SOS actions and active-trip safety events shall be visible to operations for intervention and audit. |
| BR-12 | Significant operational or configuration actions shall be logged through audit / activity tracking in the admin system. |
| BR-13 | Scheduled Package and Outstation rides shall follow explicit confirmation SLAs and exception communication paths. |
| BR-14 | Disabled white-label modules shall be hidden consistently in UI, notification flows, and dependent APIs. |
| BR-15 | Support resolution and communication must remain available across core deployments even if promotional modules are disabled. |
| BR-16 | Operational workflows shall be designed as automation-first, and manual execution should be used only for exception handling, approvals, or policy-required intervention. |
| BR-17 | AI-assisted decisions affecting safety, account status, pricing policy, payouts, or legal outcomes shall require traceable review controls and role-based human approval where configured. |
| BR-18 | Automated decisions and AI-generated recommendations shall be auditable with source signal, timestamp, and action trail retained for governance. |
| BR-19 | A single mobile number (phone identifier) may be associated with multiple roles (e.g., both Rider and Driver roles) within the same deployment, and the role shall be determined by the app context and user's authenticated role assignment. |
| BR-20 | Per user per role combination, only a single active refresh token shall be valid at any point in time; issuing a new refresh token shall invalidate any previously issued refresh token for that specific user-role pair. |

---

## 9. Stakeholders

| Stakeholder | Responsibility |
|-------------|----------------|
| Product Owner / Business Owner | Product direction, release priorities, commercial scope approval |
| Business Analyst | Requirement discovery, scope definition, document maintenance |
| Riders / Users | Consume ride services and provide experience feedback |
| Drivers | Provide transport service and execute rides on the platform |
| Operations Team | Manage live operations, approvals, support, and escalations |
| Admin Leadership | Governance, approvals, business controls, and performance tracking |
| Development Team | System design, implementation, deployment, and maintenance |
| QA Team | Quality assurance, scenario validation, and release confidence |
| SMS / OTP Partner | Authentication message delivery |
| Maps / Navigation Partner | Routing, location search, ETA, and mapping support |
| Communication Partner | Push, SMS, WhatsApp, and campaign delivery where used |

---

## 10. High-Level User Personas

### 10.1 Rider / User Persona

| Field | Details |
|-------|---------|
| Name | Priya |
| Profile | Daily commuter / occasional family trip planner |
| Goal | Book a safe and dependable ride quickly without confusion |
| Key Expectation | Low friction, visible driver status, transparent pricing, easy support |

### 10.2 Driver Persona

| Field | Details |
|-------|---------|
| Name | Ravi |
| Profile | Full-time or semi-full-time driver |
| Goal | Get eligible ride requests, execute trips clearly, and understand earnings / package status |
| Key Expectation | Simple app, clear trip rules, strong operational support |

### 10.3 Admin Persona

| Field | Details |
|-------|---------|
| Name | Operations Manager / Lead / Owner |
| Profile | Business operator managing service quality and growth |
| Goal | Keep operations under control, support users, and monitor business health |
| Key Expectation | Real-time visibility, clean workflows, meaningful reports, auditability |

---

## 11. Cross-Platform Non-Functional Requirements

| ID | Category | Requirement |
|----|----------|-------------|
| NFR-01 | Reliability | Core rider, driver, and admin journeys shall function consistently across supported devices and browsers. |
| NFR-02 | Performance | Rider booking flows and driver request flows shall meet operationally acceptable response times for live transport use cases. |
| NFR-03 | Availability | The platform shall support continuous business operations with high availability during business hours and peak periods. |
| NFR-04 | Security | User and operational data shall be protected in transit and at rest using industry-standard security controls. |
| NFR-05 | Privacy | Rider and driver personal information and live location shall only be exposed to roles and contexts permitted by business need and policy. |
| NFR-06 | Auditability | Sensitive operational actions, approvals, and configuration changes shall be traceable. |
| NFR-07 | Scalability | The platform design shall support onboarding of additional companies, users, and operational volume without reworking the full business model. |
| NFR-08 | Usability | Rider and driver apps shall remain simple enough for first-time users and low-complexity operational contexts. |
| NFR-09 | Configurability | White-label branding and plugin-based optional modules shall be applied consistently per company release. |
| NFR-10 | Supportability | Support, escalation, and business issue handling shall remain available even when optional modules are disabled. |
| NFR-11 | Automation | The platform shall support configurable workflow automation for repetitive operational tasks across onboarding, support routing, notifications, and reporting. |
| NFR-12 | AI Governance | AI-assisted outputs used in business operations shall support explainability metadata, confidence/context indicators where applicable, and audit trace retention. |
| NFR-13 | Human Oversight | High-impact automated actions shall support override and approval checkpoints based on role and policy configuration. |
| NFR-14 | Session & Token Management | User session tokens (access and refresh tokens) shall be managed per user-role combination, with role-based token isolation to prevent cross-role session leakage. |
| NFR-15 | Multi-Role Support | The platform shall support the same user identity (mobile number) operating under multiple roles (e.g., Rider and Driver) within a single deployment, with separate authentication contexts and session states per role. |

---

## 12. Dependencies & Integrations

| Dependency | Purpose |
|------------|---------|
| Maps / Navigation Provider | Location search, route display, ETA calculation, trip mapping |
| SMS / OTP Provider | Rider and driver mobile authentication |
| Email / OTP Service | Admin email verification and second-factor delivery |
| Push Notification Provider | Live ride alerts, support updates, approval updates, operational notifications |
| WhatsApp / SMS Communication Channel | Campaign, reminder, and support communication where configured |
| Cloud Hosting / Storage | Application hosting, media storage, operational services |
| KYC / Verification Support | Driver document verification and compliance workflows |
| Optional Payment Integration | Driver package purchase or company-specific monetization workflows where enabled |
| Optional AI/Automation Services | Ticket triage, anomaly detection, recommendation engines, and automation orchestration support where enabled |

---

## 13. Assumptions & Constraints

### 13.1 Assumptions

- Users have access to basic smartphones and mobile internet.
- Drivers can operate mobile apps with simple OTP and guided workflows.
- Admin users are trained internal staff with role-based access.
- Maps, messaging, and notification partners are available under agreed service conditions.
- Each company deployment will clearly define enabled ride types and optional modules before release.

### 13.2 Constraints

- Scope may vary per white-label company release.
- Disabled features must not appear partially in UI or workflow.
- Local business operations may require support for direct-pay collection practices.
- Low-end Android usability remains important for both rider and driver adoption in many target markets.

---

## 14. Out of Scope

The following items are not part of the current parent baseline unless separately approved in signed scope:

- Dynamic cross-brand runtime exposure on a shared backend instance
- Shared-ride / pool ride marketplace logic
- Open-ended marketplace wallet ecosystem for rider stored balance
- Full in-app settlement as mandatory core ride-fare model across all deployments
- Uncontrolled removal of core support or safety capabilities from standard deployments
- Any feature not covered by this master BRD or the approved child BRDs

---

## 15. Success Metrics (KPIs)

| KPI Area | Example Success Measure |
|----------|-------------------------|
| Rider Growth | Growth in registered and active riders |
| Rider Retention | Repeat booking rate and rider return frequency |
| Driver Supply Health | Active approved drivers and request acceptance behavior |
| Service Reliability | Booking success rate, completion rate, and support resolution timeliness |
| Operational Control | Approval turnaround time, open-ticket ageing, and live-issue response time |
| Package Business | Package sales volume, renewals, and expiry conversion performance |
| Product Quality | Crash-free sessions, support complaint rate, and app ratings |
| White-Label Readiness | Time needed to configure and release a new company deployment |
| Automation Coverage | Percentage of targeted operational workflows executed automatically without manual touch |
| Manual Work Reduction | Reduction in manual operations effort/time for onboarding, support routing, and reporting processes |
| AI Assistance Quality | Acceptance and effectiveness rate of AI/rule recommendations after human review |

---

## 16. Risks

The risks listed in this section are **business, delivery, operational, and deployment risks** that may affect launch or scale even when requirements are correct. They are **not requirement mismatches by default**. Their purpose is to make pre-launch uncertainties visible early so scope sign-off, white-label configuration, operations planning, and technical implementation can start with clear assumptions.

| ID | Category | Risk | Why It Is Listed | Likelihood | Impact | Mitigation |
|----|----------|------|------------------|------------|--------|------------|
| R-01 | Commercial / White-Label Scope | Misalignment between commercial expectation and enabled company scope | In a white-label product, business teams may assume features are included even when they are not part of the signed deployment scope. | Medium | High | Use signed white-label scope checklist and release configuration review. |
| R-02 | Rider Experience | Rider distrust due to unclear pricing, cancellation rules, or weak support | Rider retention depends on trust; unclear rules or poor support can reduce repeat usage even if booking technically works. | Medium | High | Prioritize transparent UX, visible support paths, and operational SLA ownership. |
| R-03 | Driver Operations | Driver dissatisfaction due to unclear package eligibility or trip rules | Since driver access and earnings are tied to package and trip rules, confusion here can reduce supply quality and adoption. | Medium | High | Surface package status clearly and keep trip flows policy-driven. |
| R-04 | Operations | Operational overload if admin workflows are manual or weakly structured | Live mobility operations generate approvals, support, disputes, and monitoring work that can become unmanageable without strong admin processes. | Medium | High | Use dashboards, alerts, live monitoring, reports, and role-based access controls. |
| R-05 | White-Label Configuration | Feature leakage across white-label deployments | A disabled module appearing in the wrong company build creates brand, scope, and trust issues. | Low | High | Enforce release-based plugin / plugout validation and QA checklist. |
| R-06 | Platform Reliability / Safety | Live tracking or notification failures degrade safety and trust | Real-time visibility and alerts are core to operational trust, rider safety perception, and admin intervention readiness. | Medium | High | Monitor core integrations and provide escalation fallback workflows. |
| R-07 | Market Adoption | Low market adoption in new launch regions | Even a correctly built product may underperform if local pricing, supply, branding, or expectations are not matched to the target market. | Medium | Medium | Adapt branding, pricing, package strategy, and communication per company market. |
| R-08 | Automation Quality | Over-automation or incorrect AI/rule outcomes may create operational mistakes at scale | Automation errors can propagate faster than manual errors if controls are weak. | Medium | High | Use staged rollout, confidence thresholds, human approval for high-impact actions, and continuous monitoring of automation outcomes. |

---

## 17. Referenced Child BRDs

- [02_BRD_Rider_App.md](02_BRD_Rider_App.md) — Rider / User App BRD
- [03_BRD_Driver_App.md](03_BRD_Driver_App.md) — Driver App BRD
- [04_BRD_Admin_Portal.md](04_BRD_Admin_Portal.md) — Admin Portal BRD

These child BRDs are the detailed functional source of truth for their respective applications. If this parent document and a child BRD ever appear inconsistent, the child BRD should drive detailed feature interpretation and the parent BRD should be updated accordingly.

---

## 18. Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Owner | | | |
| Business Analyst | | | |
| Tech Lead | | | |

---

*End of Document — BRD-REKLA-MASTER-001 v3.0*
