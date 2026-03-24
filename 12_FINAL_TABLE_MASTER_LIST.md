# Final Table Master List - ZhooCars / Rekla Platform
## Document ID: SCHEMA-MASTER-001
## Version: 1.0 | Date: March 23, 2026
## Status: Design - Pre-Implementation

---

## How to Read this Document

| Symbol | Meaning |
|--------|---------|
| ✅ EXISTS | Column/table already in database |
| ➕ ADD | New column to add to existing table |
| 🆕 NEW | Entirely new table to create |
| PK | Primary key |
| FK | Foreign key |
| IDX | Needs index |
| UNQ | Unique constraint |

Tables are grouped by **module**. Within each table, existing columns are listed first, then columns to add.

---

## MODULE 1 - IDENTITY & AUTHENTICATION

### 1.1 `users` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| user_id | SERIAL | PK | ✅ EXISTS | |
| phone_number | VARCHAR(20) | NOT NULL, IDX | ✅ EXISTS | Shared across roles (BR-19) |
| email | VARCHAR(255) | NULLABLE | ✅ EXISTS | |
| password_hash | TEXT | NULLABLE | ✅ EXISTS | Admin only |
| status | VARCHAR(50) | DEFAULT 'active' | ✅ EXISTS | |
| is_active | BOOLEAN | DEFAULT true | ✅ EXISTS | |
| last_login | TIMESTAMP | NULLABLE | ✅ EXISTS | |
| created_at | TIMESTAMP | DEFAULT now() | ✅ EXISTS | |
| updated_at | TIMESTAMP | | ✅ EXISTS | |
| referral_code | VARCHAR(20) | NULLABLE, UNQ | ➕ ADD | Plugin: referral module |

> **Note:** `role_id` column present on entity - this is a legacy single-role field. Multi-role is handled via `user_roles` table (BR-19). Do not remove; keep for backward compat.

> **Design Caution:** Keep `users` minimal because it is a high-frequency shared identity table. Role-specific restriction state must stay in role-owned tables such as `driver_details`, not in `users`.

---

### 1.2 `user_login_history` ➕ ADD

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | ➕ ADD | |
| user_id | INT | FK -> users, IDX | ➕ ADD | |
| ip_address | VARCHAR(50) | NULLABLE | ➕ ADD | Login IP for security audit |
| device_id | VARCHAR(255) | NULLABLE | ➕ ADD | Login device for security alert |
| logged_in_at | TIMESTAMP | DEFAULT now() | ➕ ADD | |

> **Design Note:** Moved from `users` to avoid write contention on a high-frequency identity table. Enables full login audit history, not just last-login snapshot.

---

### 1.3 `user_details` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| user_id | INT | PK, FK -> users | ✅ EXISTS | |
| first_name | VARCHAR(100) | NULLABLE | ✅ EXISTS | |
| last_name | VARCHAR(100) | NULLABLE | ✅ EXISTS | |
| profile_picture_url | TEXT | NULLABLE | ✅ EXISTS | |
| gender | VARCHAR(20) | NULLABLE | ✅ EXISTS | |
| date_of_birth | DATE | NULLABLE | ✅ EXISTS | |
| street1, street2, area, city, state | VARCHAR | NULLABLE | ✅ EXISTS | |
| postal_code, country | VARCHAR | NULLABLE | ✅ EXISTS | |
| emergency_contact_name | VARCHAR(100) | NULLABLE | ✅ EXISTS | |
| emergency_contact_phone | VARCHAR(20) | NULLABLE | ✅ EXISTS | |
| language | VARCHAR(20) | NULLABLE | ✅ EXISTS | UI preference language (shared across roles) |
| timezone | VARCHAR(50) | NULLABLE | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |
| language_preference_id | INT | FK -> enum_value_catalog, NULLABLE | ➕ ADD | Normalized from free-text language |
| legal_acceptance_version | VARCHAR(50) | NULLABLE | ➕ ADD | T&C version user accepted |
| legal_accepted_at | TIMESTAMP | NULLABLE | ➕ ADD | When T&C was accepted |
| default_saved_address_id | INT | FK -> saved_addresses, NULLABLE | ➕ ADD | Rider's default pickup address |

---

### 1.4 `roles` ✅ NO CHANGE

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| role_id | SERIAL | PK | ✅ EXISTS | |
| role_name | VARCHAR(50) | NOT NULL, UNQ | ✅ EXISTS | rider / driver / admin / support |
| description | TEXT | NULLABLE | ✅ EXISTS | |
| is_active | BOOLEAN | DEFAULT true | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |

---

### 1.5 `user_roles` ✅ NO CHANGE

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | ✅ EXISTS | |
| user_id | INT | FK -> users, IDX | ✅ EXISTS | |
| role_id | INT | FK -> roles | ✅ EXISTS | |
| is_active | BOOLEAN | DEFAULT true | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |

> **Note:** This is the multi-role table. One user can have entries for both 'rider' and 'driver' simultaneously (BR-19).

---

### 1.6 `user_refresh_tokens` ✅ EXTEND (CRITICAL)

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | ✅ EXISTS | |
| user_id | INT | FK -> users, IDX | ✅ EXISTS | |
| refresh_token | TEXT | NOT NULL | ✅ EXISTS | |
| is_revoked | BOOLEAN | DEFAULT false | ✅ EXISTS | |
| revoked_at | TIMESTAMP | NULLABLE | ✅ EXISTS | |
| ip_address | VARCHAR(50) | NULLABLE | ✅ EXISTS | |
| device_key | VARCHAR(255) | NULLABLE | ✅ EXISTS | |
| user_agent | TEXT | NULLABLE | ✅ EXISTS | |
| created_at | TIMESTAMP | DEFAULT now() | ✅ EXISTS | |
| expires_at | TIMESTAMP | NOT NULL | ✅ EXISTS | |
| role_id | INT | FK -> roles, IDX | ✅ EXISTS | Already on entity - enforce usage |
| client_app_type_id | INT | | ✅ EXISTS | Already on entity |
| token_hash | VARCHAR(512) | NOT NULL, UNQ, IDX | ➕ ADD | Hashed token for fast lookup |

> **Critical Constraint (BR-20):** UNIQUE index on `(user_id, role_id) WHERE revoked_at IS NULL` - ensures only one active refresh token per user per role. Issuing a new token must revoke the previous one for that user-role pair.

---

