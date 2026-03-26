# Rekla End-to-End Development Checklist (EPIC Wise)

## 0. Program Readiness (Before Development)

- [ ] All BRDs reviewed and approved by Product, Ops, Support, Finance, Compliance.
- [ ] Scope freeze confirmed for current release.
- [ ] Feature toggle list finalized (plugin/plugout per tenant).
- [ ] Acceptance criteria defined for every EPIC.
- [ ] Cross-team owners assigned (Product, Backend, Mobile, Web, QA, DevOps, Support).
- [ ] Dependency map signed off (maps, SMS/OTP, notification, analytics, KYC docs).

## 1. EPIC: Core Platform Foundation

- [ ] Auth and identity flows aligned for Rider, Driver, Admin.
- [ ] Role and access control completed (especially Admin roles).
- [ ] Config framework ready (tenant-wise runtime config, no schema break).
- [ ] Audit logging enabled for all critical business actions.
- [ ] Notification event framework ready (push/SMS/email/in-app).
- [ ] Tenant isolation validated across data and configs.

## 2. EPIC: Rider App End-to-End

- [ ] OTP login and profile onboarding complete.
- [ ] Pickup/drop selection and route estimation complete.
- [ ] Local, Package, Outstation booking flows complete.
- [ ] Ride matching and trip tracking milestones complete.
- [ ] Cancellation rules and fee policy handling complete.
- [ ] Promo/referral coupon application flow complete.
- [ ] Ride history and post-trip summary complete.
- [ ] Rating and feedback submission flow complete.
- [ ] SOS and support ticket creation from app complete.

## 2A. EPIC: Ride Booking Dispatch Resilience & Demand-Supply Controls

- [ ] No-driver-found fallback flow complete (graceful user messaging + next best options).
- [ ] Automated rematch strategy complete (retry rules, retry window, and candidate refresh logic).
- [ ] Dispatch queue timeout policy complete (max wait thresholds and expiry handling).
- [ ] Hot-zone controls complete (dynamic supply balancing, zone-level priority tuning).
- [ ] Peak-load protection complete (throttling, staged matching, and overload guardrails).
- [ ] Booking journey behavior complete for high-demand scenarios (wait, queue, retry, or fail-safe exit).
- [ ] Rider communication templates complete for delayed assignment / no driver situations.
- [ ] Admin operational controls complete for live dispatch tuning during demand spikes.
- [ ] Dispatch health KPIs complete (match rate, time-to-assign, no-driver rate, rematch success rate).

## 2B. EPIC: Scheduled Ride Reliability Controls (Package/Outstation)

- [ ] Pre-assignment controls complete (driver allocation window, lock rules, reassignment triggers).
- [ ] Scheduled-ride reconfirmation complete (rider + driver reconfirmation checkpoints before pickup).
- [ ] No-show handling policy complete (driver no-show, rider no-show, wait-time and penalty rules).
- [ ] Backup driver assignment complete (auto/manual fallback when primary driver drops or is unavailable).
- [ ] SLA breach playbook complete for scheduled rides (detection, escalation, user communication, recovery actions).
- [ ] Scheduled ride lifecycle completeness validated end-to-end (booked -> assigned -> reconfirmed -> started -> completed/closed).
- [ ] Scheduled operations dashboard complete (upcoming rides, at-risk rides, breach risk alerts).
- [ ] Support and escalation runbook complete for package/outstation booking failures.
- [ ] Reliability KPIs complete (pre-assignment success rate, reconfirmation success rate, no-show rate, backup assignment success, SLA adherence).

## 2C. EPIC: Serviceability & Geo-Governance Controls

- [ ] City and zone geofencing complete (serviceable boundaries by tenant and ride type).
- [ ] Pickup and drop validation complete against allowed service zones before booking confirmation.
- [ ] Restricted-area policy complete (no-book zones, no-pickup zones, no-drop zones, policy-based exceptions).
- [ ] Out-of-service behavior complete (block booking, show clear in-app message, provide nearest supported alternative).
- [ ] Serviceability communication complete (inform rider when pickup/drop is outside supported area, with reason and next action).
- [ ] Route-crossing policy complete (if ride enters restricted/out-of-service zones during trip lifecycle).
- [ ] Driver availability filtering complete by city/zone permissions and compliance eligibility.
- [ ] Location spoofing/tamper detection complete (mock location, abnormal jumps, impossible speed patterns).
- [ ] Tamper response flow complete (risk flag, controlled fallback, support/compliance escalation when needed).
- [ ] Geo-governance KPIs complete (out-of-service request rate, blocked booking rate, spoof detection count, false-positive review rate).

## 3. EPIC: Driver App End-to-End

- [ ] Registration, KYC docs, and admin approval lifecycle complete.
- [ ] Online/offline, engagement lock, and request timeout rules complete.
- [ ] Local, Package, Outstation trip execution flows complete.
- [ ] Navigation and trip state validations complete.
- [ ] Earnings, commission, and package status visibility complete.
- [ ] Driver pricing model switching and fallback behavior complete.
- [ ] Ride/shift history and policy alerts complete.
- [ ] Driver rating visibility and feedback handling complete.
- [ ] Driver support issue raising flow complete.

## 4. EPIC: Admin Portal End-to-End

