# 13 — API List by Module (End-to-End)

**Version:** 1.0  
**Project:** Rekla Ride App — ZhooCars Backend  
**Status Key:** ✅ Exists | 🆕 New (to be built) | 🔧 Extend (modify existing)

---

## Legend

| Column | Meaning |
|--------|---------|
| Method | HTTP verb |
| Path | Full URL path |
| Auth | `None` = public, `JWT` = any authenticated user, `Rider` = Rider role only, `Driver` = Driver role only, `Admin` = Admin roles, `Internal` = service-to-service |
| Status | ✅ Exists \| 🆕 New \| 🔧 Extend |
| Description | What the endpoint does |

---

## Module 1 — Authentication

**Base:** `api/account`  
**Service:** ZhooCars.API

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/account/send-otp | None | ✅ | Send OTP to phone number |
| POST | /api/account/resend-otp | None | ✅ | Resend OTP to same phone |
| POST | /api/account/verify-otp | None | 🔧 | Verify OTP; if new user create account; return JWT + refresh token per role (Rider/Driver) |
| POST | /api/account/me | JWT | ✅ | Get current user info from JWT claims |
| POST | /api/account/RefreshToken | None | 🔧 | Exchange refresh token for new JWT; enforce per-user-per-role single token rule |
| POST | /api/account/logout | JWT | 🆕 | Revoke current refresh token (caller's role) |
| POST | /api/account/logout-all | JWT | 🆕 | Revoke all refresh tokens across all roles for this user |
| POST | /api/account/switch-role | JWT | 🆕 | Switch active role (Rider ↔ Driver) for multi-role accounts |

---

## Module 2 — Rider App

**Service:** ZhooCars.API

### 2A — Profile

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/user/GetUserDetails | Rider | ✅ | Get rider profile (name, address, preferences) |
| POST | /api/user/upsertUserDetails | Rider | ✅ | Create or update rider profile fields |
| GET | /api/user/GetUserDashBoardDetails | Rider | ✅ | Rider home screen data (recent rides, balance, active ride) |

### 2B — Booking Flow

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/taxi/calculate-fare | Rider | ✅ | Calculate estimated fare for given route and vehicle type |
| POST | /api/taxi/get-fare-options | Rider | ✅ | Get multiple fare options across all available vehicle types |
| POST | /api/taxi/book-ride | Rider | 🔧 | Create a ride request; supports Immediate, Scheduled, Rental, Outstation |
| POST | /api/taxi/update-booking | Rider | ✅ | Modify a pending booking (time, address) |
| POST | /api/taxi/cancel-ride/{rideRequestId} | Rider | ✅ | Cancel an existing ride request |
| GET | /api/taxi/rideinfo | Rider | ✅ | Get current active ride status and driver location |
| GET | /api/taxi/rideDetails/{rideRequestId} | Rider | ✅ | Get full details for a specific ride request |

### 2C — Trip Summary

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/ride-trips/rideSummary/{rideRequestId} | Rider | ✅ | Get post-trip summary with fare, distance, payment info |
| GET | /api/ride-trips/payment/{rideTripId} | Rider | ✅ | Get payment details for a completed trip |

### 2D — Ride History

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/ride-history/passenger-history | Rider | ✅ | Get rider's past ride history (paginated, with filters) |
| POST | /api/ride-history/upcoming-rides | Rider | ✅ | Get scheduled upcoming rides |

### 2E — Saved Addresses

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/rider/addresses | Rider | 🆕 | List all saved addresses (Home, Work, Others) |
| POST | /api/rider/addresses | Rider | 🆕 | Add a new saved address |
| PUT | /api/rider/addresses/{id} | Rider | 🆕 | Update an existing saved address |
| DELETE | /api/rider/addresses/{id} | Rider | 🆕 | Delete a saved address |

### 2F — Promo Codes

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/rider/promo/validate | Rider | 🆕 | Validate and preview discount for a promo code |
| GET | /api/rider/promo/applicable | Rider | 🆕 | Get list of applicable coupons for current booking context |

### 2G — Referral Program

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/rider/referrals | Rider | 🆕 | Get rider's referral code, referral count, and earned credits |
| POST | /api/rider/referrals/apply | None | 🆕 | Apply referral code during registration |

### 2H — Loyalty Points

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/rider/loyalty/balance | Rider | 🆕 | Get current loyalty points balance |
| GET | /api/rider/loyalty/history | Rider | 🆕 | Get loyalty points ledger (earn/redeem history) |
| POST | /api/rider/loyalty/redeem | Rider | 🆕 | Redeem loyalty points against a booking |

### 2I — Feedback & Ratings

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/feedback/add | Rider | ✅ | Submit post-trip feedback and star rating for driver |
| GET | /api/rider/ratings/{rideRequestId} | Rider | 🆕 | Get rating submitted for a specific trip |

---

## Module 3 — Driver App

**Service:** ZhooCars.API

### 3A — Registration & Onboarding

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/driverdetail | Driver | ✅ | Get driver's own profile and details |
| GET | /api/driverdetail/DriverProfile | Driver | ✅ | Get full driver profile page data |
| POST | /api/driverdetail/update | Driver | ✅ | Create or update driver profile fields |

### 3B — Availability & Status

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/driverstatus/update | Driver | ✅ | Toggle driver online / offline / on-trip status |
| GET | /api/driverstatus | Driver | ✅ | Get current availability status |

### 3C — Shift Management

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/driver-shift/check-in | Driver | 🆕 | Start a new shift (records start time + GPS location) |
| POST | /api/driver-shift/check-out | Driver | 🆕 | End current shift (records end time + GPS location) |
| GET | /api/driver-shift/active | Driver | 🆕 | Get current active shift details |
| GET | /api/driver-shift/shift-logs | Driver | ✅ | Get historical shift log list |

### 3D — Ride Handling

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/taxi/accept-ride | Driver | ✅ | Accept a ride request bid |
| POST | /api/taxi/skip-bid | Driver | ✅ | Skip / reject a ride bid without penalty counting |
| POST | /api/ride-trips/update-status | Driver | ✅ | Update trip status (on-way, reached, trip-started, dropped) |
| POST | /api/ride-trips/complete | Driver | ✅ | Complete a trip (triggers payment calculations) |
| POST | /api/ride-trips/update-distance | Driver | ✅ | Update accumulated distance during live trip |

### 3E — Earnings

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/driver/earnings/summary | Driver | 🆕 | Get total earnings, commission, net payout summary |
| GET | /api/driver/earnings/breakdown | Driver | 🆕 | Per-trip earnings breakdown with date filter |
| GET | /api/driver/earnings/statement/{month} | Driver | 🆕 | Monthly earnings statement (downloadable) |

### 3F — Driver Packages (Purchase Side)

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/driver-packages/available | Driver | 🆕 | List packages available for purchase |
| GET | /api/driver-packages/my-packages | Driver | 🆕 | List driver's active and expired packages |
| GET | /api/driver-packages/{packageId} | Driver | 🆕 | Get package details before purchase |
| POST | /api/driver-packages/purchase | Driver | 🆕 | Purchase a package (initiates payment) |
| POST | /api/driver-packages/refund-claim | Driver | 🆕 | Submit a refund claim with reason and evidence |
| GET | /api/driver-packages/refund-claim/{claimId} | Driver | 🆕 | Get refund claim status |

### 3G — Ratings & Reviews

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/driver/ratings | Driver | 🆕 | Get driver's average rating and recent reviews |

### 3H — Policy Violations

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/driver/violations | Driver | 🆕 | Get driver's policy violation alerts and status |

---

## Module 4 — Vehicle Management

**Service:** ZhooCars.API

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/vehicledetails/register | Driver | ✅ | Register a new vehicle |
| POST | /api/vehicledetails/update | Driver | ✅ | Update existing vehicle details |
| GET | /api/vehicledetails/{id} | Driver \| Admin | ✅ | Get vehicle by ID |
| GET | /api/vehicledetails/filter | Admin | ✅ | List vehicles with filter (status, type, vendor) |
| PATCH | /api/vehicledetails/update-insurance/{id} | Driver | ✅ | Update vehicle insurance information |
| DELETE | /api/vehicledetails/{id} | Admin | ✅ | Delete a vehicle record |
| POST | /api/driverandvehiclelink/request-link | Driver | ✅ | Request linking this driver to a vehicle |
| POST | /api/driverandvehiclelink/verify-link | Admin | ✅ | Verify vehicle link via OTP or admin approval |
| POST | /api/driverandvehiclelink/create-link | Admin | ✅ | Directly create a driver-vehicle link |
| GET | /api/driverandvehiclelink/get-link | Driver | ✅ | Get driver's current vehicle link |
| POST | /api/driverandvehiclelink/delete-link | Admin \| Driver | ✅ | Remove a driver-vehicle link |
| GET | /api/driverandvehiclelink/vehicles-by-vendor | Admin \| Vendor | ✅ | Get all vehicles assigned under a vendor |

---

## Module 5 — Vehicle Location (Real-time)

**Service:** ZhooCars.API + ZhooSoft.Tracker

| Method | Path | Auth | Status | Description | Service |
|--------|------|------|--------|-------------|---------|
| POST | /api/vehiclelocation/update-location | Driver | ✅ | Update vehicle GPS coordinates | API |
| POST | /api/vehiclelocation/update-status | Driver | ✅ | Update vehicle availability status | API |
| GET | /api/vehiclelocation/get-location/{vehicleId} | Admin \| Internal | ✅ | Get last known GPS position for a vehicle | API |
| GET | /api/clientside/nearby-drivers | Rider | ✅ | Get nearby available drivers within radius | Tracker |
| GET | /api/clientside/getdrivers | Admin \| Internal | ✅ | Get all currently online drivers | Tracker |
| GET | /api/internal/driver-location/{driverId} | Internal | ✅ | Get single driver location from Redis | Tracker |
| POST | /api/internal/driver-locations | Internal | ✅ | Batch get multiple driver locations | Tracker |
| WS | /hubs/location | JWT | ✅ | SignalR WebSocket hub — ride events, location stream | Tracker |

---

## Module 6 — Documents

**Service:** ZhooCars.API

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/document/UpsertDocument | Driver | ✅ | Create or update a document record by type |
| POST | /api/document/UploadDocument | Driver | ✅ | Upload a single document with metadata |
| POST | /api/document/UploadDocuments | Driver | ✅ | Bulk upload multiple documents |
| POST | /api/document/UploadFile | Any | ✅ | Upload a raw file to blob storage |
| POST | /api/document/UploadDocumentByFile | Driver | ✅ | Upload document with embedded file in one call |
| PUT | /api/document/Approve/{documentId} | Admin | ✅ | Approve or reject a submitted document |
| GET | /api/document/GetDocuments | Driver \| Admin | ✅ | Get documents for the requesting user |
| GET | /api/document/GetDocuments?userId={id} | Admin | 🔧 | Get documents for any user (admin override) |

---

## Module 7 — Admin Portal

**Service:** ZhooCars.API

### 7A — Approval Management

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/admin/approve | Admin | ✅ | Approve an entity (driver, document, vehicle) |
| POST | /api/admin/reject | Admin | ✅ | Reject an entity with reason |
| GET | /api/admin/approval-history/{phoneNumber} | Admin | ✅ | Get approval history by phone number |
| GET | /api/admin/approvals/pending | Admin | 🆕 | List all pending approval items across all entity types |

### 7B — Dashboard

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/dashboard/overview | Admin | 🆕 | Summary stats: total rides, drivers online, revenue today |
| GET | /api/admin/dashboard/ride-stats | Admin | 🆕 | Ride completion, cancellation, pending rate |
| GET | /api/admin/dashboard/driver-stats | Admin | 🆕 | Driver online count, inactive count, newly registered |
| GET | /api/admin/dashboard/earnings-summary | Admin | 🆕 | Revenue breakdown by period |

### 7C — Driver Management

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/drivers | Admin | 🆕 | List all drivers with filter (status, approval, date) |
| GET | /api/admin/drivers/{userId} | Admin | ✅ | Get driver full profile (maps to GetDriverProfileById) |
| PUT | /api/admin/drivers/{userId}/status | Admin | 🆕 | Activate or suspend a driver account |
| GET | /api/admin/drivers/{userId}/documents | Admin | 🆕 | Get all documents submitted by a driver |
| GET | /api/admin/drivers/{userId}/earnings | Admin | 🆕 | View driver earnings from admin perspective |
| GET | /api/admin/drivers/{userId}/rides | Admin | 🆕 | View rides completed by a specific driver |

### 7D — Rider Management

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/riders | Admin | 🆕 | List all riders with filter |
| GET | /api/admin/riders/{userId} | Admin | 🆕 | Get full rider profile |
| PUT | /api/admin/riders/{userId}/status | Admin | 🆕 | Activate or suspend a rider account |
| GET | /api/admin/riders/{userId}/rides | Admin | 🆕 | View rides history for a specific rider |

### 7E — Ride Management

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/rides | Admin | 🆕 | List all rides with filters (status, date, driver, rider) |
| GET | /api/admin/rides/{rideRequestId} | Admin | ✅ | Get ride details (maps to GetRideDetailsInfo) |
| POST | /api/admin/rides/force-cancel | Admin | 🆕 | Force cancel any active ride with reason |
| POST | /api/admin/rides/book-on-behalf | Admin | 🆕 | Create a ride booking on behalf of a rider |

### 7F — Fare & Pricing

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/peak-hours/GetAllPeakHours | Admin | ✅ | List all peak hour pricing rules |
| GET | /api/peak-hours/{id} | Admin | ✅ | Get single peak hour rule by ID |
| POST | /api/peak-hours/AddPeakHour | Admin | ✅ | Add a new peak hour rule |
| POST | /api/peak-hours/{id} | Admin | ✅ | Update peak hour rule |
| DELETE | /api/peak-hours/{id} | Admin | ✅ | Delete peak hour rule |
| GET | /api/admin/pricing | Admin | 🆕 | List ride_pricing rules |
| POST | /api/admin/pricing | Admin | 🆕 | Create new pricing rule (vehicle type + ride type) |
| PUT | /api/admin/pricing/{id} | Admin | 🆕 | Update an existing pricing rule |
| DELETE | /api/admin/pricing/{id} | Admin | 🆕 | Delete a pricing rule |

### 7G — Promo Code Management

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/coupons | Admin | 🆕 | List all coupons with filter |
| POST | /api/admin/coupons | Admin | 🆕 | Create a new coupon/promo code |
| PUT | /api/admin/coupons/{id} | Admin | 🆕 | Update coupon details |
| DELETE | /api/admin/coupons/{id} | Admin | 🆕 | Delete / deactivate a coupon |
| GET | /api/admin/coupons/{id}/usage | Admin | 🆕 | View coupon usage and redemption stats |

### 7H — Driver Package Administration

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/driver-packages | Admin | 🆕 | List all packages defined in the system |
| POST | /api/admin/driver-packages | Admin | 🆕 | Create a new driver package |
| PUT | /api/admin/driver-packages/{id} | Admin | 🆕 | Update package pricing/features |
| DELETE | /api/admin/driver-packages/{id} | Admin | 🆕 | Delete a package |
| GET | /api/admin/driver-packages/purchases | Admin | 🆕 | View all driver package purchases |
| POST | /api/admin/driver-packages/refunds/{claimId}/approve | Admin | 🆕 | Approve a driver refund claim |
| POST | /api/admin/driver-packages/refunds/{claimId}/reject | Admin | 🆕 | Reject a driver refund claim with reason |

### 7I — User & Access Management

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/users | SuperAdmin | 🆕 | List all admin/staff users |
| POST | /api/admin/users | SuperAdmin | 🆕 | Create a new admin user |
| PUT | /api/admin/users/{id}/role | SuperAdmin | 🆕 | Assign or change admin user role |
| PUT | /api/admin/users/{id}/deactivate | SuperAdmin | 🆕 | Deactivate an admin user account |

### 7J — Quote Desk (Outstation / Custom)

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/quotes | Admin | 🆕 | List all quote requests |
| POST | /api/admin/quotes | Admin | 🆕 | Create a manual outstation or package quote |
| GET | /api/admin/quotes/{id} | Admin | 🆕 | Get quote details and status |
| POST | /api/admin/quotes/{id}/convert | Admin | 🆕 | Convert a quote to a confirmed booking |
| POST | /api/admin/quotes/{id}/send | Admin | 🆕 | Send quote link to rider via SMS / email |

### 7K — Reports

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/reports/templates | Admin | 🆕 | List all defined report templates |
| POST | /api/admin/reports/generate | Admin | 🆕 | Trigger report generation from a template |
| GET | /api/admin/reports/runs | Admin | 🆕 | List report run history |
| GET | /api/admin/reports/runs/{runId}/download | Admin | 🆕 | Download completed report file |

### 7L — Audit Logs

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/admin/audit-logs | Admin | 🆕 | Query immutable audit log with filter (actor, action, date) |

### 7M — Governance / High-Impact Change Requests

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/admin/governance/change-requests | Admin | 🆕 | Submit a high-impact change request for approval |
| GET | /api/admin/governance/change-requests | SuperAdmin | 🆕 | List pending change requests |
| POST | /api/admin/governance/change-requests/{id}/approve | SuperAdmin | 🆕 | Approve a change request |
| POST | /api/admin/governance/change-requests/{id}/reject | SuperAdmin | 🆕 | Reject a change request with reason |

---

## Module 8 — Support & Helpdesk

**Service:** ZhooCars.API

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/tickets/addTicket | JWT | ✅ | Create a support ticket |
| POST | /api/tickets/filteredTickets | Admin | ✅ | List tickets with filters |
| GET | /api/tickets/getTicketDetails/{id} | JWT | ✅ | Get ticket details and comment thread |
| POST | /api/tickets/updateTicket | Admin | ✅ | Update ticket status, assignment, notes |
| POST | /api/support/sos | Rider \| Driver | 🆕 | Submit an SOS emergency alert during trip |
| GET | /api/admin/support/sos | Admin | 🆕 | List SOS alerts with filter |
| GET | /api/admin/support/sos/{id} | Admin | 🆕 | Get SOS alert details |
| POST | /api/admin/support/policy-violations | Admin | 🆕 | Record a policy violation against a driver |
| GET | /api/admin/support/policy-violations | Admin | 🆕 | List policy violation alerts |
| GET | /api/support/faq | None | 🆕 | Get public FAQ list by category |
| POST | /api/admin/support/faq | Admin | 🆕 | Create a new FAQ entry |
| PUT | /api/admin/support/faq/{id} | Admin | 🆕 | Update an existing FAQ entry |
| DELETE | /api/admin/support/faq/{id} | Admin | 🆕 | Delete a FAQ entry |

---

## Module 9 — Notifications

**Services:** ZhooCars.API (inbox/settings) | ZhooSoft.Tracker (push delivery)

| Method | Path | Auth | Status | Description | Service |
|--------|------|------|--------|-------------|---------|
| POST | /api/notify/send-to-user | Internal | ✅ | Send push notification to a specific user | Tracker |
| GET | /api/notifications/inbox | JWT | 🆕 | Get user's in-app notification inbox (paginated) | API |
| PUT | /api/notifications/inbox/{id}/read | JWT | 🆕 | Mark a single notification as read | API |
| PUT | /api/notifications/inbox/read-all | JWT | 🆕 | Mark all inbox notifications as read | API |
| GET | /api/notifications/settings | JWT | 🆕 | Get notification preference settings | API |
| PUT | /api/notifications/settings | JWT | 🆕 | Update notification preferences by type | API |

---

## Module 10 — Configuration & Deployment

**Service:** ZhooCars.API  
**Note:** Deployment-scoped configuration, feature flags, and config registry
**Independence Rule:** Configuration is runtime value-only and independent. Configuration records are not FK-linked to core transactional/domain tables.

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/config/enums | Admin \| Internal | 🆕 | List all enum values organized by category |
| GET | /api/config/keys | Admin | 🆕 | List all registered configuration key definitions |
| GET | /api/config/values | Admin | 🆕 | Get all deployment configuration values |
| PUT | /api/config/values/{key} | SuperAdmin | 🆕 | Update a deployment configuration value |
| GET | /api/config/feature-flags | Admin \| Internal | 🆕 | Get all feature flag states for this deployment |
| PUT | /api/config/feature-flags/{flag} | SuperAdmin | 🆕 | Toggle a feature flag on/off |
| GET | /api/config/business-rules | Admin | 🆕 | Get business rule configuration values |
| PUT | /api/config/business-rules/{key} | SuperAdmin | 🆕 | Update a business rule value |
| GET | /api/config/change-history | Admin | 🆕 | List configuration change audit history |

---

## Module 11 — Metadata / Lookup

**Service:** ZhooCars.API  
**Note:** Public-read endpoints return cached data; write endpoints are admin-only

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/metadata/vehicle-types | None | ✅ | Get all active vehicle types |
| GET | /api/metadata/vehicle-models | None | ✅ | Get all vehicle models |
| GET | /api/metadata/vehicle-models/{typeId} | None | ✅ | Get vehicle models filtered by vehicle type |
| GET | /api/metadata/getLanguages | None | ✅ | Get supported display languages |
| POST | /api/metadata/vehicle-types | Admin | 🆕 | Create a new vehicle type |
| PUT | /api/metadata/vehicle-types/{id} | Admin | 🆕 | Update a vehicle type |
| POST | /api/metadata/vehicle-models | Admin | 🆕 | Create a new vehicle model |
| PUT | /api/metadata/vehicle-models/{id} | Admin | 🆕 | Update a vehicle model |
| GET | /api/metadata/ride-types | None | 🆕 | List all ride types (Immediate, Scheduled, Rental, Outstation) |
| GET | /api/metadata/app-config | None | 🆕 | Get public app configuration (min version, feature flags exposed to app) |

---

## Module 12 — Vendor Management

**Service:** ZhooCars.API

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/vendor | Vendor \| Admin | ✅ | Get vendor profile by JWT identity |
| GET | /api/vendor/filter | Admin | ✅ | List vendors with filter |
| GET | /api/vendor/vehicles | Vendor \| Admin | ✅ | Get all vehicles under this vendor |
| POST | /api/vendor/registerOrupdate | Admin \| Vendor | ✅ | Register or update vendor profile |

---

## Module 13 — Service Provider (Extended Module)

**Service:** ZhooCars.API

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/serviceproviders/register | Admin | ✅ | Register a new service provider |
| POST | /api/serviceproviders/update | Admin | ✅ | Update service provider details |
| GET | /api/serviceproviders/{id} | Admin | ✅ | Get service provider by ID |
| GET | /api/serviceproviders/filter | Admin | ✅ | List service providers with filter |
| POST | /api/service-requests | JWT | ✅ | Create a service request |
| POST | /api/service-requests/{ticketId}/notify-nearby | Admin | ✅ | Push notification to nearby service providers |
| PUT | /api/service-requests/{ticketId}/assign/{providerId} | Admin | ✅ | Assign a service request to a provider |
| PUT | /api/service-requests/{ticketId}/update-status | Admin | ✅ | Update service request status |
| POST | /api/service-requests/admin/search | Admin | ✅ | Admin search of all service requests |
| GET | /api/service-requests/GetServiceRequestDetails | Admin | ✅ | Get detailed service request |

---

## Module 14 — Spare Parts Provider

**Service:** ZhooCars.API

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/sparepartsproviders/register | Admin | ✅ | Register a spare parts provider |
| POST | /api/sparepartsproviders/update | Admin | ✅ | Update spare parts provider |
| GET | /api/sparepartsproviders/{id} | Admin | ✅ | Get by ID |
| GET | /api/sparepartsproviders/filter | Admin | ✅ | List with filter |

---

## Module 15 — BRD Sync Delta (Reliability, Serviceability, Settlement, SLA)

**Service:** ZhooCars.API (with Tracker/Internal support where noted)  
**Purpose:** Close BRD feature gaps and ensure Success + Negative + Fallback coverage per feature.

### 15A — Dispatch Resilience & No-Driver Fallback

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/dispatch/match/start | Internal | 🆕 | Start dispatch matching cycle for booking/request |
| POST | /api/dispatch/match/rematch | Internal | 🆕 | Trigger rematch attempt with incremented attempt count |
| POST | /api/dispatch/match/timeout | Internal | 🆕 | Mark dispatch timeout and apply fallback action |
| GET | /api/dispatch/health | Admin | 🆕 | Dispatch KPIs: match rate, no-driver rate, rematch success |

### 15B — Scheduled Ride Reliability (Package/Outstation)

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/scheduled-rides/pre-assign | Admin \| Internal | 🆕 | Pre-assign primary driver for scheduled booking |
| POST | /api/scheduled-rides/reconfirm/rider | Rider | 🆕 | Rider reconfirmation before scheduled pickup |
| POST | /api/scheduled-rides/reconfirm/driver | Driver | 🆕 | Driver reconfirmation before scheduled pickup |
| POST | /api/scheduled-rides/backup-assign | Admin \| Internal | 🆕 | Assign backup driver when primary fails |
| POST | /api/scheduled-rides/no-show | Admin \| Driver \| Rider | 🆕 | Record no-show actor and trigger policy flow |
| POST | /api/scheduled-rides/sla-breach/escalate | Internal \| Admin | 🆕 | Trigger scheduled-ride SLA breach playbook |
| GET | /api/scheduled-rides/at-risk | Admin | 🆕 | List upcoming at-risk scheduled rides |

### 15C — Serviceability & Geo-Governance

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/serviceability/check | Rider \| Admin | 🆕 | Validate pickup/drop serviceability and reason codes |
| GET | /api/admin/service-zones | Admin | 🆕 | List configured service zones |
| POST | /api/admin/service-zones | Admin | 🆕 | Create service zone |
| PUT | /api/admin/service-zones/{zoneId} | Admin | 🆕 | Update service zone |
| GET | /api/admin/geofences | Admin | 🆕 | List geofences by zone/type |
| POST | /api/admin/geofences | Admin | 🆕 | Create geofence polygon |
| PUT | /api/admin/geofences/{geofenceId} | Admin | 🆕 | Update geofence polygon or status |
| POST | /api/location-tamper/report | Internal \| Driver \| Rider | 🆕 | Report spoof/tamper signal |
| GET | /api/admin/location-tamper/events | Admin | 🆕 | Review tamper events and actions |

### 15D — Driver-Vehicle Linking, Switching, Vendor Mapping

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/driverandvehiclelink/switch-request | Driver | 🆕 | Raise driver-vehicle switch request |
| POST | /api/admin/driverandvehiclelink/switch-approve/{requestId} | Admin | 🆕 | Approve switch request with effective time |
| POST | /api/admin/driverandvehiclelink/switch-reject/{requestId} | Admin | 🆕 | Reject switch request with reason |
| GET | /api/driverandvehiclelink/history | Driver \| Admin | 🆕 | Get link mapping history and statuses |
| POST | /api/admin/vendor/driver-vehicle-assign | Admin | 🆕 | Assign vendor-owned vehicle to eligible driver |

### 15E — Driver Settlement & Payout (Driver-Only Monetization)

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| GET | /api/driver/wallet/ledger | Driver | 🆕 | Driver wallet ledger with filters |
| GET | /api/driver/settlements | Driver | 🆕 | List settlement cycles and status |
| GET | /api/driver/settlements/{cycleId} | Driver | 🆕 | Detailed settlement breakdown for cycle |
| GET | /api/driver/payouts | Driver | 🆕 | List payouts and failure reasons |
| POST | /api/admin/settlements/generate-cycle | Admin | 🆕 | Generate settlement cycle |
| POST | /api/admin/settlements/{cycleId}/close | Admin | 🆕 | Close and lock settlement cycle |
| POST | /api/admin/payouts/initiate | Admin | 🆕 | Initiate payout batch |
| POST | /api/admin/payouts/{payoutId}/retry | Admin | 🆕 | Retry failed payout |
| POST | /api/admin/payouts/{payoutId}/resolve | Admin | 🆕 | Resolve payout failure manually |

### 15F — Support Ticket SLA, Reopen, Assignment History

| Method | Path | Auth | Status | Description |
|--------|------|------|--------|-------------|
| POST | /api/tickets/{ticketId}/assign | Admin \| Support | 🆕 | Assign ticket to team/user |
| POST | /api/tickets/{ticketId}/reopen | Rider \| Driver \| Admin | 🆕 | Reopen ticket within policy window |
| POST | /api/tickets/{ticketId}/mark-duplicate | Admin \| Support | 🆕 | Mark duplicate and link parent ticket |
| GET | /api/tickets/{ticketId}/status-history | JWT | 🆕 | Ticket status transition history |
| GET | /api/tickets/{ticketId}/assignment-history | Admin \| Support | 🆕 | Assignment timeline history |
| GET | /api/tickets/{ticketId}/sla-events | Admin \| Support | 🆕 | SLA warning/breach/pause/resume events |

---

## Feature Scenario Coverage Matrix (Mandatory)

| Feature | Success Scenario API Coverage | Negative Scenario API Coverage | Fallback Scenario API Coverage |
|--------|-------------------------------|--------------------------------|--------------------------------|
| Local/Instant booking | `/api/taxi/book-ride`, `/api/dispatch/match/start` | `/api/dispatch/match/timeout` | `/api/dispatch/match/rematch`, rider notification flow |
| Scheduled Package/Outstation | `/api/scheduled-rides/pre-assign`, reconfirm APIs | `/api/scheduled-rides/no-show` | `/api/scheduled-rides/backup-assign`, `/api/scheduled-rides/sla-breach/escalate` |
| Serviceability/geofence | `/api/serviceability/check` (serviceable) | `/api/serviceability/check` (out_of_service/restricted) | clear user alternative + admin zone reconfiguration APIs |
| Driver-vehicle mapping | link/create/verify/switch approve APIs | switch reject / invalid active-trip blocking | previous mapping remains active until approved effective switch |
| Driver settlement/payout | cycle generate/close + payout initiate | payout failure list and failure reason | payout retry/resolve APIs |
| Support ticket lifecycle | add/update/assign/resolve APIs | invalid state changes, duplicate, SLA breach | reopen, reassignment, escalation, SLA event handling |
| Referral/reward/rating | referral qualify + coupon/wallet credit + ratings submit | fraud/rejection/dispute APIs | reversal/moderation/dispute-resolution APIs |

Coverage rule:
1. Every new feature must expose at least one API path each for success, negative, and fallback outcomes.
2. No feature is release-ready if any of the three scenario paths is missing.

---

## Summary — API Count by Module

| Module | Total | ✅ Exists | 🔧 Extend | 🆕 New |
|--------|-------|----------|----------|-------|
| 1. Authentication | 8 | 3 | 2 | 3 |
| 2. Rider App | 26 | 10 | 1 | 15 |
| 3. Driver App | 26 | 10 | 0 | 16 |
| 4. Vehicle Management | 12 | 12 | 0 | 0 |
| 5. Vehicle Location / Real-time | 8 | 8 | 0 | 0 |
| 6. Documents | 8 | 7 | 1 | 0 |
| 7. Admin Portal | 45 | 6 | 0 | 39 |
| 8. Support & Helpdesk | 14 | 4 | 0 | 10 |
| 9. Notifications | 6 | 1 | 0 | 5 |
| 10. Configuration & Deployment | 9 | 0 | 0 | 9 |
| 11. Metadata / Lookup | 10 | 4 | 0 | 6 |
| 12. Vendor Management | 4 | 4 | 0 | 0 |
| 13. Service Provider | 10 | 10 | 0 | 0 |
| 14. Spare Parts Provider | 4 | 4 | 0 | 0 |
| **TOTAL** | **190** | **83** | **4** | **103** |

> Note: Totals above are baseline counts from Modules 1-14. Module 15 is a BRD-sync delta layer and should be included in implementation scope even if counts are not yet rolled into baseline totals.

---

## Notes

1. **Multi-role flows:** Wherever Auth = `Rider` or `Driver`, the JWT must carry the matching active role claim. The `switch-role` API in Module 1 updates the active role without requiring re-login.
2. **Extend endpoints (🔧):** These exist in code but require modification — primarily adding role enforcement, adding missing fields, or wiring multi-role token logic.
3. **Internal endpoints:** Used for cross-service calls (ZhooCars.API ↔ ZhooSoft.Tracker). Should be protected via shared API key or network policy, not public JWT.
4. **Admin sub-paths:** The admin portal routes use `/api/admin/...` convention to cleanly separate admin-only operations even when data overlaps with regular controllers.
5. **SuperAdmin:** A specific role with higher privilege than Admin — used for irreversible operations (force cancel, delete, user deactivation, config changes, high-impact governance).