### 1.7 `user_notification_settings` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | ✅ EXISTS | |
| user_id | INT | FK -> users, IDX | ✅ EXISTS | |
| notification_type_id | INT | FK -> notification_types | ✅ EXISTS | |
| is_enabled | BOOLEAN | DEFAULT true | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |
| channel | VARCHAR(20) | DEFAULT 'push' | ➕ ADD | push / sms / whatsapp / in_app |
| category | VARCHAR(50) | NULLABLE | ➕ ADD | operational / marketing / safety |
| is_mandatory | BOOLEAN | DEFAULT false | ➕ ADD | Cannot be disabled by user |
| quiet_hours_start | TIME | NULLABLE | ➕ ADD | User-defined quiet start |
| quiet_hours_end | TIME | NULLABLE | ➕ ADD | User-defined quiet end |

---

## MODULE 3 - DRIVER MANAGEMENT & ONBOARDING

### 3.1 `driver_details` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| driver_id | SERIAL | PK | ✅ EXISTS | |
| user_id | INT | FK -> users, UNQ | ✅ EXISTS | |
| languages_spoken | TEXT | NULLABLE | ✅ EXISTS | Languages for rider communication (driver-role only) |
| status | VARCHAR(50) | NULLABLE | ✅ EXISTS | |
| approval_status_id | INT | Enum FK | ✅ EXISTS | |
| vehicle_type_id | INT | FK -> vehicle_types | ✅ EXISTS | |
| driver_type_id | INT | Enum | ✅ EXISTS | |
| is_active | BOOLEAN | DEFAULT true | ✅ EXISTS | |
| review_notes | TEXT | NULLABLE | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |
| primary_zone_id | INT | NULLABLE | ➕ ADD | Primary operating zone/city |
| ride_category_flags | JSONB | NULLABLE | ➕ ADD | {local:true, outstation:false, package:true} |
| package_required_for_dispatch | BOOLEAN | DEFAULT false | ➕ ADD | Dispatch blocked if no active package |
| restricted_reason | TEXT | NULLABLE | ➕ ADD | Policy restriction reason |
| restricted_until | TIMESTAMP | NULLABLE | ➕ ADD | Timed restriction |

---

### 3.2 `driver_status` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| driver_status_id | SERIAL | PK | ✅ EXISTS | |
| user_id | INT | FK -> users, UNQ, IDX | ✅ EXISTS | One status row per driver |
| vehicle_id | INT | FK -> vehicle_details, NULLABLE | ✅ EXISTS | |
| status | VARCHAR(20) | NOT NULL | ✅ EXISTS | online / offline / on_trip |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |
| lock_reason | VARCHAR(50) | NULLABLE | ➕ ADD | active_trip / policy_violation / no_package |
| current_trip_id | INT | FK -> ride_trips, NULLABLE | ➕ ADD | |
| last_status_changed_at | TIMESTAMP | NULLABLE | ➕ ADD | |
| status_source | VARCHAR(20) | NULLABLE | ➕ ADD | driver_action / system / admin |

---

### 3.3 `driver_shift_log` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| shift_id | SERIAL | PK | ✅ EXISTS | |
| user_id | INT | FK -> users, IDX | ✅ EXISTS | |
| vehicle_id | INT | FK -> vehicle_details | ✅ EXISTS | |
| check_in_time | TIMESTAMP | NOT NULL | ✅ EXISTS | |
| check_out_time | TIMESTAMP | NULLABLE | ✅ EXISTS | |
| check_in_latitude/longitude | DECIMAL | NULLABLE | ✅ EXISTS | |
| check_out_latitude/longitude | DECIMAL | NULLABLE | ✅ EXISTS | |
| shift_status | INT | Enum | ✅ EXISTS | |
| is_active | BOOLEAN | DEFAULT true | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |
| first_online_time | TIMESTAMP | NULLABLE | ➕ ADD | First time driver went online in shift |
| final_offline_time | TIMESTAMP | NULLABLE | ➕ ADD | Last time driver went offline |
| total_online_duration_min | INT | DEFAULT 0 | ➕ ADD | Running total in minutes |
| total_trips_completed | INT | DEFAULT 0 | ➕ ADD | |
| total_earnings | DECIMAL(10,2) | DEFAULT 0 | ➕ ADD | |
| total_cancellations | INT | DEFAULT 0 | ➕ ADD | |
| total_no_shows | INT | DEFAULT 0 | ➕ ADD | |

---

### 3.4 `vehicle_details` ✅ NO CHANGE

| Column | Type | Notes |
|--------|------|-------|
| vehicle_id, user_id, vehicle_type_id, vehicle_model_id | INT | Core FK fields |
| registration_number, color, year | VARCHAR/INT | Vehicle identity |
| is_active, created_at, updated_at | | |

---

### 3.5 `vehicle_locations` ✅ NO CHANGE

| Column | Type | Notes |
|--------|------|-------|
| location_id, vehicle_id, user_id | INT | |
| latitude, longitude | DECIMAL | Live GPS |
| recorded_at | TIMESTAMP | |

> **Note:** Redis mirrors this for real-time; DB is fallback/history only.

---

### 3.6 `vehicle_types`, `vehicle_models` ✅ NO CHANGE

---

### 3.7 `driver_vehicle_link` ✅ NO CHANGE

| Column | Notes |
|--------|-------|
| link_id, driver_id (user_id), vehicle_id | FK |
| approval_status_id, start_date, end_date | |
| is_active, created_at, updated_at | |

---

### 3.8 `driver_vehicle_link_history` ✅ NO CHANGE

---

## MODULE 4 - RIDE LIFECYCLE

### 4.1 `ride_types` ✅ NO CHANGE

| Column | Notes |
|--------|-------|
| ride_type_id, ride_type_name, description, is_active | Core lookup |

---

