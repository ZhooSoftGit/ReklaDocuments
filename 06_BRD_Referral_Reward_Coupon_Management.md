# Business Requirements Document (BRD) - Referral, Reward & Coupon Management

| Field | Details |
|------|---------|
| Document ID | BRD-REKLA-REFERRAL-REWARD-006 |
| Version | 1.1 |
| Status | Draft (Business Complete for Architecture Handover) |
| Date | March 25, 2026 |
| Prepared By | ZhooSoft Team |
| Parent BRDs | [01_BRD_Rekla_Ride_App.md](01_BRD_Rekla_Ride_App.md), [02_BRD_Rider_App.md](02_BRD_Rider_App.md), [03_BRD_Driver_App.md](03_BRD_Driver_App.md), [04_BRD_Admin_Portal.md](04_BRD_Admin_Portal.md) |

---

## 1. Purpose

This BRD defines complete business requirements for referral-driven rewards across rider and driver journeys, including coupon issuance, wallet credit rules, eligibility, fraud controls, lifecycle governance, and business success measurement.

The objective is to ensure reward programs are transparent, fair, auditable, scalable across white-label deployments, and ready for technical architecture design.

---

## 2. Scope

### In Scope

- Rider referral rewards issued as coupons.
- Driver referral rewards issued as wallet credits.
- Referral eligibility and qualification rules.
- Reward lifecycle (created, issued, redeemed, expired, reversed where applicable).
- Dispute and exception handling for reward issues.
- Fraud prevention and compliance controls.
- Admin governance for campaign setup and policy changes.
- Reporting KPIs and business acceptance conditions.

### Out of Scope

- Payment gateway implementation details.
- Ledger accounting implementation design.
- UI design specification.
- Tax policy design logic.

---

## 3. Business Objectives

- Increase quality user acquisition through referrals.
- Improve rider and driver retention through meaningful rewards.
- Ensure reward cost control with strong abuse prevention.
- Maintain transparency so users understand how and when rewards are earned.
- Provide complete audit visibility for support, operations, and finance.

---

## 4. Stakeholders

### External

- Riders
- Drivers

### Internal

- Product Team
- Operations Team
- Support Team
- Finance Team
- Compliance and Risk Team
- Admin Team

---

## 5. Reward Model Overview

### Rider Rewards

- Rider rewards SHALL be issued only as coupons.
- Rider wallet balance SHALL NOT be used for referral reward storage unless future policy changes this.

### Driver Rewards

- Driver referral rewards SHALL be issued as driver wallet credits.
- Wallet credits from referrals SHALL follow platform wallet usage policy for driver fee settlement.

### Core Principle

- Every reward must be traceable to a valid referral event and qualification milestone.

---

## 6. Referral Eligibility & Qualification Rules

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RC01 | A referral must be linked to a unique referrer and unique referred account. | Must |
| FR-RC02 | Self-referral must be blocked. | Must |
| FR-RC03 | Duplicate referral claims for the same referred account must be prevented. | Must |
| FR-RC04 | Referred rider qualification must require completion of defined qualifying activity (for example first completed ride). | Must |
| FR-RC05 | Referred driver qualification must require onboarding completion plus required completed rides. | Must |
| FR-RC06 | Qualification conditions must be configurable by campaign policy without business flow redesign. | Must |
| FR-RC07 | Campaign eligibility must support geography, tenant, and user-segment scoping. | Must |
| FR-RC08 | Rewards must not be issued if qualification is reversed due to fraud, cancellation abuse, or policy violation. | Must |

---

