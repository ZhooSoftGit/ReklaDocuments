# Business Requirements Document (BRD) — Rider App
## Rekla Ride App

| Field         | Details                                          |
|---------------|--------------------------------------------------|
| Document ID   | BRD-REKLA-RIDER-002                              |
| Version       | 1.2                                              |
| Status        | Draft                                            |
| Date          | March 21, 2026                                   |
| Prepared By   | ZhooSoft Team                                    |
| Parent BRD    | [BRD-REKLA-MASTER-001](01_BRD_Rekla_Ride_App.md) |

---

## Table of Contents

1. [Purpose & Scope](#1-purpose--scope)
2. [Rider Persona](#2-rider-persona)
3. [Functional Requirements](#3-functional-requirements)
   - [3.1 Registration & Authentication](#31-registration--authentication)
   - [3.2 Home Screen](#32-home-screen)
   - [3.3 Pickup & Drop Location Selection](#33-pickup--drop-location-selection)
   - [3.4 Ride Options — Distance, ETA & Estimated Fare](#34-ride-options--distance-eta--estimated-fare)
   - [3.5 Promo Code & Discounts](#35-promo-code--discounts)
   - [3.6 Ride Confirmation & Trip Flow](#36-ride-confirmation--trip-flow)
   - [3.7 Ride Cancellation](#37-ride-cancellation)
   - [3.8 Profile Management](#38-profile-management)
   - [3.9 Ride History](#39-ride-history)
   - [3.10 Referral Program](#310-referral-program)
   - [3.11 Loyalty Points Program (Plugin)](#311-loyalty-points-program-plugin)
   - [3.12 Settings Management](#312-settings-management)
   - [3.13 Support & Helpdesk](#313-support--helpdesk)
   - [3.14 White-Label Feature Plugin Controls](#314-white-label-feature-plugin-controls)
   - [3.15 User Trust, Retention & Experience Guardrails](#315-user-trust-retention--experience-guardrails)
4. [Technical Specification Readiness Inputs](#4-technical-specification-readiness-inputs)
   - [4.1 Rider Ride Lifecycle States](#41-rider-ride-lifecycle-states)
   - [4.2 Core Rider-Facing Data Objects](#42-core-rider-facing-data-objects)
   - [4.3 Notification & Event Triggers](#43-notification--event-triggers)
   - [4.4 Configurable Business Rules](#44-configurable-business-rules)
5. [Rider-Specific Non-Functional Requirements](#5-rider-specific-non-functional-requirements)
6. [Approval](#6-approval)

---

## 1. Purpose & Scope

This document defines detailed functional requirements for the **Rekla Rider mobile application** (iOS & Android). It covers all screens, user interactions, and system behaviours the rider experiences — from registration through to post-trip actions.

**In Scope:** All features of the Rider App for v1.2 (MVP).
**Business Payment Model Note:** Ride fare is collected directly by driver from rider (cash/direct UPI or equivalent) in the base model; the rider app may show reference fare and support communication, but platform ride-fare settlement is not mandatory in base white-label deployment.
**White-Label Product Note:** Rekla is delivered as company-specific app duplication (per-company configuration) with separate deployment setup for each company. Feature modules shall support plugin (enable) / plugout (disable) by company specification.
**White-Label Scope Boundary Note:** Frontend white-label customization is intended mainly for app identity and light UI behavior (such as app name, logo, theme, legal/policy content, support contact details, and UI text assets). Business behavior changes (for example base fare, vehicle types, trip-type enablement, and location-based business rules) are driven by individual backend configuration for that deployment. Core rider-flow functionality remains the Rekla baseline unless explicitly modified in signed scope.
**Configuration Independence Note:** Configuration is runtime value-driven only. Configuration records must remain independent and must not be FK-linked to core transactional/domain tables.
**Out of Scope:** Driver app, Admin portal. Refer to parent BRD for platform-wide exclusions.

---

## 2. Rider Persona

| Field       | Details                                              |
|-------------|------------------------------------------------------|
| Name        | Priya, 28, Office Professional                       |
| Goal        | Book a safe, affordable ride to work quickly         |
| Pain Points | Long waiting times, surge pricing, no real-time ETA  |
| Device      | Android smartphone (mid-range)                       |
| Tech Level  | Moderate                                             |
| Connectivity| 4G mobile data, occasional Wi-Fi                     |

---

## 3. Functional Requirements

### 3.1 Registration & Authentication

**User Journey:**
```
App Launch → Enter Mobile Number → Receive OTP → Verify OTP → (First Time) Enter Name → Account Created → Home Screen
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R01  | The app shall display a mobile number entry screen as the first screen on fresh launch.        | Must     |
| FR-R02  | The system shall send a one-time password (OTP) to the entered mobile number via SMS.          | Must     |
| FR-R03  | The user shall enter the OTP on a verification screen to complete authentication.              | Must     |
| FR-R04  | OTP shall be valid for 5 minutes; the user shall be allowed to resend OTP after 30 seconds.   | Must     |
| FR-R05  | On first-time login, the user shall be prompted to enter their name to complete profile setup. | Must     |
| FR-R06  | On successful OTP verification, the system shall create a new account (if new user) or log in the existing user. | Must |
| FR-R07  | The session shall persist so the user is not required to log in again on subsequent app opens. | Must     |
| FR-R08  | The user shall be able to log out from the Profile section, which clears the local session.    | Must     |

**Acceptance Criteria:**
- A new user who enters a valid mobile number receives an OTP within 30 seconds.
- Entering a correct OTP navigates the user to the name entry screen (first time) or Home Screen (returning user).
- An incorrect OTP shows an inline error; the user may retry up to 3 times before a cooldown applies.
- A logged-out user who reopens the app lands on the mobile number entry screen.

---

### 3.2 Home Screen

**User Journey:**
```
Authentication Complete → Home Screen (Map View) → Entry point for: Book Ride / Profile / History / Referral
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R09  | After successful authentication, the app shall navigate to the Home Screen.                   | Must     |
| FR-R10  | The Home Screen shall display a full-screen interactive map centered on the user's current location. | Must |
| FR-R11  | The app shall request location permission on first launch; if denied, the user shall be prompted to enter location manually. | Must |
| FR-R12  | The Home Screen shall display a **"Where are you going?"** search bar / bottom sheet to initiate ride booking. | Must |
| FR-R13  | The Home Screen shall display a navigation menu (drawer or bottom navigation) providing access to: Profile, Ride History, and Referral sections. | Must |
| FR-R14  | The Home Screen shall display the user's first name and profile photo (or initials avatar) in the header/menu. | Should |
| FR-R15  | If the user has an active ongoing trip, the Home Screen shall directly redirect to the live trip tracking view. | Must |
| FR-R15A | The app shall request notification permission to send ride updates, driver arrival alerts, and support updates; if denied, the user shall see guidance to enable notifications from settings. | Must |
| FR-R15B | The app shall request required device permissions contextually (location, notifications, phone-call intent, media upload when needed) with clear purpose text. | Must |

---

### 3.3 Pickup & Drop Location Selection

**User Journey:**
```
Tap "Where are you going?" → (Auto) Set Pickup → Enter Drop Location → Confirm Both Locations
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R16  | The system shall auto-detect the user's current GPS location and set it as the default pickup point. | Must  |
| FR-R17  | The user shall be able to manually adjust the pickup location by dragging the map pin or typing in the search box. | Must |
| FR-R18  | The user shall be able to search and select a drop location using a text search with auto-complete suggestions powered by the maps API. | Must |
| FR-R19  | The system shall display up to 5 recently used locations for quick selection of pickup or drop points. | Should |
| FR-R20  | The user shall be able to save and label frequently used addresses (e.g., Home, Work, Other) and select them from a shortcuts list. | Should |
| FR-R21  | Both pickup and drop locations shall be displayed as distinctly labeled pins on the map once selected. | Must |
| FR-R22  | The user shall be able to swap pickup and drop locations with a single tap action.             | Should   |

---

### 3.4 Ride Options — Distance, ETA & Estimated Fare

**User Journey:**
```
Locations Confirmed → Route calculated → Ride options displayed (per type: ETA + Fare) → User selects option + Payment method
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R23  | After locations are confirmed, the system shall calculate and display the estimated route on the map with a polyline. | Must |
| FR-R24  | The system shall display the **total trip distance** (in km) for the calculated route.         | Must     |
| FR-R25  | The system shall display ride type options as **Local Ride**, **Package Ride**, and **Outstation Ride** (as enabled by company config). | Must |
| FR-R26  | Each ride option card shall display: ride type name, vehicle icon, ETA to pickup (minutes), and estimated fare (₹). | Must |
| FR-R27  | Estimated fare shall be calculated as: Base Fare + (Distance × Per Km Rate) + (Duration × Per Min Rate) + applicable taxes. | Must |
| FR-R28  | The ETA per ride option shall reflect the estimated time for the nearest available driver of that category to reach the pickup point. | Must |
| FR-R29  | If surge pricing is active for a ride category, the card shall clearly display the surge multiplier and the adjusted fare. | Must |
| FR-R30  | The user shall be able to select one ride option before proceeding; the selected card shall be visually highlighted. | Must |
| FR-R31  | The user shall be able to select preferred collection mode for direct payment to driver (e.g., cash/direct UPI) where supported by company policy. | Should |

---

### 3.5 Promo Code & Discounts

**User Journey:**
```
Ride Options Screen → Tap "Apply Promo Code" → Enter / Select Code → Validated → Discount Reflected → Proceed to Booking
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R32  | The Ride Options screen shall display an **"Apply Promo Code"** entry field or call-to-action alongside the fare summary. | Must |
| FR-R33  | The user shall be able to type a promo code manually or select from a list of available eligible codes shown when the field is tapped. | Must |
| FR-R34  | The system shall validate the entered promo code in real time and display an inline success message (discount amount) or a descriptive error. | Must |
| FR-R35  | On successful validation, the discount amount shall be deducted from the estimated fare and the updated fare displayed immediately. | Must |
| FR-R36  | The fare breakdown shall itemize: original fare, discount amount (labeled with the code), and final payable amount. | Must |
| FR-R37  | If the promo code is invalid, expired, already used, or not applicable to the selected ride type, a clear and descriptive error message shall be shown. | Must |
| FR-R38  | The user shall be able to remove an applied promo code; the fare shall revert to the original amount. | Must |
| FR-R39  | The system shall surface promo codes the user is eligible for when the promo field is tapped.  | Should   |
| FR-R40  | Only one promo code shall be applicable per trip at a time.                                    | Must     |

---

### 3.6 Ride Confirmation & Trip Flow

**User Journey:**
```
Select Option + Payment + (Optional) Promo → "Book Ride" → Searching → Driver Assigned → Driver En Route → Arrived → Trip Started → Trip Completed → Rate Driver
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R41  | The user shall confirm the booking by tapping a prominent **"Book Ride"** CTA button.         | Must     |
| FR-R42  | The system shall display a searching / matching animation while finding an available driver.   | Must     |
| FR-R43  | Once a driver is assigned, the app shall display driver details: name, photo, vehicle model, vehicle registration number, and average rating. | Must |
| FR-R44  | The app shall display the driver's real-time location on the map and continuously update the dynamic ETA to pickup. | Must |
| FR-R45  | The rider shall receive push notifications at each trip milestone: Driver Assigned, Driver Arriving, Trip Started, Trip Completed. | Must |
| FR-R46  | The rider shall be able to contact the driver via a masked phone call from within the app.     | Should   |
| FR-R47  | On trip completion, the app shall display a trip summary: distance, time, fare/reference-fare breakdown (including any promo/points discount if used), and final payable reference. | Must |
| FR-R48  | The rider shall be prompted to rate the driver (1–5 stars) and optionally leave a text comment after the trip summary screen. | Must |
| FR-R49  | The rider shall have access to a clearly visible **SOS / Emergency** button throughout the trip, which upon activation sends the rider's live location to a pre-registered emergency contact and the ops team. | Must |
| FR-R49A | For **Local Ride**, the booking confirmation (driver assigned or explicit no-driver result) shall be provided within a maximum of 1 to 2 minutes from booking submission. | Must |
| FR-R49B | For **Package Ride**, the rider shall receive booking confirmation no later than 4 hours before scheduled pickup time. | Must |
| FR-R49C | For **Outstation Ride**, the rider shall receive booking confirmation no later than 4 hours before scheduled pickup time. | Must |
| FR-R49D | If a scheduled Package/Outstation booking cannot be confirmed within SLA, the app shall notify rider with status and support assistance options. | Must |
| FR-R49E | If a scheduled Package/Outstation booking is created within 4 hours of pickup, the rider shall receive confirmation or explicit unavailability within 15 minutes of booking submission. | Must |

---

### 3.7 Ride Cancellation

**User Journey:**
```
Active Booking → Tap "Cancel Ride" → Select Reason → Cancellation Fee Warning → Confirm → Cancel Processed → Notification + Fee/Policy Summary
```

> **Cancellation is only permitted before the driver starts the trip. Once the trip is started, the Cancel option is removed.**

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R50  | The rider shall see a clearly visible **"Cancel Ride"** button on the active booking / tracking screen at all times before the trip starts. | Must |
| FR-R51  | On tapping Cancel, the app shall prompt the rider to select a cancellation reason from a predefined list: "Driver taking too long", "Wrong vehicle", "Change of plans", "Booked by mistake", "Other". | Must |
| FR-R52  | If the rider cancels **within the free cancellation window** (default: 2 minutes after booking, before driver accepts — configurable by admin), no fee shall be charged. | Must |
| FR-R53  | If the rider cancels **after the driver has accepted and is en route**, a standard cancellation fee shall be assessed. | Must |
| FR-R54  | If the rider cancels **after the driver has arrived** at the pickup point, a higher cancellation fee shall apply. | Must |
| FR-R55  | Before the rider confirms cancellation, the app shall clearly display whether a cancellation fee will be charged and the fee amount. | Must |
| FR-R56  | On confirmed cancellation, the rider shall receive a push notification and in-app confirmation with the fee charged (if any). | Must |
| FR-R57  | As ride fare is collected directly by driver in base model, cancellation handling shall show applicable cancellation fee/reference policy and log dispute options instead of payment-gateway refund flow. | Must |
| FR-R58  | The cancellation event, selected reason, timestamp, and fee charged shall be logged in the trip record and visible to the admin. | Must |
| FR-R59  | Once the driver has tapped "Start Trip", the **"Cancel Ride"** button shall be removed from the UI and cancellation shall not be possible through the app. | Must |
| FR-R59A | The rider shall be able to cancel scheduled **Package** and **Outstation** bookings before trip start, with clear display of cancellation window and applicable fee/reference policy. | Must |

---

### 3.8 Profile Management

**User Journey:**
```
Home Screen → Menu → Profile → View / Edit → Save Changes
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R60  | The user shall be able to access the Profile section from the Home Screen navigation menu.    | Must     |
| FR-R61  | The Profile screen shall display: full name, mobile number (read-only), profile photo, and email address (optional). | Must |
| FR-R62  | The user shall be able to update their display name and upload or change their profile photo.  | Must     |
| FR-R63  | The user shall be able to add or update an email address; email shall be validated for format. | Should   |
| FR-R64  | The mobile number shall be the primary account identifier and shall not be editable without OTP re-verification. | Must |
| FR-R65  | The user shall be able to manage saved addresses (Home, Work, and custom-labeled) from the Profile section. | Should |
| FR-R66  | The user shall be able to view and manage preferred ride preferences (e.g., saved contacts, communication preferences, direct-pay collection preference) from Profile/Settings. | Should |
| FR-R67  | The user shall be able to log out from the Profile section; this shall clear the local session and return the user to the mobile number entry screen. | Must |

---

### 3.9 Ride History

**User Journey:**
```
Home Screen → Menu → Ride History → Trip List (most recent first) → Tap Trip → Trip Detail
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R68  | The user shall be able to access Ride History from the Home Screen navigation menu.           | Must     |
| FR-R69  | The Ride History screen shall display a chronological list of all past trips, most recent first. | Must   |
| FR-R70  | Each trip entry in the list shall show: date, time, pickup location name, drop location name, fare paid, and trip status (Completed / Cancelled). | Must |
| FR-R71  | The user shall be able to tap any trip entry to open the full Trip Detail screen.             | Must     |
| FR-R72  | The Trip Detail screen shall display: route map snapshot, pickup and drop addresses, distance travelled, trip duration, driver name, vehicle details, fare/reference-fare breakdown (including promo/points discounts if applied), cancellation fee (if applicable), and collection mode reference. | Must |
| FR-R73  | The user shall be able to download or share a trip receipt (PDF or shareable text summary) from the Trip Detail screen. | Should |
| FR-R74  | The user shall be able to re-book a previous trip (using the same pickup and drop) directly from the Trip Detail screen with a single tap. | Could |

---

### 3.10 Referral Program

**User Journey:**
```
Home Screen → Menu → Refer & Earn → View Code → Copy / Share → Friend Registers with Code → Friend Completes First Trip → Rewards Credited to Both
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R75  | Every registered user shall have a unique alphanumeric referral code generated automatically upon account creation. | Must |
| FR-R76  | The user shall be able to access the "Refer & Earn" section from the Home Screen navigation menu. | Must  |
| FR-R77  | The Referral screen shall display the user's unique referral code prominently with a one-tap **"Copy Code"** button. | Must |
| FR-R78  | The user shall be able to share the referral code via the device native share sheet (WhatsApp, SMS, Email, etc.). | Must |
| FR-R79  | During new user registration, the app shall display an optional **"Enter Referral Code"** field. | Must  |
| FR-R80  | When the referred user completes their first trip after registering with a valid referral code, the referrer shall receive **₹25** referral reward (default), configurable by admin/company policy. | Must |
| FR-R81  | The Referral screen shall display a summary dashboard: total invites sent, successful conversions (first trip completed), and total rewards earned. | Should |
| FR-R82  | Referral rewards shall be credited as configured by company policy (e.g., wallet credit, promo value, or points), with transparent reward ledger in app. | Must |

---

### 3.11 Loyalty Points Program (Plugin)

**Concept:** Loyalty points is a configurable plugin feature. When enabled, riders earn points per completed ride and can redeem points as ride discount/free amount based on company rules.

**User Journey:**
```
Complete Ride -> Points Credited -> View Points Ledger -> Redeem Points During Next Booking
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R83  | The app shall support a **Loyalty Points** feature as plugin mode (enable/disable by company config). | Must |
| FR-R84  | When loyalty points is enabled, every completed ride shall add points at default rule: **1 point per ₹100** of ride value (rounded as per configured rule). | Must |
| FR-R85  | The rider shall be able to view total points balance and a transaction ledger (earned/redeemed/expired) in a dedicated section. | Must |
| FR-R86  | The rider shall be able to redeem eligible points during booking; redeemed value shall be reflected in fare/reference-fare breakdown. | Must |
| FR-R87  | The points earning ratio, redemption conversion, minimum redemption threshold, and expiry period shall be configurable by admin/company policy. | Must |
| FR-R88  | If loyalty points plugin is disabled for a company, all points UI and redemption flows shall be hidden gracefully. | Must |

---

### 3.12 Settings Management

**User Journey:**
```
Home/Menu -> Settings -> Manage Preferences -> Save
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R89  | The rider shall be able to access a dedicated Settings screen from app navigation.            | Must     |
| FR-R90  | The rider shall be able to manage notification preferences (ride updates, offers, support updates). | Must |
| FR-R91  | The rider shall be able to manage location-permission guidance and re-enable flow from settings when location is denied. | Should |
| FR-R92  | The rider shall be able to manage language preference (from configured language set) and app shall apply changes immediately or on next launch. | Should |
| FR-R93  | The rider shall be able to view legal links (Terms, Privacy, Refund/Cancellation policy) from Settings. | Must |

---

### 3.13 Support & Helpdesk

**User Journey:**
```
Menu -> Help & Support -> Create Ticket / Call Support -> Track Status -> Receive Resolution
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R94  | The rider shall be able to raise a support ticket from within the app by selecting issue type and entering description. | Must |
| FR-R95  | The rider shall be able to attach relevant ride reference while raising ticket (current or past trip). | Must |
| FR-R96  | The rider shall be able to view ticket status timeline (Open/In Progress/Resolved/Closed) and resolution messages. | Must |
| FR-R97  | The rider app shall provide a clearly visible **Call Support** action with configured support number and masked calling where required by policy. | Must |
| FR-R98  | For unresolved urgent issues (e.g., active trip safety concern), the app shall provide priority escalation guidance to rider. | Should |

---

### 3.14 White-Label Feature Plugin Controls

**Concept:** Rekla Rider App is delivered per-company as white-label duplication. Feature modules shall be configurable at build/release policy level per company without shared cross-brand runtime dependency.

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-R99  | The rider app feature set shall support company-level plugin/plugout controls for optional modules, including at minimum: Referral Program, Loyalty Points, Promo Module, and specific ride categories. | Must |
| FR-R100 | When a module is disabled for a company, related UI entries, APIs, and notifications shall be hidden/disabled consistently without broken navigation. | Must |
| FR-R101 | Company-specific defaults (brand text, rewards amounts, points rules, ride type enablement, support contact details) shall be configurable through release configuration. | Must |
| FR-R102 | Core mandatory modules (OTP login, Home, booking flow, ride tracking, cancellation, ride history, profile, support access) shall remain available across all white-label deployments unless explicitly excluded in signed scope. | Must |

---

### 3.15 User Trust, Retention & Experience Guardrails

**Concept:** Riders should feel this app is dependable, transparent, and safe enough to remain their default mobility app. These requirements define trust signals and recovery paths that reduce churn.

| ID       | Requirement                                                                                    | Priority |
|----------|------------------------------------------------------------------------------------------------|----------|
| FR-R103  | Before final booking confirmation, the app shall show a clear fare-reference breakdown with all components (base, distance, time, surge, discounts, cancellation policy note) and no hidden line items. | Must |
| FR-R104  | If no driver is found, the app shall show an explicit result with retry options (same ride type, alternate ride type, or schedule later) and a support entry point. | Must |
| FR-R105  | During active trips, riders shall be able to share live trip status (driver name, vehicle, and live location link/token) with trusted contacts. | Should |
| FR-R106  | After trip completion, riders shall be able to report ride issues within 72 hours from trip history with issue category and evidence attachment support. | Must |
| FR-R107  | The app shall provide a transparent trip ledger for each rider showing completed rides, cancelled rides, promo usage, referral rewards, and points transactions (if enabled). | Must |
| FR-R108  | The app shall support account deletion/deactivation request flow with OTP verification and clear messaging about data-retention policy as per company/legal configuration. | Must |
| FR-R109  | The app shall provide language and accessibility-ready UI labels for key actions (Book, Cancel, Call Support, SOS) to reduce first-time user confusion. | Should |

---

## 4. Technical Specification Readiness Inputs

This section does not replace the technical specification document. It defines the minimum implementation anchors that should be carried into the technical specification so engineering, QA, and product use the same lifecycle, data, event, and configuration language.

### 4.1 Rider Ride Lifecycle States

| State | Description | Rider Visibility / Expected UI |
|-------|-------------|--------------------------------|
| Draft | Rider has opened booking flow but has not submitted request yet. | Pickup/drop, ride type, promo, and payment preference are editable. |
| Searching | Booking request submitted; system is trying to assign driver. | Searching animation, cancel option, fallback messaging if no driver found. |
| Assigned | Driver matched to ride request. | Driver card, ETA, vehicle details, call driver action. |
| Driver En Route | Driver is travelling toward pickup. | Live tracking, dynamic ETA, cancel rules visible. |
| Driver Arrived | Driver has reached pickup point. | Arrival notification, call driver, cancellation fee warning if rider cancels. |
| Trip Started | Driver has started trip. | Live trip tracking, SOS, support/contact actions, cancel removed. |
| Trip Completed | Ride ended successfully. | Trip summary, rating, receipt, issue reporting entry point. |
| Cancelled by Rider | Ride cancelled by rider before trip start. | Cancellation reason, fee/policy summary, support if dispute exists. |
| Cancelled by Driver | Ride cancelled by driver before trip start. | Clear rider notification, rebook/fallback options, support entry point. |
| No Driver Found | System failed to assign driver within allowed SLA. | Explicit result, retry, alternate ride type, schedule later, support. |
| Support Intervention Required | Booking/trip/ticket has entered manual handling due to exception. | Rider sees status note and support updates where relevant. |

### 4.2 Core Rider-Facing Data Objects

| Object | Minimum Fields Expected in Technical Spec |
|--------|-------------------------------------------|
| Rider Profile | rider_id, mobile_number, name, email, profile_photo_url, language_preference, notification_preferences, account_status |
| Saved Address | address_id, rider_id, label, full_address, latitude, longitude, landmark, last_used_at |
| Ride Request | request_id, rider_id, pickup_location, drop_location, ride_type, scheduled_pickup_at, promo_code, points_redeemed, collection_mode_preference, request_status |
| Ride Booking | booking_id, request_id, assigned_driver_id, vehicle_details, assignment_time, estimated_fare, surge_details, booking_status, cancellation_policy_snapshot |
| Trip Record | trip_id, booking_id, actual_pickup_time, actual_drop_time, trip_distance_km, trip_duration_min, fare_reference_breakdown, final_reference_amount, cancellation_fee, trip_status |
| Promo Application | promo_code, eligibility_status, discount_type, discount_value, applied_amount, rejection_reason |
| Loyalty Transaction | transaction_id, rider_id, trip_id, transaction_type, points_delta, balance_after, expiry_at, remarks |
| Referral Reward | referral_event_id, referrer_rider_id, referred_rider_id, referral_code, qualification_status, reward_type, reward_value, credited_at |
| Support Ticket | ticket_id, rider_id, trip_id, issue_type, description, attachment_urls, priority, ticket_status, created_at, updated_at, resolution_note |
| App Configuration | enabled_ride_types, enabled_plugins, referral_reward_rule, loyalty_rule, cancellation_rule, support_contact_details, legal_links, brand_assets |

### 4.3 Notification & Event Triggers

| Event | Trigger Condition | Rider-Facing Outcome |
|-------|-------------------|----------------------|
| OTP Sent | Mobile number accepted for authentication | OTP screen with resend timer |
| OTP Verified | Correct OTP submitted | Account login or first-time profile completion |
| Booking Submitted | Rider taps Book Ride successfully | Searching state begins |
| Driver Assigned | Matching engine assigns a driver | Driver details and ETA shown |
| Driver Arriving | Driver is near pickup or ETA threshold reached | Push + in-app alert |
| Driver Arrived | Driver marks arrival / geofence hit | Pickup-ready message shown |
| Trip Started | Driver starts trip | Trip tracking state begins; cancel removed |
| Trip Completed | Driver ends trip | Summary, rating, receipt actions available |
| Ride Cancelled | Rider/driver/system cancels before trip start | Cancellation summary and next steps |
| No Driver Found | Matching timeout/SLA exhausted | Retry and support options shown |
| Support Ticket Created | Rider submits a help request | Immediate acknowledgement and ticket ID |
| Support Ticket Updated | Agent changes ticket status or adds message | Rider sees timeline update and notification |
| Reward Credited | Referral or loyalty rule qualifies rider | Ledger updated and confirmation shown |

### 4.4 Configurable Business Rules

| Rule Area | Default / Base Assumption | Technical Spec Note |
|-----------|---------------------------|---------------------|
| OTP Expiry | 5 minutes | Keep environment configurable. |
| OTP Resend Cooldown | 30 seconds | Add abuse/rate-limit controls. |
| Local Booking SLA | 1 to 2 minutes | Define timeout and retry behavior. |
| Scheduled Booking SLA | Confirm at least 4 hours before pickup | Define exception path for late-created scheduled rides. |
| Short-Notice Scheduled Booking SLA | 15 minutes for booking created within 4 hours of pickup | Explicitly distinguish from standard scheduled flow. |
| Free Cancellation Window | 2 minutes after booking before driver accepts | Make admin configurable. |
| Cancellation Fee Rules | Standard after accept, higher after arrival | Preserve policy snapshot at booking time. |
| Referral Reward | ₹25 default | Value and crediting rule configurable. |
| Loyalty Rule | 1 point per ₹100 default | Rounding, redemption, and expiry configurable. |
| Plugin Controls | Referral, loyalty, promo, ride categories | Ensure hidden UI and disabled API flows stay aligned. |
| Support SLA | Immediate acknowledgement; first response default 15 minutes for active-trip issues | Company-specific override supported. |
| Direct Pay Collection Mode | Cash/direct UPI by company policy | Technical spec should define allowed values and receipt language. |

---

## 5. Rider-Specific Non-Functional Requirements

| ID       | Category    | Requirement                                                                      |
|----------|-------------|----------------------------------------------------------------------------------|
| NFR-R01  | Performance | The rider app Home Screen shall load within 3 seconds on a 4G connection.        |
| NFR-R02  | Performance | Ride options shall be displayed within 5 seconds of confirming pickup and drop.  |
| NFR-R03  | Performance | Driver location on map shall refresh at most every 3 seconds during active tracking. |
| NFR-R04  | Usability   | A new rider shall be able to complete their first booking in under 3 minutes from app open. |
| NFR-R05  | Usability   | All primary actions (Book, Cancel, SOS) shall be reachable within 2 taps from the relevant screen. |
| NFR-R06  | Compatibility | The app shall function correctly on Android 8.0+ and iOS 13+ devices.           |
| NFR-R07  | Security    | OTP-based authentication; no passwords stored; session tokens shall expire on logout. |
| NFR-R08  | Security    | Rider's location data shall only be shared with the matched driver and the platform; never with advertisers. |
| NFR-R09  | Offline     | The app shall display a clear "no connectivity" message and gracefully handle network interruptions during a trip. |
| NFR-R10  | Configurability | White-label plugin configuration shall be applied consistently per company release build with no runtime feature leakage from other company configurations. |
| NFR-R11  | Reliability | Booking submission shall be idempotent; duplicate booking creation from repeated taps/retries shall be prevented. |
| NFR-R12  | Reliability | App crash-free session rate target shall be at least 99.5% monthly on supported devices. |
| NFR-R13  | Supportability | In-app support ticket acknowledgement shall be generated immediately, and first human response SLA shall be configurable per company policy (default 15 minutes for active-trip issues). |

---

## 6. Approval

| Role              | Name | Signature | Date |
|-------------------|------|-----------|------|
| Product Owner     |      |           |      |
| Business Analyst  |      |           |      |
| Tech Lead         |      |           |      |

---

*End of Document — BRD-REKLA-RIDER-002 v1.2*