### 4.2 `ride_pricing` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id (pricing_id) | SERIAL | PK | ✅ EXISTS | |
| ride_type_id | INT | FK -> ride_types, IDX | ✅ EXISTS | |
| vehicle_type_id | INT | FK -> vehicle_types, IDX | ✅ EXISTS | |
| base_fare | DECIMAL(10,2) | NOT NULL | ✅ EXISTS | |
| per_km_rate | DECIMAL(10,2) | NOT NULL | ✅ EXISTS | |
| per_min_rate | DECIMAL(10,2) | NOT NULL | ✅ EXISTS | |
| minimum_fare | DECIMAL(10,2) | | ✅ EXISTS | |
| waiting_charge | DECIMAL(10,2) | | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | | ✅ EXISTS | |
| surge_enabled | BOOLEAN | DEFAULT false | ➕ ADD | |
| surge_multiplier | DECIMAL(4,2) | CHECK > 1.0 | ➕ ADD | |
| surge_start_at, surge_end_at | TIMESTAMP | NULLABLE | ➕ ADD | |
| free_cancellation_window_min | INT | DEFAULT 5 | ➕ ADD | Minutes after booking |
| post_accept_cancel_fee | DECIMAL(10,2) | DEFAULT 0 | ➕ ADD | |
| post_arrival_cancel_fee | DECIMAL(10,2) | DEFAULT 0 | ➕ ADD | |
| effective_from | TIMESTAMP | DEFAULT now() | ➕ ADD | |
| effective_to | TIMESTAMP | NULLABLE | ➕ ADD | |
| is_published | BOOLEAN | DEFAULT false | ➕ ADD | Only published rules apply |

---

### 4.3 `ride_peak_hours` ✅ NO CHANGE

---

### 4.4 `ride_requests` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| ride_request_id | SERIAL | PK | ✅ EXISTS | |
| user_id | INT | FK -> users, IDX | ✅ EXISTS | Rider |
| pick_up_location | TEXT | NOT NULL | ✅ EXISTS | |
| drop_off_location | TEXT | NULLABLE | ✅ EXISTS | |
| pick_up_lat/lng, drop_off_lat/lng | DECIMAL | | ✅ EXISTS | |
| ride_type_id | INT | FK -> ride_types | ✅ EXISTS | |
| vehicle_type_id | INT | FK -> vehicle_types | ✅ EXISTS | |
| rental_hours | INT | NULLABLE | ✅ EXISTS | Package rides |
| outstation_return | BOOLEAN | NULLABLE | ✅ EXISTS | |
| status | VARCHAR(50) | NOT NULL | ✅ EXISTS | |
| estimated_fare/distance/duration | DECIMAL/INT | | ✅ EXISTS | |
| pick_up_datetime, drop_off_datetime | TIMESTAMP | | ✅ EXISTS | |
| is_active, created_at, updated_at | | | ✅ EXISTS | |
| booking_source | VARCHAR(30) | DEFAULT 'rider_app' | ➕ ADD | rider_app / admin / phone / support |
| assigned_driver_id | INT | FK -> users, NULLABLE | ➕ ADD | Denormalized for fast read |
| scheduled_pickup_at | TIMESTAMP | NULLABLE | ➕ ADD | Scheduled/advance booking |
| promo_code | VARCHAR(30) | NULLABLE | ➕ ADD | Applied promo |
| points_redeemed | INT | DEFAULT 0 | ➕ ADD | Loyalty points used |
| collection_mode_preference | VARCHAR(30) | NULLABLE | ➕ ADD | cash / upi / other |
| cancellation_policy_snapshot_json | JSONB | NULLABLE | ➕ ADD | Policy at time of booking |
| idempotency_key | UUID | UNQ, IDX | ➕ ADD | Prevent duplicate bookings on retry |
| current_sequence_no | INT | DEFAULT 0 | ➕ ADD | Real-time bus ordering |

---

### 4.5 `ride_trips` ✅ EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| ride_trip_id | SERIAL | PK | ✅ EXISTS | |
| ride_request_id | INT | FK -> ride_requests | ✅ EXISTS | |
| driver_id | INT | FK -> users, IDX | ✅ EXISTS | |
| vehicle_id | INT | FK -> vehicle_details | ✅ EXISTS | |
| start_time, end_time | TIMESTAMP | | ✅ EXISTS | |
| distance, fare | DECIMAL | | ✅ EXISTS | |
| status | VARCHAR(50) | NOT NULL | ✅ EXISTS | |
| start_otp, end_otp | VARCHAR(10) | NULLABLE | ✅ EXISTS | |
| is_active, created_at, updated_at | | | ✅ EXISTS | |
| trip_type | VARCHAR(20) | NOT NULL | ➕ ADD | local / outstation / package |
| estimated_fare_snapshot | DECIMAL(10,2) | NULLABLE | ➕ ADD | Fare at booking time |
| fare_breakdown_json | JSONB | NULLABLE | ➕ ADD | {base, distance, surge, tax, total} |
| cancellation_fee | DECIMAL(10,2) | DEFAULT 0 | ➕ ADD | |
| driver_commission_amount | DECIMAL(10,2) | NULLABLE | ➕ ADD | |
| package_deduction_amount | DECIMAL(10,2) | DEFAULT 0 | ➕ ADD | |
| rider_rating_submitted_at | TIMESTAMP | NULLABLE | ➕ ADD | |
| driver_rating_submitted_at | TIMESTAMP | NULLABLE | ➕ ADD | |
| force_end_reason | TEXT | NULLABLE | ➕ ADD | Admin force-end reason |
| no_show_reason | TEXT | NULLABLE | ➕ ADD | |
| user_id | INT | FK -> users | ➕ ADD | Rider; denormalized from request |

---

### 4.6 `ride_bookings` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| booking_id | SERIAL | PK | |
| ride_request_id | INT | FK -> ride_requests, UNQ | One booking per request |
| assigned_driver_id | INT | FK -> users, IDX | |
| assignment_time | TIMESTAMP | | |
| estimated_fare | DECIMAL(10,2) | | At time of assignment |
| surge_details_json | JSONB | NULLABLE | If surge applied |
| booking_status | VARCHAR(30) | NOT NULL | pending / confirmed / cancelled |
| cancellation_policy_snapshot_json | JSONB | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 4.7 `ride_history` ✅ EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| ride_trip_id, user_id, driver_id, vehicle_id | INT | ✅ EXISTS | |
| pickup_location, dropoff_location | TEXT | ✅ EXISTS | |
| ride_type, status | VARCHAR | ✅ EXISTS | |
| cancellation_status, cancelled_by, cancellation_reason | | ✅ EXISTS | |
| timestamps, total_fare, total_distance, ride_duration | | ✅ EXISTS | |
| is_active, created_at, updated_at | | ✅ EXISTS | |
| booking_source | VARCHAR(30) | ➕ ADD | |
| cancellation_actor_type | VARCHAR(20) | ➕ ADD | rider / driver / admin / system |
| dispute_flag | BOOLEAN | DEFAULT false | ➕ ADD | |
| package_impact_json | JSONB | NULLABLE | ➕ ADD | Package deduction summary |