## 7. Rider Coupon Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RC09 | When rider referral qualification is met, system shall issue coupon benefit as per campaign policy to referrer and/or referee. | Must |
| FR-RC10 | Each coupon shall include at minimum coupon ID/code, user ID, campaign ID, value type, value, expiry, and status. | Must |
| FR-RC11 | Coupon value type must support fixed amount and percentage discount as defined by campaign. | Should |
| FR-RC12 | Coupon usage rules must support min ride value, eligible service types, and one-coupon-per-ride policy. | Must |
| FR-RC13 | Coupon must be validated at booking stage before confirmation. | Must |
| FR-RC14 | Coupon status shall move through lifecycle states: Issued, Active, Reserved, Used, Expired, Cancelled. | Must |
| FR-RC15 | Coupon shall be marked Used only after qualifying ride completion under policy. | Must |
| FR-RC16 | If ride is cancelled or invalidated, coupon settlement behavior (reuse or consume) shall follow campaign policy and be transparent. | Must |
| FR-RC17 | Coupon stacking with other offers shall follow defined combinability rules. | Must |

---

## 8. Driver Referral Reward Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RC18 | On driver referral qualification, reward amount shall be credited to eligible party wallet as per campaign rule. | Must |
| FR-RC19 | Wallet credit transactions shall include referral reference, campaign reference, amount, status, and timestamp. | Must |
| FR-RC20 | Reward credit must not be duplicated for the same qualification event. | Must |
| FR-RC21 | Negative wallet balances must be offset correctly by credited rewards based on wallet policy. | Must |
| FR-RC22 | Reward reversal policy must be defined for proven fraud, ineligible qualification, or policy rollback. | Must |
| FR-RC23 | Reversal actions must be auditable with approver, reason, and impact details. | Must |

---

## 9. Reward Lifecycle & State Management

Every referral and reward must follow controlled states.

### Referral Lifecycle

1. Created
2. Pending Qualification
3. Qualified
4. Rewarded
5. Rejected or Reversed

### Coupon Lifecycle

1. Issued
2. Active
3. Reserved (during checkout)
4. Used
5. Expired
6. Cancelled or Reversed

### Driver Reward Lifecycle

1. Pending Qualification
2. Qualified
3. Credit Initiated
4. Credited
5. Reversed (if approved exception)

Business Rule:

- State transitions must be deterministic, auditable, and reversible only by policy-authorized process.

---

## 10. End-to-End Business Scenarios

### Scenario A: Rider Referral Success

1. Referee signs up with referral code.
2. Referee completes qualifying ride.
3. System confirms eligibility and campaign validity.
4. Coupon(s) are issued to defined beneficiaries.
5. Rider applies coupon on next eligible booking.
6. Coupon is consumed on successful completion under policy.

Expected Outcome:

- Correct reward issuance without manual intervention.
- Full traceability from referral to coupon use.

### Scenario B: Driver Referral Success

1. Referred driver registers using referral code.
2. Driver completes onboarding and required rides.
3. System confirms qualification.
4. Wallet reward is credited.
5. Driver and support can view reward history.

Expected Outcome:

- Clear and timely driver incentive fulfillment.
- Dispute-ready audit visibility.

### Scenario C: Fraud or Policy Violation

1. Risk checks detect suspicious referral pattern.
2. Referral moves to hold/review state.
3. Reward issuance is blocked or reversed per policy.
4. User receives policy-aligned communication.

Expected Outcome:

- Abuse is controlled without breaking legitimate flows.
- Compliance and finance records remain consistent.

### Scenario D: Campaign Expiry or Rule Change

1. Campaign reaches end date or policy revision point.
2. New referrals follow updated eligibility and reward values.
3. Previously qualified rewards follow committed policy terms unless exception approved.

Expected Outcome:

- Predictable business behavior during change windows.
- No retroactive ambiguity for users and support teams.

---

## 11. Exceptions, Disputes & Operational Resolution

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RC24 | Support must be able to view referral timeline and qualification checkpoints for dispute handling. | Must |
| FR-RC25 | Missing reward disputes must have standard SLA and ownership path. | Must |
| FR-RC26 | Manual goodwill reward actions must require authorized role, reason code, and audit trail. | Must |
| FR-RC27 | Disputed coupon or wallet actions must preserve original records and add corrective entries rather than silent overwrite. | Must |
| FR-RC28 | Escalation path must exist for repeated disputes, high-value impact, or compliance-sensitive cases. | Must |

