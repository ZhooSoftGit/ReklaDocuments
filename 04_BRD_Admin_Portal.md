# Business Requirements Document (BRD) — Admin Portal
## Rekla Ride App

| Field         | Details                                          |
|---------------|--------------------------------------------------|
| Document ID   | BRD-REKLA-ADMIN-004                              |
| Version       | 1.2                                              |
| Status        | Draft                                            |
| Date          | March 21, 2026                                   |
| Prepared By   | ZhooSoft Team                                    |
| Parent BRD    | [BRD-REKLA-MASTER-001](01_BRD_Rekla_Ride_App.md) |

---

## Table of Contents

1. [Purpose & Scope](#1-purpose--scope)
2. [Admin Persona](#2-admin-persona)
3. [Functional Requirements](#3-functional-requirements)
   - [3.1 Admin Registration & Authentication](#31-admin-registration--authentication)
   - [3.2 Portal Navigation & Menu Structure](#32-portal-navigation--menu-structure)
   - [3.3 Dashboard — Section-wise Overview](#33-dashboard--section-wise-overview)
   - [3.4 Driver Management](#34-driver-management)
   - [3.5 Rider Management](#35-rider-management)
   - [3.6 Ride Management](#36-ride-management)
      - [3.6.1 Live Ride Monitoring](#361-live-ride-monitoring)
   - [3.7 Vehicle Management](#37-vehicle-management)
   - [3.8 Approvals Management](#38-approvals-management)
   - [3.9 Book via Admin](#39-book-via-admin)
   - [3.10 Fare & Pricing Configuration](#310-fare--pricing-configuration)
   - [3.11 Promo Code Management](#311-promo-code-management)
   - [3.12 Analytics & Charts](#312-analytics--charts)
   - [3.13 Support & Dispute Resolution](#313-support--dispute-resolution)
   - [3.14 Admin User & Access Management](#314-admin-user--access-management)
   - [3.15 Driver Package Subscription & Sales Operations](#315-driver-package-subscription--sales-operations)
   - [3.16 Audit & Activity Logs](#316-audit--activity-logs)
   - [3.17 Outstation & Package Quote Desk](#317-outstation--package-quote-desk)
   - [3.18 Custom Reports & Report Builder](#318-custom-reports--report-builder)
4. [Admin-Specific Non-Functional Requirements](#4-admin-specific-non-functional-requirements)
5. [Approval](#5-approval)

---

## 1. Purpose & Scope

This document defines detailed functional requirements for the **Rekla Admin Portal** — a web-based operations and management console used by the internal operations team. The portal provides visibility, control, and configuration capabilities across all platform entities: drivers, riders, rides, vehicles, pricing, and promotions.

**Business Payment Model Note:** Ride fare is collected directly by driver from rider (cash/UPI/PhonePe or equivalent), and the platform does not process ride-fare payments in-app. Platform monetization is primarily through driver package purchases and configured business plans.

**White-Label Product Note:** Rekla Admin Portal can be delivered per company as a white-label deployment with company-specific branding and configuration.
**White-Label Scope Boundary Note:** Frontend white-label customization is expected mainly for portal identity and light UI behavior (for example portal/app name, logo, legal/policy links, support contacts, and UI text assets). Business behavior changes (for example base fare, vehicle types, trip-type enablement, and location-based business rules) are driven by individual backend configuration for that deployment. Core major admin operational functionality remains the Rekla baseline unless explicitly changed in signed scope.
**Configuration Independence Note:** Configuration is runtime value-driven only. Configuration records must remain independent and must not be FK-linked to core transactional/domain tables.

**In Scope:** All features of the Admin Portal for v1.2 (MVP).
**Out of Scope:** Rider app, Driver app. Refer to parent BRD for platform-wide exclusions.
**Access:** Web browser (desktop/laptop). Not a mobile app.

---

## 2. Admin Persona

| Field       | Details                                                      |
|-------------|--------------------------------------------------------------|
| Name        | Basic Admin / Lead / Manager / Director / Owner              |
| Goal        | Monitor platform health, support riders and drivers, manage users, and resolve issues quickly |
| Pain Points | Lack of real-time visibility, reliance on manual processes   |
| Device      | Desktop or Laptop browser (Chrome / Edge)                    |
| Tech Level  | Advanced — comfortable with dashboards and data tables       |
| Access      | Role-based (Basic Admin, Lead, Manager, Director, Owner)     |
| Login       | Company email address + password + OTP (2FA)                 |

---

## 3. Functional Requirements

---

### 3.1 Admin Registration & Authentication

**Concept:** Admin accounts are created using a verified **company email address** (e.g., `@rekla.in`). On first registration, the admin's email is verified via OTP, and a password is set. Subsequent logins use email + password, with an OTP as a second factor.

**User Journeys:**

**Registration:**
```
Enter Company Email → System sends OTP to email → Enter OTP → OTP verified
→ Set Password → Account created → Land on Dashboard
```

**Login:**
```
Enter Company Email + Password → System sends OTP to email → Enter OTP → Authenticated → Dashboard
```

**Forgot Password:**
```
Click Forgot Password → Enter Company Email → OTP sent to email
→ Enter OTP → Set New Password → Login
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A01  | The admin portal shall allow new admin accounts to be registered using a company-issued email address only (non-company domains shall be rejected). | Must |
| FR-A02  | On submitting the registration form (name, email), the system shall send a 6-digit OTP to the entered email address for verification. | Must |
| FR-A03  | The OTP shall be valid for 10 minutes. The admin shall be able to request a resend after 30 seconds. A maximum of 3 resend attempts shall be allowed per session. | Must |
| FR-A04  | After successful OTP verification, the admin shall be prompted to set a password. Password must be at least 8 characters and include one uppercase letter, one number, and one special character. | Must |
| FR-A05  | Once the password is set, the account shall be created with **Pending Activation** status until an Owner activates it. | Must |
| FR-A06  | The admin portal login screen shall accept company email and password. After credential validation, a 6-digit OTP shall be sent to the registered email for 2FA verification. | Must |
| FR-A07  | After 5 consecutive failed login attempts, the account shall be locked for 15 minutes. The admin shall see a clear message explaining the lockout and remaining wait time. | Must |
| FR-A08  | The admin shall be able to reset a forgotten password by entering their company email, receiving an OTP, verifying it, and setting a new password. | Must |
| FR-A09  | Admin sessions shall time out after 30 minutes of inactivity. The admin shall be redirected to the login screen with a session-expiry notice. | Must |
| FR-A10  | The admin shall be able to log out explicitly from any page via the profile/account menu. All active sessions for that account shall be terminated on logout. | Must |

---

### 3.2 Portal Navigation & Menu Structure

**Concept:** The admin portal has a persistent left-side navigation menu that provides access to all modules. The active menu item is visually highlighted. Access to menu items is filtered by the admin's assigned role.

**Primary Navigation Menu:**

```
[Rekla Admin Portal]
├── Dashboard            (Home — default on login)
├── Drivers              (Driver management & approval)
├── Riders               (Rider management)
├── Rides                (All rides / trip management)
├── Vehicles             (Vehicle registry)
├── Approvals            (Pending driver & document approvals)
├── Book via Admin       (Admin-initiated ride booking)
├── Promo Codes          (Create / manage promos)
├── Fare & Pricing       (Fare configuration)
├── Analytics            (Charts & reports)
├── Reports Builder      (Custom business reports)
├── Support              (Tickets & dispute resolution)
├── Quote Desk           (Outstation/Package estimate support)
├── Audit Logs           (Activity and action history)
└── Settings             (Admin users, roles, system config)
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A11  | The portal shall display a persistent left-side navigation menu containing all primary sections as listed above. | Must |
| FR-A12  | Menu items shall be visible only to admin roles with appropriate access; unauthorized menu items shall be hidden, not just disabled. Role access shall follow: Basic Admin (Dashboard, Drivers, Vehicles, Approvals, Book via Admin, Rides, Support), Lead (+ Fare & Pricing draft changes), Manager (+ change verification queue, Analytics view, Reports Builder), Director (+ final approval queue for high-impact changes and full Reports Builder access), Owner (all modules including financial views and admin user controls). | Must |
| FR-A13  | The active section shall be highlighted in the navigation menu. Navigating between sections shall not require a full page reload (SPA behaviour). | Should |
| FR-A14  | The top header bar shall display the logged-in admin's name, role badge, and a notification bell icon showing unread platform alerts count. | Must |
| FR-A15  | The notification bell shall display alerts such as: new pending driver approval, document expiry within 7 days, open SOS events, Force End review awaiting action. | Must |
| FR-A16  | Each primary menu item shall display a **badge count** when there are items requiring attention (e.g., "Approvals 3" when 3 pending approvals exist). | Should |

---

### 3.3 Dashboard — Section-wise Overview

**Concept:** The Dashboard is the admin's home screen and the central monitoring hub. It is organized into **four dedicated information sections** (Drivers, Rides, Riders, Vehicles), each showing real-time aggregated KPIs. Additionally, a live map, alert strip, and charts provide operational visibility.

**Dashboard Layout:**
```
[Alert Strip — top banner for urgent items]
[Drivers Section] [Rides Section] [Riders Section] [Vehicles Section]
[Live Map — full width]
[Charts Row: Trip Trend | Package Sales Trend | Ride Type Distribution | Driver Activity]
```

---

#### 3.3.1 Drivers Section

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A17  | The Drivers section of the Dashboard shall display the following KPI tiles: **Total Onboarded Drivers** (all-time approved), **Pending Approval** (awaiting review), **Active Online Now** (currently online), **Engaged on Trip** (currently on a ride), **Suspended** (count), **Rejected** (count). | Must |
| FR-A18  | Each Drivers KPI tile shall be clickable and shall navigate to the Driver Management list pre-filtered to that status. | Must |
| FR-A19  | The Drivers section shall additionally show: **Documents Expiring in 7 Days** count as a warning indicator. | Must |

---

#### 3.3.2 Rides Section

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A20  | The Rides section of the Dashboard shall display the following KPI tiles: **Ongoing Rides Now** (live count), **Completed Today**, **Cancelled Today**, **Total Rides (All-Time)**, and **Direct-Pay Rides Today** (rides paid directly to driver). | Must |
| FR-A21  | The Rides section shall additionally show a breakdown by ride type: **Local**, **Outstation**, and **Package** — showing count of completed rides today for each type. | Should |
| FR-A22  | Each Rides KPI tile shall be clickable and shall navigate to the Ride Management list pre-filtered by that status and date. | Must |
| FR-A23  | The Rides section shall show a **Force End Awaiting Review** counter (rides that were force-ended and require admin review). | Must |

---

#### 3.3.3 Riders Section

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A24  | The Riders section of the Dashboard shall display the following KPI tiles: **Total Registered Riders**, **Active Today** (riders who took at least one ride today), **New Registrations Today**, **Suspended Riders** count. | Must |
| FR-A25  | Each Riders KPI tile shall be clickable and shall navigate to the Rider Management list pre-filtered to that category. | Should |

---

#### 3.3.4 Vehicles Section

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A26  | The Vehicles section of the Dashboard shall display: **Total Registered Vehicles**, **Approved Vehicles**, **Pending Approval**, and a **Type-wise Count** breakdown (e.g., Sedan: 24, Hatchback: 18, SUV: 10, Auto: 5). | Must |
| FR-A27  | The Vehicles section shall show vehicle status counts for each type in a compact summary format (e.g., small bar or count chips per vehicle type). | Should |

---

#### 3.3.5 Alert Strip & Live Map

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A28  | A top alert strip shall display urgent items requiring admin action: e.g., "5 drivers pending approval", "2 SOS events open", "3 Force End rides awaiting review", "4 driver documents expiring in 7 days". | Must |
| FR-A29  | Alert strip items shall be clickable and shall navigate directly to the relevant section/filtered list. | Must |
| FR-A30  | The Dashboard shall include a **Live Map** (full-width below the KPI sections) showing real-time positions of all online drivers (green pins) and active ongoing trips (colour-coded animated route lines per ride type). | Must |
| FR-A31  | Clicking a driver pin on the live map shall show a popover with: driver name, vehicle type, current status (Online / Engaged), ride ID if on a trip, and a **"Watch Ride"** link that opens the Live Ride Monitor for that ride. | Must |
| FR-A32  | Clicking an active trip route line on the live map shall show a popover with: ride ID, rider name, driver name, ride type, trip start time, and a **"Watch Ride"** link to open the Live Ride Monitor. | Must |
| FR-A32A | The live map shall stream driver and trip position updates in **near real-time (≤5 second latency)** using WebSocket or server-sent events; the admin shall not need to manually refresh. | Must |

---

#### 3.3.6 Dashboard Charts

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A33  | The Dashboard shall display a **Daily Trip Trend line chart** showing total trips per day for the last 7 days. | Must |
| FR-A34  | The Dashboard shall display a **Package Sales Trend bar chart** showing platform package-sales revenue per day for the last 7 days. | Must |
| FR-A35  | The Dashboard shall display a **Ride Type Distribution donut chart** showing the proportion of Local, Outstation, and Package rides completed today. | Should |
| FR-A36  | The Dashboard shall display a **Driver Activity bar chart** showing: Online Hours vs. Trips Completed for the top 10 drivers today. | Should |
| FR-A37  | All dashboard charts shall support a period toggle: **Today / Last 7 Days / Last 30 Days**. | Should |
| FR-A38  | The entire Dashboard shall auto-refresh every 30 seconds. A manual "Refresh" button shall also be available. | Must |

---

### 3.4 Driver Management

**User Journey:**
```
Drivers Menu → Driver List → Search / Filter → View Driver Profile → Take Action (Approve / Reject / Suspend / Reactivate / Flag Document)
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A39  | The admin shall be able to view a paginated list of all driver accounts with: name, mobile, registration date, status (Pending / Active / Suspended / Rejected), ride categories enabled, and average rating. | Must |
| FR-A40  | The admin shall be able to filter the driver list by: status, registration date range, ride category (Local/Outstation/Package), city, and rating range. | Must |
| FR-A41  | The admin shall be able to search for a specific driver by name or mobile number.                                | Must     |
| FR-A42  | The admin shall be able to open a driver's full profile page showing: personal details, vehicle details (linked), all uploaded documents with status and expiry dates (with download option), ride history summary, earnings summary, shift history summary, and ratings history. | Must |
| FR-A43  | The admin shall be able to **Approve** a pending driver application; upon approval the driver account becomes Active and the driver receives an in-app and push notification. | Must |
| FR-A44  | The admin shall be able to **Reject** a pending driver application with a mandatory rejection reason; the driver is notified with the reason and can re-apply. | Must |
| FR-A45  | The admin shall be able to **Suspend** an active driver account with a mandatory reason; a suspended driver cannot go online and sees a suspension notice in the app. | Must |
| FR-A46  | The admin shall be able to **Reactivate** a suspended driver account; the driver receives an activation notification. | Must |
| FR-A47  | The admin shall be able to view each driver's document status and expiry dates and send a manual document reminder notification to the driver. | Should |
| FR-A48  | The admin shall be able to mark a specific document as **Re-verification Required**, which prompts the driver to re-upload the document in the app. | Should |
| FR-A49  | The admin shall be able to view a driver's shift history: login sessions, total online hours, trips per shift, and earnings per shift. | Should |
| FR-A50  | The admin shall be able to view a driver's policy violation history and issue a formal warning from the admin side. | Should |
| FR-A51  | The admin shall be able to review **Force End** trips flagged for admin review: view the trip timeline, driver note, location at force-end, and mark as Reviewed or Escalate. | Must |
| FR-A51A | Authorized roles shall be able to send important communications to a single driver or multiple drivers (bulk) via in-app notification, WhatsApp, and SMS for urgent operational instructions or updates. | Must |
| FR-A51B | The driver profile shall display package subscription details including active package name, validity period, usage balance (if applicable), and package expiry date. | Must |
| FR-A51C | The system shall support package-expiry monitoring and allow authorized roles to send expiry reminders to affected drivers individually or in bulk. | Should |
| FR-A51D | Driver package status shall control ride-request eligibility: drivers with active eligible package can receive ride requests; expired/inactive package drivers shall be restricted based on configured business rules. | Must |

---

### 3.5 Rider Management

**Concept:** Rider Management includes both account control and rider engagement workflows. Admins can identify inactive riders and send targeted re-engagement notifications in bulk.

**User Journey:**
```
Riders Menu → Rider List → Search / Filter → View Rider Profile → Take Action (Suspend / Reactivate)
Riders Menu → Filter Inactive Riders → Select Audience → Compose/Select Template → Send via App / WhatsApp / SMS → Track delivery and response
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A52  | The admin shall be able to view a paginated list of all rider accounts with: name, mobile, registration date, status, total trips taken, and total spend. | Must |
| FR-A53  | The admin shall be able to filter the rider list by: status (Active/Suspended), registration date range, and activity level (active in last 7/30 days vs. inactive). | Should |
| FR-A54  | The admin shall be able to search for a specific rider by name or mobile number.                                 | Must     |
| FR-A55  | The admin shall be able to open a rider's full profile page showing: personal details, complete ride history, total spend, promo codes used, referral activity, and ratings given/received. | Must |
| FR-A56  | The admin shall be able to **Suspend** a rider account with a mandatory reason; the suspended rider cannot book rides and sees a suspension notice. | Must |
| FR-A57  | The admin shall be able to **Reactivate** a suspended rider account.                                             | Must     |
| FR-A58  | The admin shall be able to view all promo codes and referral credits applied to a rider's account.               | Should   |
| FR-A58A | The admin shall be able to create rider audience segments based on inactivity (e.g., no rides in 7/15/30/60/90 days), city, last ride type, and total trips. | Must |
| FR-A58B | Authorized roles shall be able to send bulk notifications to selected rider segments through **In-App Push**, **WhatsApp**, and **SMS** channels. | Must |
| FR-A58C | The system shall support message templates for campaign types such as reactivation, promo reminder, referral reminder, and service updates; templates shall support placeholders (e.g., rider name, promo code, expiry date). | Should |
| FR-A58D | The admin shall be able to send a message immediately or schedule it for a specified date/time. | Should |
| FR-A58E | The campaign screen shall show estimated reachable audience counts per channel before sending (eligible, unreachable, opted-out). | Should |
| FR-A58F | The system shall maintain a rider communication log with campaign ID, channel, delivery status, sent timestamp, and engagement outcome (opened/clicked where available). | Must |
| FR-A58G | The system shall enforce consent and policy checks before sending messages (e.g., channel opt-out, do-not-disturb windows, regulatory constraints). | Must |
| FR-A58H | Suspended riders shall not receive promotional campaigns; only compliance or account-status messages may be sent to suspended accounts. | Must |

---

### 3.6 Ride Management

**Concept:** Covers all ride types — Local, Outstation, and Package. Admins can monitor, filter, investigate, and live-track any ride across its full lifecycle.

**User Journey:**
```
Rides Menu → Ride List → Filter by Type / Status / Date → View Ride Detail → Watch Live (for ongoing rides) → Intervene if Required
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A59  | The admin shall be able to view a paginated list of all rides with: ride ID, date/time, ride type (Local/Outstation/Package), rider name, driver name, pickup, drop, fare, and status (Scheduled / In Progress / Completed / Cancelled / Force Ended). | Must |
| FR-A60  | The admin shall be able to filter rides by: status, ride type, date range, driver, rider, and collection mode (cash/direct UPI/direct digital). | Must |
| FR-A61  | The admin shall be able to view a full **Ride Detail** page including: route map (start to end trace), full event timeline (accepted, arrived, started, ended, each OTP/KM entry), fare breakdown/reference fare, promo applied (if applicable), collection mode, and cancellation/force-end details if applicable. | Must |
| FR-A62  | For **Outstation** and **Package** rides, the detail page shall show the odometer KM readings captured at start and end, and the calculated distance. | Must |
| FR-A63  | The admin shall be able to view the SOS event log for any ride where the rider triggered the emergency feature.  | Must     |
| FR-A64  | The admin shall be able to record disputed-fare outcomes with a reason note and optional reference-fare correction for reporting purposes; all changes shall be logged in the audit trail. | Should |
| FR-A65  | As the platform does not collect ride fare, the admin shall not process gateway refunds for ride payments; instead, admin shall record a fare dispute outcome note and resolution guidance in the ride/support record. | Must |
| FR-A66  | The admin shall be able to view and act on **Force End Review** items: rides where the driver invoked Force End; the admin reviews the reason and marks as Acknowledged or Escalated. | Must |

---

#### 3.6.1 Live Ride Monitoring

**Concept:** For any ride currently **In Progress**, the admin can open a dedicated **Live Ride Monitor** — a real-time, full-screen view that streams the vehicle's current GPS position, navigation route, live status events, and key ride metrics with no manual refresh required. This enables the operations team to watch an individual ride end-to-end without any delay.

**User Journey:**
```
Ongoing Rides List → Click "Watch Live" on any In Progress ride
   OR Dashboard Live Map → Click ride pin/route → "Watch Ride"
→ Live Ride Monitor opens → Real-time map + status panel
→ Admin monitors until ride completes or navigates away
```

**Live Ride Monitor Screen Layout:**
```
[Ride Header Bar: Ride ID | Type | Rider | Driver | Status badge | Elapsed Time]
[Live Map — large, dominant]        [Status & Info Panel — right side]
  • Driver marker (moving pin)         • Current GPS coordinates
  • Planned route (grey)               • Current speed (km/h)
  • Travelled path (colour trace)      • Distance covered / remaining
  • Pickup pin                         • Estimated Time of Arrival (ETA)
  • Drop pin                           • Fare meter (running estimate)
  • Driver heading indicator           • Live Event Log (timestamped)
```

| ID       | Requirement                                                                                                      | Priority |
|----------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A67A  | The admin shall be able to open a **Live Ride Monitor** for any ride with status **In Progress** via a "Watch Live" button in the ride list row and in the ride detail page. | Must |
| FR-A67B  | The Live Ride Monitor shall display the driver's **current GPS location** as a moving marker on a map, updated in near real-time with a maximum latency of 5 seconds. | Must |
| FR-A67C  | The Live Ride Monitor map shall display: the **planned navigation route** (grey line), the **path already travelled** (coloured trace), the **pickup pin**, the **drop pin**, and the driver's **current heading direction**. | Must |
| FR-A67D  | The Live Ride Monitor shall display the following live metrics in a side panel, updated continuously: **current GPS coordinates** (lat/long), **current speed** (km/h), **distance covered** so far, **estimated remaining distance**, and **estimated time of arrival (ETA)** at the drop location. | Must |
| FR-A67E  | The Live Ride Monitor shall display a **running fare estimate** that updates as the trip progresses (based on distance and elapsed time). | Should |
| FR-A67F  | The Live Ride Monitor shall display a **Live Event Log** — a timestamped feed of all status changes for that ride as they happen: e.g., Accepted, Driver En Route, Driver Arrived, Trip Started, OTP Verified (Local), KM Capture (Outstation/Package), SOS Triggered, Trip Ended. | Must |
| FR-A67G  | The Live Ride Monitor shall display the **elapsed trip duration** as a live running timer from the moment the trip was started. | Must |
| FR-A67H  | If the rider triggers an **SOS alert** during the monitored ride, the Live Ride Monitor shall immediately highlight the event with a prominent visual alert (red banner) and sound a notification in the admin's browser. | Must |
| FR-A67I  | The admin shall be able to open the Live Ride Monitor for **multiple simultaneous rides** in separate browser tabs without degradation in tracking accuracy. | Should |
| FR-A67J  | When a monitored ride reaches **Completed** or **Cancelled / Force Ended** status, the Live Ride Monitor shall show a completion banner with the final fare, total distance, and trip duration; live tracking shall stop automatically. | Must |
| FR-A67K  | The Live Ride Monitor shall show a **connectivity status indicator**: if the data stream is interrupted (driver app offline / network loss), the panel shall show a "Signal Lost" warning with the timestamp of the last known position. | Must |
| FR-A67L  | The admin shall be able to access the **full ride detail page** (static history view) directly from the Live Ride Monitor via a link, without losing the live view. | Should |

---

### 3.7 Vehicle Management

**Concept:** Each driver registers one or more vehicles during onboarding. This section provides a standalone registry of all registered vehicles with approval workflow, type-based classification, and status tracking.

**User Journey:**
```
Vehicles Menu → Vehicle List → Filter by Type / Status → View Vehicle Detail → Approve / Reject Documents → Manage Status
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A67  | The admin shall be able to view a paginated list of all registered vehicles with: vehicle number plate, type (Sedan/Hatchback/SUV/Auto/Other), make & model, year, linked driver name, document status, and current approval status. | Must |
| FR-A68  | The admin shall be able to filter the vehicle list by: type, approval status (Pending/Approved/Rejected/Suspended), and registration date range. | Must |
| FR-A69  | The admin shall be able to search vehicles by vehicle number plate or linked driver name.                        | Must     |
| FR-A70  | The admin shall be able to open a **Vehicle Detail** page showing: full vehicle information, all uploaded documents (RC, insurance, PUC — with download links and expiry dates), and the linked driver profile link. | Must |
| FR-A71  | The admin shall be able to **Approve** a vehicle's documents, marking the vehicle as verified and eligible for rides. | Must |
| FR-A72  | The admin shall be able to **Reject** a vehicle's document submission with a mandatory rejection reason; the driver is notified to re-upload. | Must |
| FR-A73  | The admin shall be able to **Suspend** a specific vehicle from operation (e.g., expired insurance) without suspending the driver's account. | Should |
| FR-A74  | The dashboard Vehicles section shall reflect the type-wise counts sourced from this vehicle registry in real time. | Must |
| FR-A75  | The admin shall be able to configure the list of allowed **Vehicle Types** (e.g., add or rename categories) from this section. | Should |

---

### 3.8 Approvals Management

**Concept:** A dedicated consolidated view for all items currently awaiting admin action — new driver applications, re-submitted documents, vehicle document re-verifications. This removes the need to navigate to individual sections to find pending items.

**User Journey:**
```
Approvals Menu → View Pending Items (tabbed: Drivers | Documents | Vehicles) → Review → Approve / Reject with Reason
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A76  | The Approvals screen shall present pending items in three tabs: **Driver Applications** (new registrations), **Document Re-verifications** (driver re-uploads), and **Vehicle Documents** (vehicle document submissions). | Must |
| FR-A77  | Each tab shall show a count badge of pending items (e.g., "Driver Applications (7)"). | Must |
| FR-A78  | The admin shall be able to open any pending item and perform **Approve** or **Reject** actions directly from the Approvals screen without navigating to the individual driver/vehicle page. | Must |
| FR-A79  | Rejection shall require a mandatory reason; the reason shall be sent to the driver as an in-app notification. | Must |
| FR-A80  | The admin shall be able to **Approve or Reject multiple items in bulk** by selecting multiple rows and applying a batch action. | Should |
| FR-A81  | Approved or rejected items shall be removed from the pending list immediately; a history of past approvals/rejections shall be available with filters by date and admin. | Should |
| FR-A82  | The Approvals section badge count shall match the Dashboard alert strip pending approval counts at all times. | Must |

---

### 3.9 Book via Admin

**Concept:** Admin can manually book a ride on behalf of a registered rider — used in scenarios such as corporate bookings, customer support escalations, or rider-assisted bookings via phone/call centre. Rekla shall support a unified dispatch provisioning model where bookings created from rider app and bookings created from admin/phone follow the same backend assignment pipeline.

**User Journey:**
```
Book via Admin Menu → Select Rider → Enter Pickup & Drop → Select Ride Type
→ System calculates fare estimate → Backend auto-assignment by location and ride type
→ If no acceptance / urgent need, support sends targeted request to location drivers
→ Confirm Booking → Ride dispatched
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A83  | The admin shall be able to book a ride on behalf of any registered rider by searching for the rider by name or mobile number. | Must |
| FR-A84  | The admin shall be able to enter pickup and drop locations using the map or by typing an address.                | Must     |
| FR-A85  | The admin shall be able to select the ride type: **Local**, **Outstation**, or **Package**.                      | Must     |
| FR-A86  | For **Package** rides, the admin shall be able to select the applicable package from the rider's active packages. | Must |
| FR-A87  | The system shall display a fare estimate before confirming the booking.                                          | Must     |
| FR-A88  | The admin shall be able to select a specific available driver from a list of nearby online drivers, or use **Auto-Assign** to let the system dispatch to the nearest driver. | Should |
| FR-A89  | On confirmation, the ride shall be dispatched to the selected or auto-assigned driver, identical to a rider-initiated booking flow. | Must |
| FR-A90  | Admin-initiated bookings shall be tagged as "Booked via Admin" in the ride record and visible in the Ride Management list and ride detail page. | Must |
| FR-A91  | The admin shall be able to apply a promo code to an admin-booked ride at the time of booking.                    | Should   |
| FR-A91A | The platform shall treat booking source as one of: **Rider App**, **Admin Portal**, or **Phone/Call Centre**; all sources shall enter the same dispatch orchestration pipeline with source tagging for reporting. | Must |
| FR-A91B | For **Local**, **Outstation**, and **Package** rides, the backend dispatch job shall attempt automatic assignment to eligible nearby drivers using ride type, location proximity, driver availability, and package eligibility rules. | Must |
| FR-A91C | If automatic assignment is not successful within configurable thresholds (e.g., no driver found or repeated declines), support users shall be able to trigger a **Targeted Driver Request** to drivers in the relevant pickup zone/location cluster. | Must |
| FR-A91D | Targeted Driver Request shall allow filters by ride type, distance radius/zone, driver availability, and driver package eligibility, and shall send request notifications to selected drivers with acceptance timeout tracking. | Must |
| FR-A91E | The system shall maintain dispatch-attempt logs for each booking (auto-attempts, manual targeted requests, accepted/rejected/no-response outcomes) so support can audit and intervene quickly. | Must |
| FR-A91F | When an urgent support intervention is used, the booking record shall show an **Intervention Required** flag and capture the acting support admin, reason, and timestamp. | Should |
| FR-A91G | For bookings created via phone/call-centre, the system shall capture call-handling metadata: booking source = Phone/Call Centre, support agent (creator), created timestamp, and optional call reference ID, and show these in ride details and reports. | Must |
| FR-A91H | The portal shall provide daily/weekly metrics for call-assisted bookings: total created, converted to assigned rides, cancelled, and unresolved/no-driver cases, with breakdown by support agent. | Must |

---

### 3.10 Fare & Pricing Configuration

**User Journey:**
```
Fare & Pricing Menu → Select Ride Type → Edit Parameters → Save & Publish → Changes apply to new bookings immediately
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A92  | The admin shall be able to view and edit fare parameters per ride type: base fare, per-km rate, per-minute rate, and minimum fare. | Must |
| FR-A93  | The admin shall be able to configure driver package plans (price, validity, eligible ride categories, and activation rules) that govern ride-request access. | Must |
| FR-A94  | The admin shall be able to configure the **Default Free Promo Code** for each ride category that is automatically applied to new drivers during their first X rides.| Must |
| FR-A95  | The admin shall be able to configure cancellation fee amounts for each cancellation window (post-acceptance, post-arrival). | Must |
| FR-A96  | The admin shall be able to configure the free cancellation window duration (in minutes).                         | Must     |
| FR-A97  | The admin shall be able to activate or deactivate surge pricing for a defined time window with a configurable surge multiplier. | Must |
| FR-A98  | All fare configuration changes shall take effect for new bookings immediately upon saving; in-progress trips shall not be affected. | Must |
| FR-A99  | The admin shall be able to view a history log of all fare configuration changes: field changed, old value, new value, acting admin, and timestamp. | Should |
| FR-A100 | The admin shall be able to configure fallback behavior when a driver has no active package (e.g., block rides, grace-period rides, or restricted-category rides) as per business policy. | Must |

---

### 3.11 Promo Code Management

**User Journey:**
```
Promo Codes Menu → Create Promo → Set Parameters → Activate → Monitor Usage
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A101 | The admin shall be able to create a new promo code with: code string, discount type (flat / percentage), discount value, max discount cap, applicable ride types, validity dates, and maximum total redemption count. | Must |
| FR-A102 | The admin shall be able to set a per-user usage limit for a promo code.                                          | Must     |
| FR-A103 | The admin shall be able to view all promo codes with: status (Active / Inactive / Expired), total uses, and remaining uses. | Must |
| FR-A104 | The admin shall be able to activate or deactivate any promo code at any time.                                    | Must     |
| FR-A105 | The admin shall be able to view per-code usage analytics: total redemptions, total discount given, rider list. | Should |
| FR-A106 | The admin shall be able to manage referral reward configurations: reward value per referee and referrer, and eligibility rules. | Must |
| FR-A107 | The admin shall be able to designate a promo code as a **Default Free Promo** for new drivers (auto-applied to their first rides). | Must |

---

### 3.12 Analytics & Charts

**Concept:** A dedicated Analytics section for deep-dive reporting and visualisation. Covers driver performance, ride trends, package-sales revenue, location-based demand, and rider acquisition — all with interactive charts and date range filters.

**User Journey:**
```
Analytics Menu → Select Analytics Category → Apply Date Range / Filters → View Charts + Data Table → Export
```

---

#### 3.12.1 Ride Analytics

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A108 | The Ride Analytics view shall show a **Daily Trips line chart** (total / by ride type) for the selected period (configurable: 7 / 30 / 90 / Custom days). | Must |
| FR-A109 | The Ride Analytics view shall show a **Ride Type Distribution donut chart** with counts for Local, Outstation, and Package rides. | Must |
| FR-A110 | The Ride Analytics view shall show a **Cancellation Rate trend line chart** showing daily cancellation % over the selected period. | Should |
| FR-A111 | The admin shall be able to export the underlying ride data table (ride ID, date, type, driver, rider, fare, status) as a CSV. | Must |

---

#### 3.12.2 Package Revenue Analytics

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A112 | The Package Revenue Analytics view shall show a **Daily Package Sales bar chart** showing total package purchase value per day. | Must |
| FR-A113 | The Package Revenue Analytics view shall show a **Revenue by Package Type stacked bar chart** for the selected period. | Should |
| FR-A114 | The Package Revenue Analytics view shall provide a summary table: Gross Package Revenue, Discounts Given on Packages (if any), Taxes (if applicable), and Net Platform Revenue for the selected period. | Must |

---

#### 3.12.3 Driver Analytics

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A115 | The Driver Analytics view shall show a **Driver Performance leaderboard table** for the selected period: driver name, total trips, total earnings, acceptance rate, cancellation rate, and average rating — sortable by each column. | Must |
| FR-A116 | The Driver Analytics view shall show a **Driver Activity bar chart**: total online hours vs. trips completed for the top N drivers. | Should |
| FR-A117 | The Driver Analytics view shall show a **New Driver Registrations trend line** (daily count) for the selected period. | Should |
| FR-A118 | The Driver Analytics view shall allow filtering by ride category (Local / Outstation / Package) to see performance per category. | Should |

---

#### 3.12.4 Rider Analytics

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A119 | The Rider Analytics view shall show a **New Rider Registrations line chart** for the selected period.            | Should   |
| FR-A120 | The Rider Analytics view shall show a **Rider Retention** metric: percentage of riders who took more than one trip within the selected period. | Should |
| FR-A121 | The Rider Analytics view shall show **First-Trip Conversion Rate**: registered riders who completed at least one ride within 7 days of registration. | Should |

---

#### 3.12.5 Location-Based Analytics

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A122 | The Location Analytics view shall show a **Demand Heatmap** overlay on a map, visualising ride pickup density by geographic area for the selected time period. | Must |
| FR-A123 | The Location Analytics view shall show a **Driver Availability Heatmap** showing where online drivers are concentrated during a selected time window. | Should |
| FR-A124 | The Location Analytics view shall identify and display the **Top 10 Pickup Zones** and **Top 10 Drop Zones** (by ride count) for the selected period in a ranked list. | Should |
| FR-A125 | The admin shall be able to filter the location analytics by ride type (Local / Outstation / Package) and by time window (Morning / Afternoon / Evening / Night). | Should |

---

#### 3.12.6 Export & Scheduled Reports

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A126 | The admin shall be able to export any analytics view as a **CSV file** (raw data) or **PDF** (chart + summary). | Must |
| FR-A127 | The admin shall be able to schedule a report (e.g., daily package-sales summary, weekly driver performance) to be emailed automatically to specified admin email addresses. | Should |

---

### 3.13 Support & Dispute Resolution

**Concept:** This section covers the day-to-day support operations handled by Rekla support staff, telecallers, and other operational admins. Basic Admin is the frontline role for answering rider and driver queries using the information available in the admin portal.

**User Journey:**
```
Support Menu → View Open Tickets → Assign to Self → Investigate → Add Notes → Resolve → Close
```

**Support Query Types (Implementation Reference):**

1. Rider booking issue
2. Rider cancellation issue
3. Rider login/account issue
4. Driver onboarding or profile approval issue
5. Driver package purchase / expiry / activation issue
6. Vehicle approval / document issue
7. Ride dispute or fare reference issue
8. Driver no-ride / ride-request eligibility issue
9. SOS / safety escalation
10. Payment collection clarification (direct pay to driver)
11. Promo / referral clarification
12. Technical app issue (rider or driver app)
13. Driver package refund request (no ride assigned complaint)

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A128 | The admin shall be able to view a list of all support tickets raised by riders and drivers with: ticket ID, user name, user type, subject, status (Open / In Progress / Resolved / Closed), and creation date. | Must |
| FR-A129 | The admin shall be able to filter support tickets by: status, user type (rider/driver), and date range.          | Should   |
| FR-A130 | The admin shall be able to open a ticket to view: user profile, linked ride detail (if any), full description, and all prior admin notes and actions. | Must |
| FR-A131 | The admin shall be able to add internal notes (not visible to the user) and update the ticket status.            | Must     |
| FR-A132 | The admin shall be able to send a resolution message directly to the user (rider or driver) from within the ticket interface. | Should |
| FR-A133 | The admin shall be able to log and track fare/payment disputes from within a support ticket, but shall not process in-app ride-payment refunds as ride fare is collected directly by driver. | Must |
| FR-A134 | The admin shall be able to view and manage **SOS Alerts**: user, trip, GPS location at trigger time, current status (Open / In Contact / Resolved), and escalation history. | Must |
| FR-A134A | **Basic Admin** shall have sufficient access to respond to routine rider and driver queries by viewing linked rider profile, driver profile, trip details, package status, document status, support history, and admin notes, subject to role permissions. | Must |
| FR-A134B | If a rider or driver query cannot be resolved by Basic Admin, the system shall allow escalation to higher roles with internal notes, status handoff, and assignee tracking. | Must |
| FR-A134C | Every support ticket shall require a **query type classification** selected from the Support Query Types list so the issue can be routed, reported, and filtered consistently. | Must |
| FR-A134D | The system shall support ticket routing/SLA assignment rules based on query type, priority, and user type (rider/driver), with unresolved high-risk categories such as SOS or safety issues flagged for immediate escalation. | Should |
| FR-A134E | For query type **Driver package refund request (no ride assigned complaint)**, Basic Admin shall be able to create and manage a structured refund-claim case linked to package purchase ID and driver account. | Must |
| FR-A134F | The refund-claim case view shall show evidence fields: package activation timestamp, package validity window, driver online duration during validity, eligible ride categories, dispatch attempts, ride requests sent to driver, acceptance/decline/no-response counts, and geo-zone coverage at the time of requests. | Must |
| FR-A134G | Refund-claim cases shall support workflow statuses: **Open**, **Evidence Collected**, **Pending Lead Review**, **Pending Manager Verification**, **Pending Director Approval**, **Approved**, **Rejected**, **Resolved with Alternative Compensation**. | Must |
| FR-A134H | Support users shall be able to record alternative resolutions (e.g., package extension, goodwill credits, priority dispatch window) when refund is not approved, with reason and customer acknowledgement note. | Should |

---

### 3.14 Admin User & Access Management

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A135 | The portal shall support Role-Based Access Control (RBAC) with at least the following admin hierarchy: **Basic Admin**, **Lead**, **Manager**, **Director**, and **Owner**. | Must |
| FR-A136 | Access shall be hierarchical by default: Owner > Director > Manager > Lead > Basic Admin. Higher roles inherit lower-role permissions unless explicitly restricted. | Must |
| FR-A137 | **Basic Admin** shall have operational permissions for day-to-day execution: approve/reject driver profiles, approve/reject vehicle documents, manage driver-vehicle linking, create bookings via **Book via Admin**, monitor driver live activity, send important messages to one or many drivers, check driver package details and expiry information, and answer routine rider and driver support queries from the Support module. | Must |
| FR-A138 | **Basic Admin** shall not have permission to modify pricing policies, per-km reference rates, promo strategy, package definitions, or confidential financial dashboards. | Must |
| FR-A139 | **Lead** shall include all Basic Admin permissions, and additionally shall be able to create high-impact change requests (draft proposals) for driver rules/policies, per-km rates by vehicle/trip, promo changes, and package model changes. Lead cannot publish these changes directly. | Must |
| FR-A140 | **Lead** shall not have access to Owner-only confidential financial views (full package-revenue ledger, strategic business financial exports) unless explicitly granted by Owner. | Must |
| FR-A141 | **Manager** shall include all Lead permissions, and shall act as the **Verifier** for high-impact configuration requests created by Lead. Manager can approve for Director review or reject back to Lead with comments; Manager cannot publish directly. | Must |
| FR-A142 | **Manager** shall be able to create and schedule operational and performance reports, and view analytics/performance dashboards, but shall not access Owner-only secret financial or strategic business data. | Must |
| FR-A143 | **Owner** shall have complete platform visibility including total package revenue, tax summaries (if configured), confidential business KPIs, strategic model data, and all operational/financial dashboards and exports. | Must |
| FR-A144 | Owner shall be able to create admin users, assign or change roles (Basic Admin / Lead / Manager / Director / Owner), activate/deactivate accounts, and review login history. | Must |
| FR-A145 | All admin actions (approvals, rejections, policy changes, promo edits, fare/reference-rate edits, dispute resolutions, bookings, force-end reviews, and role changes) shall be recorded in an immutable audit trail with acting admin ID, role, action type, target entity, old value, new value, and timestamp. | Must |
| FR-A146 | Audit logs shall be viewable by Owner and optionally by Director/Manager (read-only, if enabled by Owner); export to CSV shall be supported with date and actor filters. | Must |

**High-Impact Configuration Workflow (Maker -> Verifier -> Approver):**

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A147 | High-impact changes shall follow a mandatory 3-step workflow: **Lead creates** -> **Manager verifies** -> **Director approves** -> system publishes. Direct publish by Lead or Manager shall be blocked. | Must |
| FR-A148 | High-impact changes include at minimum: per-km/base reference fare/surge changes, promo strategy changes, package model/pricing changes, and driver policy/rule changes. | Must |
| FR-A149 | A Lead-submitted request shall move to status **Pending Manager Verification** with full change diff (old value vs new value), rationale, impact note, and effective time. | Must |
| FR-A150 | Manager verification outcome shall be either **Verified** (forward to Director) or **Rejected to Lead** with mandatory comments. | Must |
| FR-A151 | Director final decision shall be either **Approved for Publish** or **Rejected** with mandatory reason; only Director approval can activate the change in production. | Must |
| FR-A152 | Every workflow transition shall be captured in immutable audit logs with actor role, timestamp, decision note, and value-level diff. | Must |

**Role Permission Matrix (Implementation Reference):**

| Capability / Module | Basic Admin | Lead | Manager | Director | Owner |
|---------------------|-------------|------|---------|----------|-------|
| Driver profile approve/reject | Yes | Yes | Yes | Yes | Yes |
| Vehicle document approve/reject | Yes | Yes | Yes | Yes | Yes |
| Driver-vehicle link management | Yes | Yes | Yes | Yes | Yes |
| Monitor live driver activity | Yes | Yes | Yes | Yes | Yes |
| Send important message to driver(s) | Yes | Yes | Yes | Yes | Yes |
| View driver package details and expiry | Yes | Yes | Yes | Yes | Yes |
| Rider reactivation campaigns (Push/WhatsApp/SMS) | Yes | Yes | Yes | Yes | Yes |
| Book via Admin (create booking) | Yes | Yes | Yes | Yes | Yes |
| Promo/package/policy high-impact request creation | No | Yes | Yes | Yes | Yes |
| High-impact change verification | No | No | Yes | Yes | Yes |
| High-impact final approval and publish | No | No | No | Yes | Yes |
| Package purchase verification and activation | No | Yes | Yes | Yes | Yes |
| Package cancellation/reversal approval | No | No | Yes | Yes | Yes |
| Package refund claim case creation (no-ride complaint) | Yes | Yes | Yes | Yes | Yes |
| Package refund recommendation | No | Yes | Yes | Yes | Yes |
| Package refund verification | No | No | Yes | Yes | Yes |
| Package refund final approval | No | No | No | Yes | Yes |
| Package sales statement export | No | No | No | Optional | Yes |
| Custom report creation (Report Builder) | No | No | Yes | Yes | Yes |
| Custom report schedule/share | No | No | Yes | Yes | Yes |
| End-to-end business report visibility | No | No | Yes | Yes | Yes |
| Analytics dashboards (driver/ride/location performance) | No | Yes | Yes | Yes | Yes |
| Scheduled analytics reports | No | Yes | Yes | Yes | Yes |
| Package revenue and financial business views | No | No | No | No | Yes |
| Admin user/role management | No | No | No | No | Yes |
| Immutable audit log view/export | No | No | Optional read-only | Optional read-only | Yes |

**Role Impact on Business Operations:**

| Role | Business Impact |
|------|-----------------|
| Basic Admin | Acts as frontline support and operations staff by handling rider/driver queries, driver/vehicle approvals, dispatch support, live monitoring, and urgent communications. Directly reduces onboarding delays, unresolved support load, and missed trips. |
| Lead | Acts as maker for high-impact operational changes (policy/rates/promo/package proposals), ensuring domain-grounded change requests are initiated with business context. |
| Manager | Acts as verifier, ensuring proposed high-impact changes are operationally correct, measurable, and safe before escalation to Director. |
| Director | Acts as final approver for high-impact production changes, balancing risk, business outcomes, and governance before release. |
| Owner | Controls business confidentiality and financial truth with full package-revenue and strategic KPI visibility. Directly drives top-level decisions on profitability, expansion, and long-term business model direction. |

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A153 | The system shall enforce permissions exactly as defined in the Role Permission Matrix; actions outside role scope shall return a clear "Access Denied" message and be logged as denied attempts. | Must |
| FR-A154 | Any change to role permissions (matrix update) shall require Owner approval and shall be versioned in a permission policy history log with effective timestamp. | Must |

---

### 3.15 Driver Package Subscription & Sales Operations

**Concept:** This section governs platform monetization via **driver package purchases**. It is separate from rider fare collection. Riders pay drivers directly for rides, while drivers pay the platform for package access.

**User Journey:**
```
Driver buys package -> Purchase event created -> Payment verification/capture
-> Package activated on success -> Driver becomes eligible for rides
-> Expiry reminders -> Renewal / expiry action based on policy
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A155 | The admin shall be able to view a complete list of driver package purchases with: purchase ID, driver, package name, amount, payment mode, payment reference, purchase time, status, activation time, expiry time. | Must |
| FR-A156 | Package purchase status shall support at least: **Initiated**, **Pending Verification**, **Payment Failed**, **Active**, **Expired**, **Cancelled**. | Must |
| FR-A157 | The admin shall be able to search and filter package purchases by driver, package type, status, date range, and payment reference ID. | Must |
| FR-A158 | For manually verified payments (e.g., direct transfer/UPI evidence), authorized roles shall be able to mark payment as verified and activate the package with mandatory remarks and proof reference. | Must |
| FR-A159 | Package activation shall immediately update driver ride-request eligibility according to package rules (categories enabled, validity window, and restrictions). | Must |
| FR-A160 | The system shall generate automated expiry reminders for packages (e.g., T-3 days, T-1 day, day of expiry) and allow admin-triggered manual reminders. | Should |
| FR-A161 | The system shall support configurable expiry behavior: block rides immediately, grace period, or limited-category access after expiry. | Must |
| FR-A162 | The admin shall be able to view package sales analytics by period: total packages sold, active subscriptions, renewals, expiry count, and package-wise revenue contribution. | Must |
| FR-A163 | The system shall maintain an immutable package transaction log containing purchase events, verification actions, activation/deactivation events, actor, timestamp, and reason notes. | Must |
| FR-A164 | Any package cancellation/reversal after activation shall require maker-verifier-approver flow and capture the financial/business reason in audit logs. | Must |
| FR-A165 | Owner and authorized finance roles shall be able to export package-sales statements for accounting reconciliation (period-wise and package-wise). | Must |
| FR-A166 | The system shall support configurable **Package Refund Policy** for no-ride complaints, including eligibility thresholds, pro-rata rules, excluded scenarios, and maximum claim window from package activation date. | Must |
| FR-A167 | Refund eligibility for no-ride complaints shall be computed using objective metrics (minimum online hours met, dispatch attempts sent, ride offers received, driver response behaviour, and serviceable zone participation). | Must |
| FR-A168 | Package refund decision workflow shall be: **Basic Admin logs case** -> **Lead review/recommendation** -> **Manager verification** -> **Director final approval/rejection**; Owner may override in exceptional cases. | Must |
| FR-A169 | On refund approval, the system shall support settlement modes as configured by finance policy (original payment reversal where supported, or manual credit note/adjustment entry), with mandatory transaction reference capture. | Must |
| FR-A170 | On refund rejection, the system shall store the rejection reason code and communicate the resolution to the driver with evidence summary and next steps. | Must |
| FR-A171 | The system shall prevent duplicate refund claims for the same package purchase unless a reopened case is explicitly approved by Director/Owner with reason. | Must |

---

### 3.16 Audit & Activity Logs

**Concept:** Every meaningful action in the portal shall be auditable with actor identity, timestamp, and before/after values so business decisions and support actions are fully traceable.

**User Journey:**
```
Audit Logs Menu -> Select Entity/Module -> Filter by Date/Actor/Action -> View Full Activity Timeline -> Export
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A172 | The portal shall log all major activities with immutable records including at minimum: action type, module/entity, record ID, actor user ID, actor role, action timestamp, and source (web/admin/phone workflow). | Must |
| FR-A173 | For update actions, the audit log shall store both **old value** and **new value** for changed fields, plus optional change reason/comment. | Must |
| FR-A174 | For approval/rejection flows (driver, vehicle, package, high-impact configs, refund claims), the log shall capture who raised, who reviewed, who approved/rejected, decision note, and timestamps for each stage. | Must |
| FR-A175 | For support and call-centre workflows, the log shall capture who created the ticket/booking, who updated it, who closed it, and each status transition timestamp. | Must |
| FR-A176 | The portal shall provide module-level activity timelines (Drivers, Riders, Rides, Vehicles, Packages, Support, Configs) and record-level drill-down history. | Must |
| FR-A177 | Audit Logs shall support filtering by date range, actor, role, module, action type, and source; export to CSV shall be supported for authorized roles. | Must |
| FR-A178 | The system shall provide operational reports for support leadership: calls handled, rides created from calls, conversion to completed rides, and agent-wise action counts. | Should |

---

### 3.17 Outstation & Package Quote Desk

**Concept:** A dedicated support menu for quickly answering customer queries for Outstation and Package rides with transparent estimate communication before booking.

**User Journey:**
```
Quote Desk Menu -> Select Outstation/Package -> Enter pickup, drop, expected hours/days, approximate KM
-> System computes estimate -> Share quote details with customer -> Optional convert to booking
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A179 | The admin portal shall provide a dedicated **Quote Desk** menu for Outstation and Package ride inquiry handling. | Must |
| FR-A180 | Quote Desk shall capture and display mandatory communication fields: pickup point, drop point, estimated hours, estimated days (if multi-day), approximate KM, ride type, and estimated amount range. | Must |
| FR-A181 | The estimate response shall show a clear breakdown: base/reference amount, per-km/per-hour components (as applicable), allowances/extra charges, and a disclaimer that final fare may vary by actual usage and policy. | Must |
| FR-A182 | Support users shall be able to save quote history with customer identifier (name/mobile), quoted by (admin), quoted timestamp, and quote validity period. | Should |
| FR-A183 | Support users shall be able to convert an accepted quote directly into a booking via Book via Admin while preserving quote reference and source = Phone/Support. | Must |
| FR-A184 | Quote Desk shall support region/city policy variations and applicable package/outstation rules so communicated estimates match configured business logic. | Must |

---

### 3.18 Custom Reports & Report Builder

**Concept:** Director/Owner and authorized management users shall be able to create custom reports for end-to-end ride business visibility using flexible filters, groupings, and metrics.

**User Journey:**
```
Reports Builder Menu -> Select Data Domain -> Pick Filters/Dimensions/Metrics
-> Preview Result -> Save Report Template -> Run/Export/Schedule
```

| ID      | Requirement                                                                                                      | Priority |
|---------|------------------------------------------------------------------------------------------------------------------|----------|
| FR-A185 | The portal shall provide a **Reports Builder** module to generate custom reports across major domains: Drivers, Riders, Rides, Packages, Support, Dispatch, and Revenue. | Must |
| FR-A186 | Reports Builder shall support customizable filters including at minimum: date/day range, city/zone, ride type (Local/Outstation/Package), booking source (App/Admin/Phone), driver, rider, package type/status, and support query type. | Must |
| FR-A187 | Reports Builder shall support group-by dimensions such as: day/week/month, driver, ride type, package type, city/zone, support agent, and booking source. | Must |
| FR-A188 | Reports Builder shall support key metrics including: rides created/completed/cancelled, dispatch success rate, driver acceptance rate, no-driver cases, call-assisted bookings, package sales, package expiry/renewal, refund claims, and support resolution SLA. | Must |
| FR-A189 | Authorized users shall be able to save report templates, clone existing templates, and share templates with selected roles/users. | Should |
| FR-A190 | The portal shall support report output formats: table view, chart view, CSV export, and PDF export with report metadata (generated by, generated at, applied filters). | Must |
| FR-A191 | The portal shall support scheduled report delivery (daily/weekly/monthly) to configured recipients, with role-based restrictions on confidential report contents. | Must |
| FR-A192 | Director and Owner shall have end-to-end business report visibility across operational and package-revenue domains; access to sensitive fields shall follow role policy configuration. | Must |

---

## 4. Admin-Specific Non-Functional Requirements

| ID       | Category      | Requirement                                                                                      |
|----------|---------------|--------------------------------------------------------------------------------------------------|
| NFR-A01  | Performance   | The dashboard shall load within 4 seconds on a standard broadband connection.                    |
| NFR-A02  | Performance   | The dashboard live map shall update driver and trip positions every 10 seconds.                  |
| NFR-A02A | Performance   | The **Live Ride Monitor** shall stream individual ride position updates with a maximum end-to-end latency of **5 seconds** from driver GPS event to admin screen render. |
| NFR-A02B | Real-time     | The Live Ride Monitor shall use a persistent WebSocket or server-sent events (SSE) connection; polling is not acceptable for individual ride tracking. |
| NFR-A03  | Performance   | Analytics report generation for periods up to 90 days shall complete within 10 seconds.         |
| NFR-A04  | Performance   | CSV exports for up to 10,000 records shall complete within 15 seconds.                           |
| NFR-A05  | Security      | The admin portal shall enforce HTTPS on all pages; plain HTTP access shall redirect to HTTPS.    |
| NFR-A06  | Security      | Admin accounts shall require company email + password + OTP (2FA) for every login.              |
| NFR-A07  | Security      | After 5 consecutive failed login attempts, the account shall be locked for 15 minutes.          |
| NFR-A08  | Security      | Admin sessions shall expire after 30 minutes of inactivity; the admin shall be redirected to the login page. |
| NFR-A09  | Security      | All sensitive admin actions (fare/reference-rate change, dispute/refund decision, suspension, approval) shall be logged in an immutable audit trail. Audit log entries cannot be edited or deleted. |
| NFR-A10  | Compatibility | The portal shall be fully functional on the latest 2 versions of Chrome and Microsoft Edge on desktop; minimum screen resolution 1280×720. |
| NFR-A11  | Usability     | Bulk actions (e.g., approving multiple driver applications) shall be supported via checkboxes and a batch action control. |
| NFR-A12  | Usability     | All data tables shall support column-level sorting and pagination (25/50/100 rows per page). |
| NFR-A13  | Usability     | All filter states in list views shall persist within the browser session (i.e., navigating away and back restores filters). |
| NFR-A14  | Reliability   | The admin portal shall maintain 99.5% uptime. Scheduled maintenance windows shall be communicated to admins in advance via email. |
| NFR-A15  | Scalability   | The portal shall support up to 50 concurrent admin users without degradation in dashboard load time. |
| NFR-A16  | Reliability   | Package purchase and activation events shall be processed idempotently to prevent duplicate activations and duplicate revenue entries during retries/network failures. |

---

## 5. Approval

| Role              | Name | Signature | Date |
|-------------------|------|-----------|------|
| Product Owner     |      |           |      |
| Business Analyst  |      |           |      |
| Tech Lead         |      |           |      |

---

*End of Document — BRD-REKLA-ADMIN-004 v1.2*