---

### 4.8 `ride_cancellations` ✅ EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| cancellation_id, ride_trip_id, user_id | INT | ✅ EXISTS | |
| cancellation_reason | TEXT | ✅ EXISTS | |
| cancellation_timestamp | TIMESTAMP | ✅ EXISTS | |
| is_active, created_at, updated_at | | ✅ EXISTS | |
| cancelled_by_role | VARCHAR(20) | ➕ ADD | rider / driver / admin / system |
| cancellation_stage | VARCHAR(20) | ➕ ADD | pre_accept / post_accept / post_arrival |
| cancellation_fee_charged | DECIMAL(10,2) | ➕ ADD | |

---

### 4.9 `ride_payments` ✅ EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| payment_id, ride_trip_id, user_id, driver_id, vehicle_id | INT | ✅ EXISTS | |
| payment_method, payment_status, payment_amount | | ✅ EXISTS | |
| payment_timestamp, transaction_id, payment_gateway | | ✅ EXISTS | |
| payment_details, is_active, created_at, updated_at | | ✅ EXISTS | |
| payer_type | VARCHAR(20) | ➕ ADD | rider / driver / platform |
| purpose | VARCHAR(30) | ➕ ADD | ride_fare / package_purchase / refund |
| promo_discount_amount | DECIMAL(10,2) | ➕ ADD | |
| tax_amount | DECIMAL(10,2) | ➕ ADD | |
| net_amount | DECIMAL(10,2) | ➕ ADD | After discounts and tax |
| settlement_reference | VARCHAR(100) | ➕ ADD | External settlement ID |

---

### 4.10 `trip_events` 🆕 NEW (Reliability / Outbox Pattern)

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| event_id | UUID | PK, DEFAULT gen_random_uuid() | |
| trip_id | INT | FK -> ride_trips, IDX | |
| event_type | VARCHAR(50) | NOT NULL | status_changed / otp_verified / ended etc |
| sequence_no | INT | NOT NULL | Strict ordering |
| payload_json | JSONB | NULLABLE | |
| persisted_at | TIMESTAMP | DEFAULT now() | |
| INDEX | | (trip_id, sequence_no) | |

---

### 4.11 `outbox_events` 🆕 NEW (Reliability)

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| outbox_id | SERIAL | PK | |
| aggregate_id | VARCHAR(100) | NOT NULL, IDX | Entity reference (trip_id, booking_id) |
| aggregate_type | VARCHAR(50) | NOT NULL | ride_trip / ride_request |
| event_type | VARCHAR(50) | NOT NULL | |
| payload_json | JSONB | NOT NULL | |
| status | VARCHAR(20) | DEFAULT 'pending' | pending / sent / failed / dead_lettered |
| retry_count | INT | DEFAULT 0 | |
| next_retry_at | TIMESTAMP | NULLABLE | IDX |
| dead_lettered_at | TIMESTAMP | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 4.12 `outbox_delivery_attempts` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| attempt_id | SERIAL | PK | |
| outbox_id | INT | FK -> outbox_events, IDX | |
| provider | VARCHAR(50) | NOT NULL | azure_service_bus / redis |
| status | VARCHAR(20) | | |
| response_detail | TEXT | NULLABLE | |
| attempted_at | TIMESTAMP | DEFAULT now() | |

---

## MODULE 5 - DRIVER PACKAGES & MONETIZATION

### 5.1 `driver_packages` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| package_id | SERIAL | PK | |
| package_name | VARCHAR(255) | NOT NULL | |
| description | TEXT | NULLABLE | |
| price | DECIMAL(10,2) | NOT NULL | |
| validity_days | INT | NOT NULL | |
| ride_categories_json | JSONB | NOT NULL | {local, outstation, package} enabled flags |
| commission_benefit_json | JSONB | NULLABLE | Override commission rates |
| activation_rules_json | JSONB | NULLABLE | |
| is_active | BOOLEAN | DEFAULT true | |
| created_at, updated_at | TIMESTAMP | | |

---

### 5.2 `driver_package_purchases` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| purchase_id | SERIAL | PK | |
| driver_id | INT | FK -> users, IDX | |
| package_id | INT | FK -> driver_packages | |
| amount | DECIMAL(10,2) | NOT NULL | |
| promo_code | VARCHAR(30) | NULLABLE | |
| discount_amount | DECIMAL(10,2) | DEFAULT 0 | |
| payment_mode | VARCHAR(30) | NOT NULL | |
| payment_ref | VARCHAR(100) | NULLABLE | |
| status | VARCHAR(20) | NOT NULL | pending / active / expired / refunded |
| purchased_at | TIMESTAMP | DEFAULT now() | |
| activated_at | TIMESTAMP | NULLABLE | |
| expires_at | TIMESTAMP | NULLABLE | IDX |
| auto_renew | BOOLEAN | DEFAULT false | |

---

### 5.3 `driver_package_transactions` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| txn_id | SERIAL | PK | |
| purchase_id | INT | FK -> driver_package_purchases, IDX | |
| event_type | VARCHAR(50) | NOT NULL | activated / expired / refunded / renewed |
| actor_user_id | INT | FK -> users | |
| actor_role | VARCHAR(20) | | driver / admin / system |
| remarks | TEXT | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 5.4 `package_refund_claims` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| claim_id | SERIAL | PK | |
| purchase_id | INT | FK -> driver_package_purchases | |
| driver_id | INT | FK -> users, IDX | |
| status | VARCHAR(20) | NOT NULL | pending / approved / rejected |
| reason_code | VARCHAR(50) | NULLABLE | |
| created_by | INT | FK -> users | |
| created_at | TIMESTAMP | DEFAULT now() | |
| final_decision_by | INT | FK -> users, NULLABLE | |
| final_decision_at | TIMESTAMP | NULLABLE | |

---

### 5.5 `package_refund_evidence` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| evidence_id | SERIAL | PK | |
| claim_id | INT | FK -> package_refund_claims, UNQ | One evidence per claim |
| online_duration_min | INT | NULLABLE | |
| dispatch_attempts | INT | DEFAULT 0 | |
| offers_received | INT | DEFAULT 0 | |
| accepted_count | INT | DEFAULT 0 | |
| declined_count | INT | DEFAULT 0 | |
| no_response_count | INT | DEFAULT 0 | |
| geo_zone_coverage_json | JSONB | NULLABLE | Zones where dispatch was attempted |
| created_at | TIMESTAMP | DEFAULT now() | |