---

## 12. Fraud Prevention & Compliance Controls

System policy shall include at minimum:

- Device, phone, and account pattern checks to reduce duplicate/fake accounts.
- Velocity checks for abnormal referral creation or rapid qualification bursts.
- Geo and behavioral anomaly flags where relevant.
- Delayed payout windows where policy requires risk verification.
- Blacklist and watchlist checks for known abuse entities.

Business control rules:

- Fraud checks must not permanently block valid users without review path.
- High-risk rewards may be held until verification outcome.
- All compliance actions must be logged and reviewable.

---

## 13. Transparency & Communication Requirements

Riders must be able to see:

- Referral status (Pending, Qualified, Rewarded, Rejected).
- Available coupons and coupon expiry.
- Coupon terms and usage constraints.

Drivers must be able to see:

- Referral qualification progress.
- Wallet reward credits and reversals.
- Reward transaction history with reason/context.

Communication rules:

- Trigger notifications on referral qualification, reward issuance, expiry reminders, and rejection reasons.
- Messaging must be clear, non-technical, and policy-consistent.

---

## 14. Admin Governance & Campaign Management

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RC29 | Authorized admin roles shall create and manage referral campaigns with validity dates and reward definitions. | Must |
| FR-RC30 | Campaign rules shall support tenant-level, geography-level, and segment-level targeting. | Must |
| FR-RC31 | Campaign changes shall use effective dates and must not silently alter already-earned rewards unless policy allows. | Must |
| FR-RC32 | Critical campaign changes must require maker-checker approval workflow. | Must |
| FR-RC33 | All campaign and policy changes must maintain immutable audit history. | Must |

---

## 15. Reporting & Business KPIs

The business must monitor:

- Referral conversion rate (code shared -> qualified).
- Reward issuance count and value by campaign.
- Coupon redemption rate and expiry loss rate.
- Driver reward credit volume and reversal rate.
- Cost per acquired active rider and cost per acquired active driver.
- Fraud-block rate and false-positive review outcome.
- Support dispute rate related to referral/reward flows.

Reporting dimensions shall include:

- Tenant/company
- City/region
- Campaign
- Cohort and date range

---

## 16. Multi-Company (White-Label) Considerations

Each company/tenant should be able to configure:

- Referral qualification criteria.
- Reward values and reward types by user segment.
- Coupon validity, usage rules, and campaign windows.
- Fraud sensitivity level and hold policy.
- Communication templates and language.

Isolation rule:

- Tenant data, rewards, and policy execution must remain isolated between companies.

---

## 17. Assumptions & Dependencies

1. Rider and driver identity validation controls exist in core platform onboarding.
2. Wallet policy for driver fee settlement is governed by driver pricing and finance BRDs.
3. Support and operations teams will have role-based access to reward timelines for dispute handling.
4. Compliance and finance teams will define final thresholds for fraud holds and reversals.
5. Final legal terms for promotional campaigns will be approved by business owners.

---

## 18. Business Acceptance Criteria

This BRD is considered business-complete for architecture handover when:

1. Rider and driver reward rules are unambiguous for qualification, issuance, usage, expiry, and reversal.
2. End-to-end scenarios cover success, failure, fraud hold, and policy-change outcomes.
3. Admin governance, auditability, and approval controls are documented.
4. Dispute and escalation ownership is clearly defined.
5. KPI definitions and reporting dimensions are approved by Product, Operations, and Finance.
6. Multi-tenant policy isolation and configurability requirements are defined.

---

## 19. Final Statement

The platform shall implement a complete and governable referral-reward ecosystem where rider incentives are coupon-based and driver incentives are wallet-credit-based, with clear qualification logic, lifecycle controls, fraud safeguards, policy transparency, and measurable business outcomes.