- [ ] Secure admin auth with role-based access complete.
- [ ] Dashboard KPIs and live operations monitoring complete.
- [ ] Driver, rider, vehicle, ride management modules complete.
- [ ] Approval workflows (driver/docs/critical actions) complete.
- [ ] Fare/pricing and package configuration controls complete.
- [ ] Promo/referral campaign management complete.
- [ ] Support/dispute management and escalation controls complete.
- [ ] Audit logs, reports, and analytics module complete.
- [ ] Quote desk and business operations utilities complete.

## 5. EPIC: Driver Pricing Model

- [ ] Commission model rules complete (percentage, min/max, visibility).
- [ ] Package model rules complete (daily/weekly/monthly, eligibility, validity).
- [ ] Purchase success/failure and activation handling complete.
- [ ] Auto-fallback to commission on package expiry/invalidity complete.
- [ ] Transition rules for active rides vs next rides complete.
- [ ] Pricing communication and transparency checks complete.
- [ ] Pricing exceptions, adjustments, and approval governance complete.

## 6. EPIC: Referral, Reward, Coupon

- [ ] Referral eligibility and anti-self-referral rules complete.
- [ ] Rider coupon issuance and lifecycle states complete.
- [ ] Driver wallet referral credit and reversal policy complete.
- [ ] Campaign-level rule config by tenant/region/segment complete.
- [ ] Dispute handling for missing rewards complete.
- [ ] Fraud checks and hold/review workflow complete.
- [ ] Reward reporting and KPI tracking complete.

## 7. EPIC: Rating & Feedback

- [ ] Post-ride rating eligibility and submission window complete.
- [ ] Rider-to-driver and optional driver-to-rider ratings complete.
- [ ] Feedback reason taxonomy and free-text moderation complete.
- [ ] Low-rating threshold triggers and intervention workflow complete.
- [ ] Dispute/moderation workflow and SLA complete.
- [ ] Safety-sensitive feedback escalation to compliance complete.
- [ ] Rating KPIs and reporting views complete.

## 8. EPIC: Unified Support Ticketing

- [ ] Ticket intake from Rider, Driver, and Internal teams complete.
- [ ] Ticket classification and priority-SLA mapping complete.
- [ ] Ownership assignment and lifecycle state transitions complete.
- [ ] Escalation matrix (L1-L4) and triggers complete.
- [ ] Parent-child ticket linking for cross-team dependencies complete.
- [ ] Notification rules for status/SLA risk complete.
- [ ] Resolution, closure, reopen, and duplicate merge policies complete.
- [ ] Support metrics and quality feedback loop complete.

## 9. EPIC: Driver Settlement and Payout Operations (Driver-Only Monetization)

- [ ] Business boundary locked: rider pays driver directly; platform does not collect ride fare from riders.
- [ ] Driver monetization model confirmed: commission and/or package collections from drivers only.
- [ ] Driver wallet ledger rules finalized (dues, credits, adjustments, reversals).
- [ ] Driver settlement cycle defined (daily/weekly cut-off, posting, and closure rules).
- [ ] Payout eligibility rules for drivers defined (hold, release, compliance flags, dispute holds).
- [ ] Failed payout handling flow complete (retry, alternate account, manual review queue).
- [ ] Settlement dispute workflow complete (owner, SLA, evidence, resolution path).
- [ ] Finance reconciliation complete across rides, pricing, package sales, wallet, and adjustments.
- [ ] Manual financial override controls complete (role, approval, full audit trail).
- [ ] Driver-facing settlement/payout statement visibility complete (history, status, reasons).

## 10. EPIC: Security, Compliance, and Risk

- [ ] PII/privacy controls applied to all user and driver data.
- [ ] Role-based access and action authorization enforced.
- [ ] Fraud detection signals integrated for referrals, ratings, and support abuse.
- [ ] Compliance review path for safety/fraud incidents complete.
- [ ] Data retention and audit retrieval policy implemented.

## 11. EPIC: Quality Engineering and UAT

- [ ] Feature-wise test scenarios mapped to BRD acceptance criteria.
- [ ] End-to-end scenario tests completed across Rider-Driver-Admin.
- [ ] Cross-feature regression suite completed.
- [ ] SLA and operational alert scenarios validated.
- [ ] Multi-tenant configuration tests completed.
- [ ] UAT sign-off from Product, Ops, Support, Finance completed.

## 12. EPIC: Release Readiness

- [ ] Production config checklist completed (tenant-wise).
- [ ] Rollout plan and rollback plan approved.
- [ ] Monitoring dashboards and alerts active.
- [ ] Support runbook and escalation contacts published.
- [ ] Training completed for Support/Ops/Admin users.
- [ ] Legal/policy pages and in-app communications finalized.
- [ ] Go/No-Go meeting completed with all owners.

## 13. EPIC: Post-Release Stabilization

- [ ] Hypercare team active for first release window.
- [ ] P0/P1 incident workflow active and tested.
- [ ] Daily KPI review cadence active (rides, pricing, support, ratings, rewards).
- [ ] Defect triage and patch release process active.
- [ ] Customer and driver feedback loop integrated into backlog.
- [ ] Release retrospective and improvement plan documented.

## 14. Mandatory Exit Gates (Do Not Skip)

- [ ] No open blocker for booking, assignment, trip execution, or support closure.
- [ ] No unresolved financial reconciliation blocker.
- [ ] No unresolved compliance blocker for safety/fraud handling.
- [ ] No unresolved tenant isolation blocker.
- [ ] All EPIC owners have provided release sign-off.