---

## MODULE 6 - RIDER ENGAGEMENT (Plugin: Conditional)

### 6.1 `saved_addresses` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| address_id | SERIAL | PK | |
| user_id | INT | FK -> users, IDX | |
| label | VARCHAR(50) | NOT NULL | Home / Work / Other |
| full_address | TEXT | NOT NULL | |
| latitude | DECIMAL(10,7) | NOT NULL | |
| longitude | DECIMAL(10,7) | NOT NULL | |
| landmark | TEXT | NULLABLE | |
| is_default | BOOLEAN | DEFAULT false | |
| last_used_at | TIMESTAMP | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 6.2 `referral_events` 🆕 NEW (Plugin: referral_enabled)

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| referral_event_id | SERIAL | PK | |
| referrer_user_id | INT | FK -> users, IDX | |
| referred_user_id | INT | FK -> users | |
| referral_code | VARCHAR(20) | NOT NULL | |
| qualification_status | VARCHAR(20) | NOT NULL | pending / qualified / rewarded |
| reward_type | VARCHAR(20) | NULLABLE | points / cash / credit |
| reward_value | DECIMAL(10,2) | NULLABLE | |
| credited_at | TIMESTAMP | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 6.3 `loyalty_ledger` 🆕 NEW (Plugin: loyalty_enabled)

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| transaction_id | SERIAL | PK | |
| user_id | INT | FK -> users, IDX | |
| ride_trip_id | INT | FK -> ride_trips, NULLABLE | |
| transaction_type | VARCHAR(20) | NOT NULL | earn / redeem / expire / adjust |
| points_delta | INT | NOT NULL, CHECK != 0 | Positive = earn, Negative = redeem |
| balance_after | INT | NOT NULL, CHECK >= 0 | Running balance |
| expiry_at | TIMESTAMP | NULLABLE | |
| remarks | TEXT | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

## MODULE 7 - SUPPORT & ESCALATION

### 7.1 `tickets` ✅ EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| ticket_id, user_id, subject, description | | ✅ EXISTS | |
| status, priority, assigned_to_user_id | | ✅ EXISTS | |
| created_at, updated_at | TIMESTAMP | ✅ EXISTS | |
| user_type | VARCHAR(20) | ➕ ADD | rider / driver |
| query_type_id | INT | ➕ ADD | FK -> enum_value_catalog (ticket category) |
| linked_ride_trip_id | INT | ➕ ADD | FK -> ride_trips, NULLABLE |
| linked_package_purchase_id | INT | ➕ ADD | FK -> driver_package_purchases, NULLABLE |
| sla_due_at | TIMESTAMP | ➕ ADD | Response SLA deadline |
| first_response_at | TIMESTAMP | ➕ ADD | |
| resolved_at | TIMESTAMP | ➕ ADD | |
| escalation_level | INT | DEFAULT 0 | ➕ ADD | 0=none, 1=L1, 2=L2, 3=management |
| escalated_to_user_id | INT | ➕ ADD | FK -> users, NULLABLE |
| resolution_code | VARCHAR(50) | ➕ ADD | |
| resolution_note_public | TEXT | ➕ ADD | Visible to rider/driver |

---

### 7.2 `ticket_comments` ✅ EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| comment_id, ticket_id, user_id, comment_text, created_at | | ✅ EXISTS | |
| is_internal_note | BOOLEAN | ➕ ADD | Hidden from rider/driver |
| attachment_url | TEXT | ➕ ADD | NULLABLE |
| actor_role | VARCHAR(20) | ➕ ADD | rider / driver / support / admin |

---

### 7.3 `ticket_actions` ✅ NO CHANGE

---

### 7.4 `sos_alerts` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| sos_id | SERIAL | PK | |
| user_id | INT | FK -> users, IDX | Rider or driver who triggered |
| ride_trip_id | INT | FK -> ride_trips, NULLABLE | |
| triggered_at | TIMESTAMP | NOT NULL | |
| latitude | DECIMAL(10,7) | NOT NULL | |
| longitude | DECIMAL(10,7) | NOT NULL | |
| status | VARCHAR(20) | NOT NULL | open / acknowledged / resolved |
| escalation_history_json | JSONB | NULLABLE | [{timestamp, status_from, status_to, actor}] |
| resolved_at | TIMESTAMP | NULLABLE | |
| resolved_by | INT | FK -> users, NULLABLE | |

---

### 7.5 `policy_violation_alerts` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| alert_id | SERIAL | PK | |
| driver_id | INT | FK -> users, IDX | |
| violation_type | VARCHAR(50) | NOT NULL | no_package / idle_too_long / complaint |
| severity | VARCHAR(20) | NOT NULL | info / warning / critical |
| action_required | TEXT | NULLABLE | |
| status | VARCHAR(20) | NOT NULL | open / acknowledged / actioned |
| acknowledged_at | TIMESTAMP | NULLABLE | |
| actioned_at | TIMESTAMP | NULLABLE | |
| actioned_by | INT | FK -> users, NULLABLE | |
| expires_at | TIMESTAMP | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

## MODULE 8 - GOVERNANCE & APPROVALS

### 8.1 `approval_history` ✅ EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| id, entity_id, entity_type | | ✅ EXISTS | |
| previous_status_id, new_status_id | | ✅ EXISTS | |
| changed_by, changed_at | | ✅ EXISTS | |
| review_notes | | ✅ EXISTS | |
| workflow_type | VARCHAR(50) | ➕ ADD | driver_onboarding / document / high_impact |
| workflow_step | INT | DEFAULT 1 | ➕ ADD | Step # in multi-stage workflow |
| decision | VARCHAR(20) | ➕ ADD | approved / rejected / verified |
| old_value_json | JSONB | ➕ ADD | Pre-change snapshot |
| new_value_json | JSONB | ➕ ADD | Post-change snapshot |
| change_request_id | INT | ➕ ADD | FK -> high_impact_change_requests, NULLABLE |

---

