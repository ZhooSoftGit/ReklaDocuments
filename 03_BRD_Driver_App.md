# Business Requirements Document (BRD) — Driver App
## Rekla Ride App

| Field         | Details                                          |
|---------------|--------------------------------------------------|
| Document ID   | BRD-REKLA-DRIVER-003                             |
| Version       | 1.7                                              |
| Status        | Draft — Expanded Driver Features                 |
| Date          | March 21, 2026                                   |
| Updated       | March 21, 2026 — v1.7: ride history, shift history, policy alerts, and app permissions added |
| Prepared By   | ZhooSoft Team                                    |
| Parent BRD    | [BRD-REKLA-MASTER-001](01_BRD_Rekla_Ride_App.md) |

---

## Table of Contents

1. [Purpose & Scope](#1-purpose--scope)
2. [Driver Persona](#2-driver-persona)
3. [Functional Requirements](#3-functional-requirements)
   - [3.1 Registration & Onboarding](#31-registration--onboarding)
   - [3.2 Login & Authentication](#32-login--authentication)
   - [3.3 Primary Driving Location](#33-primary-driving-location)
   - [3.4 Home Screen / Driver Dashboard](#34-home-screen--driver-dashboard)
   - [3.5 Availability Management (Online / Offline)](#35-availability-management-online--offline)
   - [3.6 Ride Categories & Driver Modes](#36-ride-categories--driver-modes)
   - [3.7 Ride Request Handling](#37-ride-request-handling)
   - [3.8 Navigation & Trip Execution](#38-navigation--trip-execution)
       - [3.8.1 Local Ride End-to-End Flow](#381-local-ride-end-to-end-flow)
       - [3.8.2 Outstation Ride End-to-End Flow](#382-outstation-ride-end-to-end-flow)
       - [3.8.3 Package Ride End-to-End Flow](#383-package-ride-end-to-end-flow)
       - [3.8.4 Driver Real-Time Validation Scenarios](#384-driver-real-time-validation-scenarios)
   - [3.9 Earnings, Commission & Packages](#39-earnings-commission--packages)
   - [3.10 Profile, Support & Language Settings](#310-profile-support--language-settings)
   - [3.11 Ratings & Reviews](#311-ratings--reviews)
   - [3.12 Notifications](#312-notifications)
    - [3.13 Ride History](#313-ride-history)
    - [3.14 Shift History](#314-shift-history)
    - [3.15 Rekla Policy Violation Alerts](#315-rekla-policy-violation-alerts)
    - [3.16 Driver App Permissions](#316-driver-app-permissions)
4. [Driver-Specific Non-Functional Requirements](#4-driver-specific-non-functional-requirements)
5. [Approval](#5-approval)

---

## 1. Purpose & Scope

This document defines detailed functional requirements for the **Rekla Driver mobile application** (iOS & Android). It covers all screens, user interactions, and system behaviours for the driver — from registration and KYC through to trip completion and earnings management.

**In Scope:** All features documented in this Driver BRD v1.7 for the current release baseline.
**White-Label Product Note:** Rekla Driver App can be delivered per company as a white-label deployment with company-specific branding and configuration.
**White-Label Scope Boundary Note:** Frontend white-label customization is expected mainly for app identity and light UI behavior (for example app name, logo, theme, legal/policy links, support contacts, and UI text assets). Business behavior changes (for example base fare, vehicle types, trip-type enablement, and location-based business rules) are driven by individual backend configuration for that deployment. Core major driver workflows remain the Rekla baseline unless explicitly changed in signed scope.
**Configuration Independence Note:** Configuration is runtime value-driven only. Configuration records must remain independent and must not be FK-linked to core transactional/domain tables.
**Out of Scope:** Rider app, Admin portal. Refer to parent BRD for platform-wide exclusions.

---

## 2. Driver Persona

| Field       | Details                                              |
|-------------|------------------------------------------------------|
| Name        | Ravi, 35, Full-time cab driver                       |
| Goal        | Maximize daily trips, earn transparently, work flexibly |
| Pain Points | Unclear commissions, complex apps, unfair trip allocation |
| Device      | Budget Android smartphone (Android 8.0+)             |
| Tech Level  | Basic — familiar with phone calls and simple apps    |
| Connectivity| 4G mobile data; may have intermittent coverage       |

---

## 3. Functional Requirements

### 3.1 Registration & Onboarding

**User Journey:**
```
Download App → Enter Mobile Number → OTP Verification → Fill Personal Details → Add Primary Driving Location → Upload Documents → Submit for Review → Admin Approves → Account Activated
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D01  | The app shall display a mobile number entry screen as the first screen for new drivers.        | Must     |
| FR-D02  | The system shall send an OTP to the entered mobile number for initial verification.            | Must     |
| FR-D03  | After OTP verification, the driver shall be prompted to complete a registration form with: full name, date of birth, gender, residential address, and emergency contact number. | Must |
| FR-D04  | The driver shall upload the following mandatory documents: Driving Licence, Aadhaar card, Vehicle RC Book, and a profile photo. | Must |
| FR-D05  | The system shall allow document upload via camera capture or file picker.                      | Must     |
| FR-D06  | Each uploaded document shall display a preview and allow the driver to re-upload if unsatisfied. | Must   |
| FR-D07  | The system shall require the driver to complete all mandatory registration fields and document uploads before allowing final submission for approval. | Must |
| FR-D08  | After submission, the driver shall see a clear **"Under Review"** status screen indicating that the profile is pending admin approval. | Must |
| FR-D09  | The driver shall receive a push notification and SMS when the profile is approved or rejected by admin. | Must |
| FR-D10  | If rejected, the rejection reason shall be displayed in the app with an option to re-submit only the rejected details or documents. | Must |
| FR-D11  | A driver account shall only be activated after admin approval; the driver cannot access Online mode before activation. | Must |
| FR-D11A | During registration, the driver shall explicitly accept Terms of Service and Privacy Policy before final submission. | Must |
| FR-D11B | If a mobile number is already registered, the system shall prevent duplicate profile creation and guide the user to login or recovery flow. | Must |

---

### 3.2 Login & Authentication

**User Journey:**
```
App Launch → Enter Mobile Number → OTP Verification → Home Screen (if approved) or Status Screen (if pending/rejected)
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D12  | A registered driver shall log in using their mobile number and OTP.                           | Must     |
| FR-D13  | OTP shall be valid for 5 minutes; the driver shall be allowed to resend OTP after 30 seconds.  | Must     |
| FR-D14  | The session shall persist so the driver is not required to log in again on subsequent app opens. | Must   |
| FR-D15  | If the account is pending review, the app shall display an **"Application Under Review"** status screen after login. | Must |
| FR-D16  | If the account is rejected, the app shall display the rejection reason and option to re-submit documents. | Must |
| FR-D17  | Once the admin approves the profile, the driver shall receive a notification and on next login or refresh shall be taken to the Driver Dashboard with access to Online/Offline controls. | Must |
| FR-D18  | The driver shall be able to log out from the Profile section, which clears the local session.  | Must     |

---

### 3.3 Primary Driving Location

**User Journey:**
```
Registration Form → Select Native / Primary Driving Location → Save Service Area → Submit for Approval
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D19  | During onboarding, the driver shall be required to set a **Native Location** or **Primary Driving Location**. | Must |
| FR-D20  | The primary driving location shall be selectable via map search, pin drop, or saved city/area list. | Must |
| FR-D21  | The selected primary driving location shall be stored as part of the driver profile and visible to admin. | Must |
| FR-D22  | The driver shall be able to update the primary driving location later from Profile settings, subject to admin review if required by business policy. | Should |
| FR-D23  | The platform should use the primary driving location as an input for driver discovery, onboarding review, and reporting. | Should |

---

### 3.4 Home Screen / Driver Dashboard

**User Journey:**
```
Login (Approved Account) → Driver Dashboard → Go Online → Receive Ride Requests
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D24  | After login, an approved driver shall land on the Driver Dashboard / Home Screen.             | Must     |
| FR-D25  | The Dashboard shall display a full-screen map showing the driver's current location.          | Must     |
| FR-D26  | The Dashboard shall prominently display the driver's current availability status: **Online** or **Offline**, with a toggle to switch. | Must |
| FR-D27  | The Dashboard shall display a summary of today's stats: trips completed, earnings, and hours online. | Should |
| FR-D28  | The Dashboard shall provide quick navigation to: Earnings, Trip History, Shift History, Profile, Rekla Policy Alerts, Support, Notifications, and Language Settings. | Must |
| FR-D29  | The driver's average rating shall be visible on the Dashboard.                                | Should   |
| FR-D30  | The Dashboard shall display the driver's active commission package status, expiry date, and remaining validity where applicable. | Must |
| FR-D31  | The Dashboard shall display the currently enabled ride categories and whether **Acting Driver Mode** is active. | Should |

---

### 3.5 Availability Management (Online / Offline)

**User Journey:**
```
Dashboard → Toggle Online → Ready to Receive Requests → Toggle Offline → No Requests Received
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D32  | The driver shall be able to toggle their availability status between **Online** and **Offline** with a single prominent control. | Must |
| FR-D33  | When Online, the driver's live location shall be visible to the dispatch system and the driver becomes eligible for ride request matching. | Must |
| FR-D34  | When Offline, the driver shall not receive ride requests and their active availability shall not be broadcast to matching services. | Must |
| FR-D35  | The driver shall be able to go Offline at any time except during an active trip; during an active trip, the Offline toggle shall be disabled. | Must |
| FR-D36  | If an active trip is in progress and the app is force-closed or loses connectivity, the driver's Online status shall be preserved until the trip is completed or manually resolved. | Must |
| FR-D37  | The system shall prevent a driver from going Online if mandatory documents are expired, the account is suspended, or no valid commission package is active when package-based operation is required by business policy. | Must |

---

### 3.6 Ride Categories & Driver Modes

**User Journey:**
```
Driver Dashboard → Select Enabled Ride Categories → Toggle Acting Driver Mode (Optional) → Go Online
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D38  | The driver shall be able to enable or disable supported ride categories from the dashboard or profile settings before going Online. | Must |
| FR-D39  | The platform shall support at least three ride categories for drivers: **Local Ride**, **Package-Based Ride**, and **Outstation Ride**. | Must |
| FR-D40  | The driver shall be able to opt in only to the ride categories for which the vehicle, permits, and admin approvals are valid. | Must |
| FR-D41  | Ride requests sent to the driver shall respect the currently enabled ride categories.          | Must     |
| FR-D42  | The driver shall be able to enable an optional **Acting Driver Mode** to indicate availability to drive another owner's or partner's vehicle, subject to admin approval and policy rules. | Should |
| FR-D43  | When Acting Driver Mode is enabled, the driver's profile shall be discoverable for approved alternate-vehicle driving opportunities and the mode shall be visibly indicated on the dashboard. | Should |
| FR-D44  | Acting Driver Mode shall not interfere with the driver's current active trip flow; it can only be toggled when the driver is not in an active ride. | Must |

---

### 3.7 Ride Request Handling

**User Journey:**
```
Driver Online → Ride Request Received → Request Card Displayed → Accept or Decline (within timer) → Navigate to Pickup
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D45  | When a ride is matched, the driver shall receive an audio alert, push notification, and a full-screen ride request card. | Must |
| FR-D45A | For **Local Ride** category, ride request notification shall be sent immediately after matching and shown as an instant request card. | Must |
| FR-D46  | The ride request card shall display: pickup location name, pickup distance from current location, estimated trip distance, ride category, fare or earnings estimate, and pickup ETA. | Must |
| FR-D47  | For package-based rides, the request card shall show package duration, included kilometers, and package earnings estimate. | Must |
| FR-D47A | For **Package-Based Ride**, the driver shall receive a scheduled request notification **4 hours or 8 hours before trip start time** as configured by admin and rider booking type. | Must |
| FR-D48  | For outstation rides, the request card shall show outstation indicator, approximate route distance, and any special trip conditions configured by admin. | Must |
| FR-D48A | For **Outstation Ride**, the driver shall receive a scheduled request notification **4 hours or 8 hours before trip start time** as configured by admin and rider booking type. | Must |
| FR-D49  | The driver shall have a configurable countdown timer (default: **1 minute / 60 seconds**) to accept or decline the request; if no action is taken, the request shall auto-expire and be reassigned. | Must |
| FR-D50  | The driver shall be able to tap **"Accept"** to accept the ride or **"Decline"** to reject the request. | Must |
| FR-D51  | Upon acceptance, the app shall navigate to the navigation screen showing the route to the rider's pickup location. | Must |
| FR-D52  | If the driver declines or misses 3 consecutive requests, the system shall display a warning; after 5 consecutive misses, the driver shall be temporarily moved to Offline status with a notification. | Must |
| FR-D52A | Once the driver accepts a ride request, the driver shall immediately be marked as **Engaged** and the dispatch system shall stop sending any new ride requests to that driver until the accepted trip is cancelled or completed. | Must |
| FR-D52B | A driver who already has an accepted ride or an active in-progress trip shall not be eligible for any other ride matching cycle. | Must |

---

### 3.8 Navigation & Trip Execution

This section is intentionally split by ride type so the trip execution logic is easy to read and implement without mixing Local, Outstation, and Package rules.

#### 3.8.1 Local Ride End-to-End Flow

**Local Ride Journey (Driver):**
```
Receive Local Request (Immediate) → Accept/Decline (60s) → Navigate to Pickup → Arrived at Pickup → Start Ride with Rider OTP → Navigate to Drop → End Ride with Rider OTP → Trip Complete
```

**Local Ride Exception Journey:**
```
End Ride with OTP Failed/Unavailable → Force End Ride (Reason + Required Proof) → Trip Marked for Admin Review
```

| ID       | Requirement                                                                                   | Priority |
|----------|-----------------------------------------------------------------------------------------------|----------|
| FR-D53   | After accepting a **Local Ride**, the app shall display turn-by-turn navigation to the rider's pickup point using the in-app maps integration. | Must |
| FR-D54   | The local-ride navigation screen shall display: rider's name, masked phone number, pickup address, and ETA to pickup. | Must |
| FR-D55   | The driver shall tap **"I've Arrived"** when reaching the local-ride pickup point; this shall trigger a notification to the rider. | Must |
| FR-D56   | The driver shall tap **"Start Trip"** once the rider has boarded; this begins the trip meter and locks trip start time and location. | Must |
| FR-D56C  | For **Local Ride**, trip start shall require rider OTP verification before the trip can be started. | Must |
| FR-D57   | After local-ride start, the app shall display navigation to the rider's drop location.       | Must |
| FR-D59A  | Before local-ride trip start, the driver shall be able to cancel an accepted ride by selecting a cancellation reason from a predefined list, subject to platform policy and monitoring. | Must |
| FR-D59B  | The system shall log local-ride driver-side cancellations and make them visible to admin for operational review and driver performance monitoring. | Must |
| FR-D59C  | If the rider does not arrive within a configured wait-time threshold, the driver shall be able to mark the local ride as **Rider No-Show**, with event timestamp and location captured for policy enforcement. | Should |
| FR-D60   | The driver shall tap **"End Trip"** upon reaching the local-ride drop location; this shall finalise the fare and mark the trip as Complete. | Must |
| FR-D60E  | For **Local Ride**, trip end shall require rider OTP verification before trip completion.    | Must |
| FR-D60F  | For **Local Ride**, if rider OTP verification cannot be completed due to exceptional reasons, driver may use **Force End Ride** by selecting a mandatory reason and submitting required proof fields configured by admin. | Must |
| FR-D60G  | A Local Ride **Force End Ride** action shall be logged with timestamp, location, reason, and audit metadata, and the trip shall be marked for admin review. | Must |
| FR-D61   | The final local-ride fare shall be automatically calculated by the system at trip end; the driver shall not manually enter the fare. | Must |
| FR-D62   | On local-ride completion, the driver shall see a trip summary: distance, duration, gross earnings, commission deduction, and net earnings. | Must |
| FR-D63   | The driver shall be prompted to rate the rider (1–5 stars) after local-ride completion.      | Must |
| FR-D64   | The driver shall be able to contact the rider via a masked phone call at any point during an active local ride. | Should |

#### 3.8.2 Outstation Ride End-to-End Flow

**Outstation Ride Journey (Driver):**
```
Receive Scheduled Outstation Request (4h or 8h Before) → Accept/Decline (60s) → Navigate to Pickup → Arrived at Pickup → Start Ride (Enter Start KM) → Navigate Across Route → End Ride (Enter End KM) → Fare Computed from KM Rules → Trip Complete
```

| ID       | Requirement                                                                                   | Priority |
|----------|-----------------------------------------------------------------------------------------------|----------|
| FR-D65A  | After accepting an **Outstation Ride**, the app shall display turn-by-turn navigation to the rider's pickup point using the in-app maps integration. | Must |
| FR-D65B  | The outstation navigation screen shall display: rider's name, masked phone number, pickup address, ride category, and ETA to pickup. | Must |
| FR-D65C  | The driver shall tap **"I've Arrived"** when reaching the outstation pickup point; this shall trigger a notification to the rider. | Must |
| FR-D65D  | For **Outstation Ride**, when the driver taps **"Start Trip"**, the app shall prompt for the vehicle's **current odometer KM** before trip start. | Must |
| FR-D65E  | The system shall validate that the outstation start KM is a positive numeric value and within reasonable tolerance against prior recorded odometer values, when available. | Must |
| FR-D65F  | After outstation trip start, the app shall display navigation to the rider's outstation destination and preserve route continuity across city boundaries. | Must |
| FR-D65G  | Before outstation trip start, the driver shall be able to cancel the accepted ride with a mandatory reason, subject to platform policy and monitoring. | Must |
| FR-D65H  | When the driver taps **"End Trip"** for an outstation ride, the app shall prompt for the vehicle's **ending odometer KM** before completion is finalised. | Must |
| FR-D65I  | The system shall validate that the outstation ending KM is greater than or equal to the starting KM; invalid values shall block trip completion. | Must |
| FR-D65J  | The outstation fare calculation shall use the captured **start KM** and **end KM** to determine actual distance travelled and any additional kilometer charges. | Must |
| FR-D65K  | The outstation trip summary shall display start KM, end KM, total traveled KM, applicable kilometer charges, gross earnings, commission/package deduction, and net earnings. | Must |
| FR-D65L  | For **Outstation Ride**, trip-end OTP is not mandatory by default unless explicitly enabled by admin policy. | Should |
| FR-D65M  | The driver shall be prompted to rate the rider after outstation trip completion.             | Must |
| FR-D65N  | The driver shall be able to contact the rider via a masked phone call at any point during an active outstation ride. | Should |

#### 3.8.3 Package Ride End-to-End Flow

**Package Ride Journey (Driver):**
```
Receive Scheduled Package Request (4h or 8h Before) → Accept/Decline (60s) → Navigate to Pickup → Arrived at Pickup → Start Ride (Enter Start KM) → Run Package Trip → Track Time and KM Usage → End Ride (Enter End KM) → Fare Computed from Package Rules → Trip Complete
```

| ID       | Requirement                                                                                   | Priority |
|----------|-----------------------------------------------------------------------------------------------|----------|
| FR-D66A  | After accepting a **Package Ride**, the app shall display turn-by-turn navigation to the rider's pickup point using the in-app maps integration. | Must |
| FR-D66B  | The package-ride navigation screen shall display: rider's name, masked phone number, pickup address, package type, included time, included kilometers, and ETA to pickup. | Must |
| FR-D66C  | The driver shall tap **"I've Arrived"** when reaching the package-ride pickup point; this shall trigger a notification to the rider. | Must |
| FR-D66D  | For **Package Ride**, when the driver taps **"Start Trip"**, the app shall prompt for the vehicle's **current odometer KM** before trip start. | Must |
| FR-D66E  | The system shall validate that the package-ride start KM is a positive numeric value and within reasonable tolerance against prior recorded odometer values, when available. | Must |
| FR-D66F  | After package-ride start, the app shall display trip navigation and package usage details such as elapsed time and used kilometers during the trip. | Must |
| FR-D66G  | Before package-ride trip start, the driver shall be able to cancel the accepted ride with a mandatory reason, subject to platform policy and monitoring. | Must |
| FR-D66H  | When the driver taps **"End Trip"** for a package ride, the app shall prompt for the vehicle's **ending odometer KM** before completion is finalised. | Must |
| FR-D66I  | The system shall validate that the package-ride ending KM is greater than or equal to the starting KM; invalid values shall block trip completion. | Must |
| FR-D66J  | The package fare calculation shall use the captured **start KM** and **end KM** to determine traveled distance and any extra-kilometer charges beyond included package limits. | Must |
| FR-D66K  | The package-ride trip summary shall display start KM, end KM, total traveled KM, included KM allowance, elapsed package time, extra KM/time charges if any, gross earnings, commission/package deduction, and net earnings. | Must |
| FR-D66L  | For **Package Ride**, trip-end OTP is not mandatory by default unless explicitly enabled by admin policy. | Should |
| FR-D66M  | The driver shall be prompted to rate the rider after package-ride completion.                | Must |
| FR-D66N  | The driver shall be able to contact the rider via a masked phone call at any point during an active package ride. | Should |

#### 3.8.4 Driver Real-Time Validation Scenarios

| ID       | Validation Scenario                                                                              | Expected System Behaviour |
|----------|--------------------------------------------------------------------------------------------------|---------------------------|
| RTV-D01  | Driver is **Offline**                                                                            | No ride request shall be sent to the driver. |
| RTV-D02  | Driver has **accepted** a ride but trip has not yet started                                      | Driver shall be marked as Engaged and shall not receive any other ride request. |
| RTV-D03  | Driver is in an **active trip**                                                                  | Driver shall not appear in any availability pool or matching result for other riders. |
| RTV-D04  | Driver attempts to manually go Online from another device while already in an active trip       | The system shall preserve the active-trip state and prevent duplicate availability. |
| RTV-D05  | Dispatch service attempts to assign another ride to the same driver due to network delay/race condition | The backend shall reject the second assignment if the driver already has an accepted or active trip. |
| RTV-D06  | Driver cancels the accepted ride before trip start                                               | Driver may become eligible for new ride matching again after cancellation is confirmed. |
| RTV-D07  | Trip is completed successfully                                                                   | Driver status shall return to Available only if the driver remains Online. |
| RTV-D08  | Driver loses connectivity during an active trip                                                  | Driver shall remain locked to the current trip until completion or administrative resolution. |

---

### 3.9 Earnings, Commission & Packages

**User Journey:**
```
Dashboard → Tap "Earnings" → View Earnings from Login Time / Last 10 Days / Custom Range → View Per-Trip Breakdown → Review Package Deductions
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D65  | The driver shall be able to access an Earnings section from the Dashboard.                    | Must     |
| FR-D66  | The Earnings screen shall display quick filters for: **From Login Time**, **Today**, **Last 10 Days**, **This Week**, **This Month**, and **Custom Date Range**. | Must |
| FR-D67  | Each earnings period shall show: gross earnings, platform commission or package fee deducted, incentives if any, and net earnings. | Must |
| FR-D68  | The driver shall be able to view a per-trip earnings list with: date, time, trip route summary, ride category, gross fare, commission/package deduction, and net earnings per trip. | Must |
| FR-D69  | Cancellation fees earned by the driver, if applicable under business rules, shall be reflected separately in the earnings breakdown. | Should |
| FR-D70  | The earnings data shall be updated in near real-time after each trip completion.               | Must     |
| FR-D71  | The driver shall be able to view payout history (amounts settled to their bank account) if the payout feature is enabled by admin. | Should |
| FR-D72  | The driver shall be able to view available **commission packages** or subscription plans inside the app. | Must |
| FR-D72A | The package listing screen shall present packages in a clear comparison layout showing price, validity, commission benefit, applicable ride categories, and net earning impact examples so the driver is not confused while purchasing. | Must |
| FR-D72B | The system shall highlight a **recommended package** based on the driver's recent ride volume and categories, with a clear explanation of why it is recommended. | Should |
| FR-D73  | Each commission package shall display package name, validity period, price, commission benefit, and included conditions. | Must |
| FR-D73A | Package pricing shall show transparent fare components: base price, taxes/fees (if any), promo discount amount (if applied), and final payable amount. | Must |
| FR-D74  | The driver shall be able to purchase a package using supported payment methods configured by the platform. | Must |
| FR-D74A | During package purchase, the driver shall have an option to **apply a promo code** and validate it before payment confirmation. | Must |
| FR-D74B | During initial launch/free campaign period, a **default promo code** shall appear automatically on the package purchase screen and be auto-applied for eligible drivers. | Must |
| FR-D74C | If the default promo makes the package fully free, the payable amount shall be shown as **0** and purchase shall complete without requiring payment collection. | Must |
| FR-D74D | The driver shall be able to remove or replace the auto-applied promo code before confirming package purchase, if policy allows. | Should |
| FR-D74E | If package purchase payment fails, the app shall display a clear failure reason and provide retry and alternate payment options without losing selected package and promo context. | Must |
| FR-D75  | After successful package purchase, the driver's account shall be updated immediately with the new package benefits and expiry details. | Must |
| FR-D76  | The driver shall be able to view current package status, purchase history, and upcoming expiry reminders. | Must |
| FR-D76A | The app shall notify drivers before package expiry at configurable intervals (for example 7 days, 3 days, and 1 day prior) and show post-expiry impact on commission. | Must |
| FR-D76B | Package auto-renewal setting shall be explicit and opt-in only; if enabled, the app shall request consent for auto-debit according to payment rules. | Should |
| FR-D76C | If package expires during active availability, the system shall apply configured fallback commission rules for new rides and clearly notify the driver before accepting next ride. | Must |

---

### 3.10 Profile, Support & Language Settings

**User Journey:**
```
Dashboard → Profile / Support / Settings → View or Update Driver Details → Manage Documents / Raise Issue / Change Language
```

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D77  | The driver shall be able to access the Profile section from the Dashboard navigation.         | Must     |
| FR-D78  | The Profile screen shall display: name, mobile number (read-only), profile photo, vehicle details, primary driving location, active ride categories, and document status. | Must |
| FR-D79  | The driver shall be able to update their display name, profile photo, and residential address. | Must |
| FR-D80  | The driver shall be able to view the status of each submitted document: Approved, Pending Review, Rejected, or Expiring Soon. | Must |
| FR-D81  | The driver shall be able to re-upload a specific document if it was rejected or is expiring.  | Must     |
| FR-D82  | The app shall display a warning when any document is due to expire within 30 days and prompt the driver to renew it. | Must |
| FR-D83  | The mobile number shall be the primary identifier and shall not be editable without OTP re-verification. | Must |
| FR-D84  | The driver shall be able to update vehicle details, enabled ride categories, and primary driving location, subject to admin re-verification where applicable. | Should |
| FR-D85  | The driver shall be able to access a **Support / Help** section to create a support ticket, raise an issue, or raise a concern related to trips, payments, account approval, package purchase, or technical problems. | Must |
| FR-D86  | The support ticket flow shall allow the driver to choose a category, enter a description, attach screenshots or documents, and submit the issue. | Must |
| FR-D87  | The driver shall be able to view the status of submitted issues: Open, In Progress, Resolved, or Closed. | Must |
| FR-D88  | The driver shall be able to add follow-up comments to an open support issue.                  | Should   |
| FR-D89  | The driver shall be able to access **Language Settings** and switch the app language from the supported list without logging out. | Must |
| FR-D90  | The selected language shall be applied across the driver app UI and persist across future sessions. | Must |
| FR-D90A | The Settings section shall allow the driver to manage notification preferences including scheduled package/outstation alerts and alarm behaviour, while keeping mandatory operational alerts always enabled. | Must |
| FR-D90B | The app shall provide a test option for default package/outstation alarm sound so drivers can verify alert audibility on their device. | Should |

---

### 3.11 Ratings & Reviews

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D91  | The driver shall be prompted to rate the rider (1–5 stars) on the trip summary screen after every completed trip. | Must |
| FR-D92  | The driver shall be able to view their own overall average rating on the Dashboard and Profile screen. | Must |
| FR-D93  | The driver shall be able to view a breakdown of their last 20 ratings, including star level and any text feedback left by riders. | Should |
| FR-D94  | The driver shall not be able to see the rider's rating of the trip until the driver has submitted their own rating, to reduce bias. | Should |

---

### 3.12 Notifications

| ID      | Requirement                                                                                    | Priority |
|---------|------------------------------------------------------------------------------------------------|----------|
| FR-D95  | The driver shall receive a push notification and audio alert for each new incoming ride request. | Must   |
| FR-D95A | For **Package-Based Ride** and **Outstation Ride** requests, the app shall trigger a **default high-priority alarm** along with the notification; the alarm shall continue until acknowledged, timed out, or actioned. | Must |
| FR-D95B | The default alarm for Package-Based and Outstation requests shall work even when the app is in background, subject to OS notification permissions. | Must |
| FR-D96  | The driver shall receive notifications for key trip events: Rider Cancelled, Driver Arrived acknowledgement, Trip Started confirmation, and Trip Completed. | Must |
| FR-D97  | The driver shall receive a notification when the account status changes: Approved, Suspended, Rejected, or Document Expired. | Must |
| FR-D98  | The driver shall receive periodic earnings summaries, such as end-of-day earnings, package expiry reminders, and payout updates, based on notification preferences. | Should |
| FR-D99  | The driver shall receive document expiry reminder notifications 30 days and 7 days before any mandatory document expires. | Must |
| FR-D100 | The driver shall receive a notification when a support issue is updated or resolved by the admin team. | Must |
| FR-D101 | The driver shall be able to view all past notifications in a Notifications inbox within the app. | Should |
| FR-D102 | For scheduled Package-Based and Outstation rides, the driver shall receive at least one reminder notification before ride start in addition to the primary 4-hour or 8-hour notification window. | Must |

---

### 3.13 Ride History

**User Journey:**
```
Dashboard → Trip History → Select Filter (Today / Day / From Login / Custom Date Range) → View Trip List → Open Trip Detail
```

| ID       | Requirement                                                                                   | Priority |
|----------|-----------------------------------------------------------------------------------------------|----------|
| FR-D103  | The driver shall be able to access a **Trip History** section from the Dashboard.            | Must |
| FR-D104  | The Trip History screen shall display a chronological list of driver trips including Local, Outstation, and Package rides. | Must |
| FR-D105  | The driver shall be able to filter trip history by: **Today**, **Specific Day**, **From Login Time**, and **Custom Date Range**. | Must |
| FR-D106  | Each trip history item shall display: trip date, start time, end time, ride category, rider name, pickup, drop, trip status, and net earning. | Must |
| FR-D107  | The driver shall be able to open a trip-history record to view full trip details including route summary, trip duration, distance/KM details, fare breakdown, commission/package deduction, and final earning. | Must |
| FR-D108  | Trip History shall support filtering by ride category: Local Ride, Outstation Ride, and Package Ride. | Should |
| FR-D109  | Cancelled trips, no-show trips, and force-ended trips shall be visible in Trip History with their final status and reason where applicable. | Must |

---

### 3.14 Shift History

**User Journey:**
```
Dashboard / Profile → Shift History → View Login/Logout Sessions → Open Shift Detail
```

| ID       | Requirement                                                                                   | Priority |
|----------|-----------------------------------------------------------------------------------------------|----------|
| FR-D110  | The driver app shall maintain a **Shift History** of driver sessions based on login/logout and Online/Offline usage. | Must |
| FR-D111  | A shift record shall capture at minimum: login time, logout time, first online time, final offline time, total online duration, total trips completed, and total earning during the shift. | Must |
| FR-D112  | The driver shall be able to view shift history by: Today, Last 7 Days, Last 10 Days, and Custom Date Range. | Must |
| FR-D113  | If the driver logs in but never goes Online, the shift history shall still record the session separately from active driving time. | Should |
| FR-D114  | The driver shall be able to open an individual shift record to view shift summary metrics including completed trips, cancellations, no-shows, and net earnings. | Should |

---

### 3.15 Rekla Policy Violation Alerts

**User Journey:**
```
System Detects Violation / Support Flags Violation → Driver Receives Alert → View Alert Detail → Acknowledge / Act → Escalation if repeated
```

| ID       | Requirement                                                                                   | Priority |
|----------|-----------------------------------------------------------------------------------------------|----------|
| FR-D115  | The driver shall receive a **Rekla Policy Violation Alert** when the platform detects or records policy breaches such as repeated ride declines, customer misuse complaints, safety violations, abusive behaviour, suspicious cancellations, or fraud-related events. | Must |
| FR-D116  | A policy violation alert shall display: violation type, date/time, short description, severity level, and any action required by the driver. | Must |
| FR-D117  | The driver shall be required to acknowledge severe or repeated violation alerts before continuing to receive new ride requests, where configured by policy. | Should |
| FR-D118  | The app shall display warning-stage alerts before suspension for repeat behaviour such as excessive ride declines, frequent driver-side cancellations, or confirmed customer complaints. | Must |
| FR-D119  | If the driver's account is temporarily restricted or suspended due to policy violation, the app shall show the restriction reason, start time, duration if applicable, and next steps. | Must |
| FR-D120  | Policy violation alerts shall remain accessible in the app for later review until they are resolved or expire under platform retention rules. | Should |

---

### 3.16 Driver App Permissions

**User Journey:**
```
First App Launch / Feature Access → Request Permission → User Grants or Denies → Feature Continues or Graceful Guidance Shown
```

| ID       | Requirement                                                                                   | Priority |
|----------|-----------------------------------------------------------------------------------------------|----------|
| FR-D121  | The app shall explicitly request all required device permissions at the appropriate moment of feature use rather than requesting unrelated permissions upfront. | Must |
| FR-D122  | The driver app shall require **Location Permission** to show current position, enable ride matching, and support trip navigation. | Must |
| FR-D123  | The driver app shall require **Background Location Permission** where needed for active trip tracking and online availability while app is not foregrounded. | Must |
| FR-D124  | The driver app shall require **Notification Permission** to deliver ride requests, approval updates, scheduled package/outstation alerts, alarms, and support updates. | Must |
| FR-D125  | The driver app shall require **Camera / Media / File Access Permission** to upload registration documents, profile photo, and support attachments. | Must |
| FR-D126  | The app shall require or invoke **Phone Call capability** only when the driver initiates masked calling to rider/support. | Should |
| FR-D127  | If a required permission is denied, the app shall clearly explain which feature is affected and guide the driver to retry permission or open device settings. | Must |
| FR-D128  | If Location or Notification permission remains denied, the app shall clearly indicate that the driver cannot receive or operate rides until the required permission is enabled. | Must |

---

## 4. Driver-Specific Non-Functional Requirements

| ID       | Category      | Requirement                                                                      |
|----------|---------------|----------------------------------------------------------------------------------|
| NFR-D01  | Performance   | The ride request card shall appear on the driver's screen within 5 seconds of a match being made. |
| NFR-D02  | Performance   | Navigation directions shall load within 3 seconds of ride acceptance.            |
| NFR-D03  | Usability     | The Online/Offline toggle shall be the most prominent control on the Dashboard.  |
| NFR-D04  | Usability     | The Accept / Decline buttons on the request card shall be large enough for one-handed operation on a budget device. |
| NFR-D05  | Compatibility | The app shall function correctly on Android 8.0+ and iOS 13+, including on budget-tier devices with 2GB RAM. |
| NFR-D06  | Battery       | The app shall implement background location updates efficiently to minimise excessive battery drain when the driver is Online. |
| NFR-D07  | Security      | Document images uploaded during onboarding shall be transmitted over TLS and stored securely; access restricted to admin roles. |
| NFR-D08  | Offline       | If connectivity is lost mid-trip, the app shall continue tracking the trip locally and sync data when connectivity is restored. |
| NFR-D09  | Reliability   | The "End Trip" action shall be idempotent — retrying it (e.g., on network retry) shall not create duplicate trip completion records. |
| NFR-D10  | Localization  | The driver app shall support language switching without reinstalling or re-authenticating. |
| NFR-D11  | Performance   | Earnings filters, including Last 10 Days and Custom Date Range, shall load within 3 seconds for normal data volumes. |
| NFR-D12  | Security      | Package purchases and driver payout history shall be stored with auditable transaction records. |
| NFR-D13  | Reliability   | The platform shall enforce a **single active ride state per driver** across app, backend, and dispatch services, preventing duplicate assignment even under retry, lag, or reconnect scenarios. |
| NFR-D14  | Usability     | A driver shall be able to complete package purchase confirmation in a maximum of 3 primary steps: select package, review promo/price, confirm purchase. |
| NFR-D15  | Reliability   | Scheduled package/outstation notifications (4-hour or 8-hour) shall be delivered with retry logic and delivery-status logging for audit and support investigation. |
| NFR-D16  | Security      | Package purchase, promo application, and fallback commission events shall be traceable with immutable audit logs for dispute handling. |

---

## 5. Approval

| Role              | Name | Signature | Date |
|-------------------|------|-----------|------|
| Product Owner     |      |           |      |
| Business Analyst  |      |           |      |
| Tech Lead         |      |           |      |

---

*End of Document — BRD-REKLA-DRIVER-003 v1.7*