### 8.2 `high_impact_change_requests` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| request_id | SERIAL | PK | |
| module | VARCHAR(50) | NOT NULL | pricing / driver / support / config |
| target_entity | VARCHAR(100) | NOT NULL | Table or config key name |
| old_value_json | JSONB | NULLABLE | |
| new_value_json | JSONB | NOT NULL | |
| rationale | TEXT | NULLABLE | |
| impact_note | TEXT | NULLABLE | |
| effective_time | TIMESTAMP | NULLABLE | |
| status | VARCHAR(20) | NOT NULL | pending / approved / rejected / applied |
| created_by | INT | FK -> users | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 8.3 `high_impact_change_approvals` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| approval_id | SERIAL | PK | |
| request_id | INT | FK -> high_impact_change_requests, IDX | |
| step_role | VARCHAR(30) | NOT NULL | lead / manager / director |
| step_order | INT | NOT NULL | 1, 2, 3 |
| decision | VARCHAR(20) | NULLABLE | approved / rejected |
| decision_note | TEXT | NULLABLE | |
| acted_by | INT | FK -> users, NULLABLE | |
| acted_at | TIMESTAMP | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

## MODULE 9 - AUDIT LOGGING

### 9.1 `immutable_audit_logs` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| audit_id | BIGSERIAL | PK | Large primary key for scale |
| actor_user_id | INT | FK -> users, IDX | |
| actor_role | VARCHAR(20) | NOT NULL | |
| module | VARCHAR(50) | NOT NULL, IDX | |
| entity_name | VARCHAR(100) | NOT NULL, IDX | Table name |
| entity_id | VARCHAR(50) | NOT NULL, IDX | Entity PK (stored as string) |
| action_type | VARCHAR(20) | NOT NULL | create / update / delete / view |
| old_value_json | JSONB | NULLABLE | |
| new_value_json | JSONB | NULLABLE | |
| source | VARCHAR(50) | NOT NULL | api / admin_portal / system / event_bus |
| occurred_at | TIMESTAMP | DEFAULT now(), IDX | |

> **Critical:** No UPDATE or DELETE permitted on this table. Enforce via DB trigger or row-level security.

---

## MODULE 10 - QUOTE DESK

### 10.1 `quote_requests` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| quote_id | SERIAL | PK | |
| user_id | INT | FK -> users, NULLABLE | If raised by logged-in user |
| quote_type | VARCHAR(20) | NOT NULL | outstation / package |
| pickup_location | TEXT | NOT NULL | |
| drop_location | TEXT | NULLABLE | |
| pickup_lat, pickup_lng | DECIMAL | | |
| est_hours | INT | NULLABLE | |
| est_days | INT | NULLABLE | |
| est_km | DECIMAL | NULLABLE | |
| estimated_amount_min | DECIMAL(10,2) | NULLABLE | |
| estimated_amount_max | DECIMAL(10,2) | NULLABLE | |
| validity_until | TIMESTAMP | NULLABLE | |
| quoted_by | INT | FK -> users, NULLABLE | Admin who quoted |
| source | VARCHAR(20) | NOT NULL | admin / phone / portal |
| status | VARCHAR(20) | NOT NULL | draft / shared / accepted / expired |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 10.2 `quote_booking_links` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| id | SERIAL | PK | |
| quote_id | INT | FK -> quote_requests | |
| ride_request_id | INT | FK -> ride_requests | |
| linked_at | TIMESTAMP | DEFAULT now() | |

---

## MODULE 11 - REPORTING

### 11.1 `report_templates` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| template_id | SERIAL | PK | |
| name | VARCHAR(255) | NOT NULL | |
| domain | VARCHAR(50) | NOT NULL | rides / drivers / revenue / support |
| filters_json | JSONB | NULLABLE | |
| dimensions_json | JSONB | NULLABLE | |
| metrics_json | JSONB | NULLABLE | |
| visibility_scope | VARCHAR(20) | DEFAULT 'admin' | admin / management |
| created_by | INT | FK -> users | |
| created_at | TIMESTAMP | DEFAULT now() | |

---

### 11.2 `report_schedules` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| schedule_id | SERIAL | PK | |
| template_id | INT | FK -> report_templates, IDX | |
| frequency | VARCHAR(20) | NOT NULL | daily / weekly / monthly |
| next_run_at | TIMESTAMP | IDX | |
| recipients_json | JSONB | NOT NULL | [email, email, ...] |
| output_format | VARCHAR(10) | DEFAULT 'csv' | csv / xlsx / pdf |
| is_enabled | BOOLEAN | DEFAULT true | |

---

### 11.3 `report_runs` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| run_id | SERIAL | PK | |
| template_id | INT | FK -> report_templates, IDX | |
| started_at | TIMESTAMP | NOT NULL | |
| completed_at | TIMESTAMP | NULLABLE | |
| status | VARCHAR(20) | NOT NULL | running / completed / failed |
| file_url | TEXT | NULLABLE | |
| generated_by | INT | FK -> users, NULLABLE | NULL = scheduled |
| created_at | TIMESTAMP | DEFAULT now() | |

---

## MODULE 12 - NOTIFICATIONS

### 12.1 `notification_types` ✅ NO CHANGE

### 12.2 `notification_inbox` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| notification_id | BIGSERIAL | PK | |
| user_id | INT | FK -> users, IDX | |
| role | VARCHAR(20) | NULLABLE | rider / driver / admin |
| category | VARCHAR(50) | NOT NULL | operational / marketing / safety |
| title | VARCHAR(255) | NOT NULL | |
| body | TEXT | NOT NULL | |
| payload_json | JSONB | NULLABLE | {action_url, deep_link, icon_url} |
| is_read | BOOLEAN | DEFAULT false | |
| created_at | TIMESTAMP | DEFAULT now(), IDX | |
| expires_at | TIMESTAMP | NULLABLE | |

---

## MODULE 13 - MISC / LOOKUP (Existing, No Change)

| Table | Status | Notes |
|-------|--------|-------|
| `banners` | ✅ NO CHANGE | Promotional banners |
| `ratings` | ✅ NO CHANGE | Rider/driver ratings |
| `feedback` | ✅ NO CHANGE | General feedback |
| `documents` + `document_types` | ✅ MINOR EXTEND | See column additions below |
| `frequently_asked_questions` | ✅ NO CHANGE | |
| `general_queries` | ✅ NO CHANGE | |
| `language` (meta_language) | ✅ NO CHANGE | |
| `coupons` | ✅ EXTEND | See column additions below |
| `bids` | ✅ NO CHANGE | (If bids still in scope) |

---

### 13.1 `documents` ✅ EXTEND

| Column | Status | Notes |
|--------|--------|-------|
| All existing columns | ✅ EXISTS | |
| expiry_date | ➕ ADD | DATE - document legal expiry |
| renewal_required_from | ➕ ADD | DATE - reminder threshold |
| rejection_code | ➕ ADD | VARCHAR(50) - catalog code for rejection |
| source_channel | ➕ ADD | VARCHAR(20) - app / admin |

---

### 13.2 `coupons` ✅ EXTEND

| Column | Status | Notes |
|--------|--------|-------|
| All existing columns | ✅ EXISTS | |
| applicable_ride_types | ➕ ADD | JSONB - [local, outstation, package] |
| per_user_limit | ➕ ADD | INT - max uses per user |
| total_redemption_limit | ➕ ADD | INT - global cap |
| current_redemption_count | ➕ ADD | INT DEFAULT 0 |
| default_for_new_driver | ➕ ADD | BOOLEAN DEFAULT false |
| target_audience | ➕ ADD | VARCHAR(10) - rider / driver / both |

---

## MODULE 2 - DEPLOYMENT & CONFIGURATION

### Module 2 Quick Understanding (Why 8 tables?)

This module separates platform configuration into clear layers so every deployment (company) can have its own behavior without code changes.

> **Design Decision (Updated):** Module 2 remains independent from core domain tables. No hard FK from Module 2 tables to core tables (such as `users`, `ride_requests`, `driver_details`).

> **Architecture Clarification:** Because each customer has a separate backend/database deployment, `deployment_id` is not required on core/business tables. It is relevant only inside this independent configuration module.

| Layer | Table(s) | Purpose |
|------|----------|----------|
| Deployment identity | `deployment_profiles` | Master record for one deployment/company |
| Deployment branding | `deployment_branding` | Name/logo/colors/support contact for that deployment |
| Feature toggles | `deployment_feature_flags` | On/off switches (referral, loyalty, package, quote desk, etc.) |
| Business rules | `deployment_business_rules` | Dynamic rule values (cancel window, thresholds, limits) |
| Key registry | `configuration_keys_catalog` | Global dictionary of allowed config keys and value types |
| Enum registry | `enum_value_catalog` | Allowed enum values for catalog-backed keys |
| Deployment values | `deployment_configuration_values` | Actual runtime value per deployment for each key |
| Audit trail | `configuration_change_history` | Human-readable history of who changed what and why |

### Module 2 Relationship Model

1. Create one row in `deployment_profiles` for each deployment.
2. Add optional single-row companions per deployment:
	- `deployment_branding` (1:1)
	- `deployment_feature_flags` (1:1)
3. Store frequently changing dynamic rules in `deployment_business_rules` (1:many).
4. Register all supported keys once in `configuration_keys_catalog` (global catalog).
5. If a key is enum-based, store allowed values in `enum_value_catalog`.
6. Store each deployment's active value in `deployment_configuration_values`.
7. Record every admin change in `configuration_change_history`.

### Decoupling Rules (Module 2 vs Core)

1. Module 2 configuration records remain independent and runtime value-driven.
2. No database FK from Module 2 to core tables.
3. No database FK inside configuration value tables; use logical keys/IDs only.
4. Actor tracking fields like `updated_by` / `changed_by` are stored as plain IDs (audit reference), not FK.
5. Core module tables should consume configuration through service/API lookup, not direct relational dependency.

### Runtime Read Priority (Recommended)

When API reads a config value for a deployment:

1. Check `deployment_configuration_values` active row.
2. If missing, fall back to `configuration_keys_catalog.default_value`.
3. If key is enum-based, validate against `enum_value_catalog`.

### Write Rules (Recommended)

1. Validate key exists in `configuration_keys_catalog`.
2. Validate value type using `value_type`.
3. If enum key, value must exist in `enum_value_catalog`.
4. Close current active row by setting `effective_to`.
5. Insert new active row with incremented `version`.
6. Insert audit row in `configuration_change_history`.

### Important Uniqueness Constraints (must keep)

| Constraint | Why |
|-----------|-----|
| `deployment_branding.deployment_id` UNIQUE | Exactly one branding profile per deployment |
| `deployment_feature_flags.deployment_id` UNIQUE | Exactly one feature-flag profile per deployment |
| `deployment_business_rules` UNIQUE active `(deployment_id, rule_key)` | Only one active rule per key |
| `configuration_keys_catalog` UNIQUE `(module, config_key)` | Prevent duplicate key definitions |
| `deployment_configuration_values` UNIQUE active `(deployment_id, key_id)` | Only one active value per key per deployment |

### Example (one key end-to-end)

- Catalog key: `module=ride`, `config_key=free_cancel_window_min`, `value_type=int`, `default_value=5`
- Deployment override for REKLA: `config_value=7`
- API behavior: Rider gets 7-minute free cancel window in REKLA deployment, 5 where no override exists.

### 2.1 `deployment_profiles` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| deployment_id | SERIAL | PK | |
| deployment_code | VARCHAR(50) | NOT NULL, UNQ | Short code: REKLA, CITYCAB etc |
| deployment_name | VARCHAR(255) | NOT NULL | Full company name |
| status | VARCHAR(20) | DEFAULT 'active' | active / suspended / archived |
| contact_email | VARCHAR(255) | NULLABLE | |
| contact_phone | VARCHAR(20) | NULLABLE | |
| country_code | VARCHAR(5) | NULLABLE | |
| timezone | VARCHAR(50) | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | | |

---

### 2.2 `deployment_branding` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| branding_id | SERIAL | PK | |
| deployment_id | INT | UNQ | One branding record per deployment (Module 2 internal reference) |
| app_name | VARCHAR(100) | NOT NULL | |
| logo_url | TEXT | NULLABLE | |
| splash_url | TEXT | NULLABLE | |
| primary_color | VARCHAR(10) | NULLABLE | Hex color |
| secondary_color | VARCHAR(10) | NULLABLE | |
| support_phone | VARCHAR(20) | NULLABLE | |
| support_email | VARCHAR(255) | NULLABLE | |
| privacy_policy_url | TEXT | NULLABLE | |
| terms_url | TEXT | NULLABLE | |
| created_at, updated_at | TIMESTAMP | | |

---

### 2.3 `deployment_feature_flags` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| flag_id | SERIAL | PK | |
| deployment_id | INT | UNQ | Module 2 internal reference |
| referral_enabled | BOOLEAN | DEFAULT false | |
| loyalty_enabled | BOOLEAN | DEFAULT false | |
| promo_enabled | BOOLEAN | DEFAULT true | |
| package_enabled | BOOLEAN | DEFAULT true | |
| quote_desk_enabled | BOOLEAN | DEFAULT false | |
| report_builder_enabled | BOOLEAN | DEFAULT false | |
| sos_enabled | BOOLEAN | DEFAULT true | |
| in_app_payment_enabled | BOOLEAN | DEFAULT false | |
| updated_at | TIMESTAMP | | |
| updated_by | INT | NULLABLE | Actor user ID snapshot (no FK to core tables) |

---

### 2.4 `deployment_business_rules` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| rule_id | SERIAL | PK | |
| deployment_id | INT | IDX | Module 2 internal reference |
| rule_key | VARCHAR(100) | NOT NULL, IDX | e.g. free_cancel_window_min |
| rule_value_json | JSONB | NOT NULL | Typed value |
| version | INT | DEFAULT 1 | |
| effective_from | TIMESTAMP | DEFAULT now() | |
| effective_to | TIMESTAMP | NULLABLE | |
| updated_by | INT | NULLABLE | Actor user ID snapshot (no FK to core tables) |
| UNIQUE | | (deployment_id, rule_key) WHERE effective_to IS NULL | Active rule uniqueness |

---

### 2.5 `configuration_keys_catalog` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| key_id | SERIAL | PK | |
| module | VARCHAR(50) | NOT NULL, IDX | ride / driver / support / pricing |
| config_key | VARCHAR(100) | NOT NULL | |
| value_type | VARCHAR(20) | NOT NULL | string / int / bool / decimal / json / enum |
| default_value | TEXT | NULLABLE | |
| description | TEXT | NULLABLE | |
| is_enum_catalog | BOOLEAN | DEFAULT false | True if key is an enum list |
| is_editable | BOOLEAN | DEFAULT true | Some keys are system-locked |
| created_at | TIMESTAMP | DEFAULT now() | |
| UNIQUE | | (module, config_key) | |

---

### 2.6 `enum_value_catalog` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| enum_value_id | SERIAL | PK | |
| key_id | INT | IDX | Logical key reference (no FK) |
| enum_code | VARCHAR(50) | NOT NULL | Machine code: RIDE_LOCAL |
| enum_label | VARCHAR(100) | NOT NULL | Display: "Local Ride" |
| sort_order | INT | DEFAULT 0 | |
| is_active | BOOLEAN | DEFAULT true | |
| metadata_json | JSONB | NULLABLE | icon_url, color, extra flags |
| UNIQUE | | (key_id, enum_code) | |

---

### 2.7 `deployment_configuration_values` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| value_id | SERIAL | PK | |
| deployment_id | INT | IDX | Module 2 internal reference |
| key_id | INT | IDX | Logical key reference (no FK) |
| config_value | TEXT | NOT NULL | Stored as string, typed by key |
| version | INT | DEFAULT 1 | |
| effective_from | TIMESTAMP | DEFAULT now() | |
| effective_to | TIMESTAMP | NULLABLE | |
| changed_by | INT | NULLABLE | Actor user ID snapshot (no FK to core tables) |
| changed_at | TIMESTAMP | DEFAULT now() | |
| UNIQUE | | (deployment_id, key_id) WHERE effective_to IS NULL | |

---

### 2.8 `configuration_change_history` 🆕 NEW

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| change_id | SERIAL | PK | |
| deployment_id | INT | IDX | Module 2 internal reference |
| module | VARCHAR(50) | NOT NULL | |
| table_name | VARCHAR(100) | NULLABLE | |
| field_name | VARCHAR(100) | NOT NULL | |
| old_value | TEXT | NULLABLE | |
| new_value | TEXT | NULLABLE | |
| changed_by | INT | NULLABLE | Actor user ID snapshot (no FK to core tables) |
| changed_at | TIMESTAMP | DEFAULT now() | |
| change_reason | TEXT | NULLABLE | |

---

## SUMMARY COUNTS

| Category | Tables | Status |
|----------|--------|--------|
| Identity & Auth | 6 tables | 5 extend, 0 new |
| Deployment & Config | 8 tables | 0 extend, 8 new |
| Driver | 7 tables | 4 extend, 0 new |
| Ride Lifecycle | 11 tables | 5 extend, 4 new |
| Driver Packages | 5 tables | 0 extend, 5 new |
| Rider Engagement | 3 tables | 0 new plugins, 3 new |
| Support | 5 tables | 3 extend, 2 new |
| Governance | 3 tables | 1 extend, 2 new |
| Audit | 1 table | 0 extend, 1 new |
| Quote Desk | 2 tables | 0 extend, 2 new |
| Reporting | 3 tables | 0 extend, 3 new |
| Notifications | 2 tables | 0 extend, 1 new |
| Misc/Lookup | 8 tables | 2 extend, 0 new |
| **TOTAL** | **64 tables** | **20 extend, 31 new** |

---

## MIGRATION PHASE ASSIGNMENT

| Phase | Tables Included |
|-------|----------------|
| **M1 - Foundation** | deployment_profiles, deployment_branding, deployment_feature_flags, deployment_business_rules, configuration_keys_catalog, enum_value_catalog, deployment_configuration_values, configuration_change_history + user_refresh_tokens token_hash + users auth columns |
| **M2 - Ride Lifecycle** | ride_bookings, trip_events, outbox_events, outbox_delivery_attempts + extend ride_requests, ride_trips, ride_payments, ride_history, ride_cancellations + extend ride_pricing |
| **M3 - Driver Packages** | driver_packages, driver_package_purchases, driver_package_transactions, package_refund_claims, package_refund_evidence + extend driver_details, driver_status, driver_shift_log |
| **M4 - Support & Governance** | sos_alerts, policy_violation_alerts, high_impact_change_requests, high_impact_change_approvals, immutable_audit_logs + extend tickets, ticket_comments, approval_history |
| **M5 - Growth Modules** | saved_addresses, referral_events, loyalty_ledger, quote_requests, quote_booking_links, report_templates, report_schedules, report_runs, notification_inbox + extend user_details, user_notification_settings, documents, coupons |




