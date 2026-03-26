# Final Table Master List - ZhooCars / Rekla Platform
## Document ID: SCHEMA-MASTER-001
## Version: 1.2 | Date: March 25, 2026
## Status: Design - Pre-Implementation

---

## How to Read this Document

| Symbol | Meaning |
|--------|---------|
| ✅ | Column/table already in database |
| ➕ | New column to add to existing table |
| 🆕 | Entirely new table to create |
| PK | Primary key |
| FK | Foreign key |
| IDX | Needs index |
| UNQ | Unique constraint |

Tables are grouped by **module**. Within each table, existing columns are listed first, then columns to add.

Core design rule for this version:
- Keep `users` and `ride_bookings` lightweight.
- Move high-write audit/security and reliability/SLA details to dedicated side tables.
- V2 is greenfield (not live): remove legacy compatibility columns and assumptions.

## V2 Canonical Section Order (Use This Reading Order)

The detailed module definitions below are complete, but implementation and review should follow this order:

### A. Lookup Tables First

1. Core lookup tables:
- `vehicle_types`
- `vehicle_models`
- `ride_types`
- `roles`
- `notification_types`
- `document_types`
- `language`

2. Central enum/config lookup:
- `configuration_keys_catalog`
- `enum_value_catalog`

### B. Primary Workflow Tables

1. Identity and access:
- `users`
- `user_roles`
- `user_details`
- `user_refresh_tokens`
- `user_login_history`
- `user_notification_settings`
- `user_referral_profiles`

2. Core booking/trip workflow:
- `ride_requests`
- `ride_bookings`
- `ride_booking_reliability`
- `ride_trips`
- `ride_history`
- `ride_cancellations`
- `ride_payments`

### C. Driver App High-End Section

1. Driver profile and operations:
- `driver_details`
- `driver_status`
- `driver_shift_log`
- `driver_vehicle_link`
- `driver_vehicle_link_history`
- `vehicle_details`
- `vehicle_locations`

2. Driver monetization and settlement:
- `driver_packages`
- `driver_package_purchases`
- `driver_package_transactions`
- `package_refund_claims`
- `package_refund_evidence`
- `driver_wallet_ledger`
- `driver_settlement_cycles`
- `driver_settlement_items`
- `driver_payouts`
- `driver_payout_failures`

### D. Rider App High-End Section

1. Rider engagement and reliability:
- `saved_addresses`
- `referral_events`
- `loyalty_ledger`
- `service_zones`
- `geo_fences`
- `serviceability_decisions`
- `location_tamper_events`

### E. Admin High-End Section

1. Support and governance:
- `tickets`
- `ticket_comments`
- `ticket_actions`
- `ticket_status_history`
- `ticket_assignments`
- `ticket_sla_events`
- `approval_history`
- `high_impact_change_requests`
- `high_impact_change_approvals`

2. Admin operations and analytics:
- `quote_requests`
- `quote_booking_links`
- `report_templates`
- `report_schedules`
- `report_runs`
- `sos_alerts`
- `policy_violation_alerts`

3. Deployment and platform configuration:
- `deployment_profiles`
- `deployment_branding`
- `deployment_feature_flags`
- `deployment_business_rules`
- `deployment_configuration_values`
- `configuration_change_history`

### F. Remaining Shared and Platform Tables

1. Reliability and eventing:
- `trip_events`
- `outbox_events`
- `outbox_delivery_attempts`

2. Shared communication/audit/content tables:
- `notification_inbox`
- `immutable_audit_logs`
- `documents`
- `coupons`
- `banners`
- `ratings`
- `feedback`
- `frequently_asked_questions`
- `general_queries`
- `bids`

Implementation note:
- Use this canonical order for design reviews, migration scripts, and team execution.
- Keep existing detailed module definitions below as the field-level source of truth.

---

## MODULE A - LOOKUP & ENUM REGISTRY

### 13.0 Lookup Strategy (V2 Canonical)

Use two layers for all enum-like values:
- Domain lookup tables for stable business entities (for example: `vehicle_types`, `vehicle_models`, `ride_types`, `roles`, `notification_types`, `document_types`).
- Central enum catalog for configurable/app-facing enums: `configuration_keys_catalog` + `enum_value_catalog`.

Examples:
- Vehicle type and ride type come from domain lookup tables (`vehicle_types`, `ride_types`).
- Ticket query types and language preference enums can come from `enum_value_catalog`.

Rule:
- Avoid hardcoded enums in app logic where runtime configurability is required.
- For fixed core identities (like `ride_types`), keep them as first-class lookup tables.

### 13.0A Existing Lookup Inventory

| Table | Status | Notes |
|-------|--------|-------|
| `banners` | NO CHANGE | Promotional banners |
| `ratings` | NO CHANGE | Rider/driver ratings |
| `feedback` | NO CHANGE | General feedback |
| `documents` + `document_types` | MINOR EXTEND | See column additions below |
| `frequently_asked_questions` | NO CHANGE | |
| `general_queries` | NO CHANGE | |
| `language` (meta_language) | NO CHANGE | |
| `coupons` | EXTEND | See column additions below |
| `bids` | NO CHANGE | (If bids still in scope) |

---

### 13.1 `documents` EXTEND

| Column | Status | Notes |
|--------|--------|-------|
| document_id | ✅ | INT PK |
| vehicle_id | ✅ | INT NULLABLE FK -> vehicle_details |
| user_id | ✅ | INT FK -> users |
| document_type_id | ✅ | INT/ENUM FK -> document_types |
| document_no | ✅ | VARCHAR(50) - document number |
| document_url | ✅ | VARCHAR(255) - stored document URL |
| approval_status_id | ✅ | INT/ENUM - approval status |
| approval_date | ✅ | TIMESTAMP NULLABLE - approval timestamp |
| approved_by | ✅ | INT NULLABLE FK -> users |
| review_notes | ✅ | TEXT NULLABLE - reviewer comments |
| is_active | ✅ | BOOLEAN DEFAULT true |
| created_at | ✅ | TIMESTAMP |
| updated_at | ✅ | TIMESTAMP NULLABLE |
| expiry_date | ➕ | DATE - document legal expiry |
| renewal_required_from | ➕ | DATE - reminder threshold |
| rejection_code | ➕ | VARCHAR(50) - catalog code for rejection |
| source_channel | ➕ | VARCHAR(20) - app / admin |

---

### 13.2 `coupons` EXTEND

| Column | Status | Notes |
|--------|--------|-------|
| id | ✅ | INT PK |
| code | ✅ | VARCHAR(50) UNIQUE coupon code |
| description | ✅ | TEXT NULLABLE |
| discount_type | ✅ | VARCHAR - fixed / percentage |
| discount_value | ✅ | DECIMAL - discount amount/value |
| start_date | ✅ | TIMESTAMP - coupon validity start |
| end_date | ✅ | TIMESTAMP - coupon validity end |
| is_active | ✅ | BOOLEAN DEFAULT true |
| min_fare | ✅ | DECIMAL NULLABLE - minimum fare condition |
| max_discount | ✅ | DECIMAL NULLABLE - maximum discount cap |
| created_at | ✅ | TIMESTAMP |
| updated_at | ✅ | TIMESTAMP NULLABLE |
| applicable_ride_types | ➕ | JSONB - [local, outstation, package] |
| per_user_limit | ➕ | INT - max uses per user |
| total_redemption_limit | ➕ | INT - global cap |
| current_redemption_count | ➕ | INT DEFAULT 0 |
| default_for_new_driver | ➕ | BOOLEAN DEFAULT false |
| target_audience | ➕ | VARCHAR(10) - rider / driver / both |

---


## MODULE A2 - DEPLOYMENT & CONFIGURATION (ENUM/CONFIG LAYER)

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

1. Validate key ✅ in `configuration_keys_catalog`.
2. Validate value type using `value_type`.
3. If enum key, value must exist in `enum_value_catalog`.
4. Close current active row by setting `effective_to`.
5. Insert 🆕 active row with incremented `version`.
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
- API behavior: Rider gets 7-minute free cancel window in REKLA deployment, 5 where no override ✅.

### 2.1 `deployment_profiles` 🆕

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

### 2.2 `deployment_branding` 🆕

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
| created_at | TIMESTAMP |  |  |
| updated_at | TIMESTAMP |  |  |

---

### 2.3 `deployment_feature_flags` 🆕

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

### 2.4 `deployment_business_rules` 🆕

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

### 2.5 `configuration_keys_catalog` 🆕

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

### 2.6 `enum_value_catalog` 🆕

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

### 2.7 `deployment_configuration_values` 🆕

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

### 2.8 `configuration_change_history` 🆕

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


## MODULE B1 - PRIMARY WORKFLOW: IDENTITY & AUTHENTICATION (V2 BASELINE - CREATE ALL)

Use this section directly for full database creation scripts. All tables below are treated as V2 create-from-scratch definitions.

### 1.1 `users` ✅ EXISTS

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| user_id | SERIAL | PK | ✅ | |
| phone_number | VARCHAR(20) | NOT NULL, IDX | ✅ | Shared across roles (BR-19) |
| email | VARCHAR(255) | NULLABLE | ✅ | |
| password_hash | TEXT | NULLABLE | ✅ | Admin only |
| status | VARCHAR(50) | DEFAULT 'active' | ✅ | |
| is_active | BOOLEAN | DEFAULT true | ✅ | |
| last_login | TIMESTAMP | NULLABLE | ✅ | |
| created_at | TIMESTAMP | DEFAULT now() | ✅ | |
| updated_at | TIMESTAMP | NULLABLE | ✅ | |

> **V2 Rule:** Do not create `users.role_id`. Multi-role is modeled only by `user_roles`.

> **Design Caution:** Keep `users` minimal because it is a high-frequency shared identity table. Role-specific restriction state must stay in role-owned tables such as `driver_details`, not in `users`.

> **V2:** `referral_code` is not in `users`; it is tracked in `user_referral_profiles`.

---

### 1.2 `user_login_history` 🆕 (V2 BASE)

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | 🆕 | |
| user_id | INT | FK -> users, IDX | 🆕 | |
| ip_address | VARCHAR(50) | NULLABLE | 🆕 | Login IP for security audit |
| device_id | VARCHAR(255) | NULLABLE | 🆕 | Login device for security alert |
| logged_in_at | TIMESTAMP | DEFAULT now() | 🆕 | |

---

### 1.3 `user_details` ✅ EXISTS

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| user_id | INT | PK, FK -> users | ✅ | |
| first_name | VARCHAR(100) | NULLABLE | ✅ | |
| last_name | VARCHAR(100) | NULLABLE | ✅ | |
| profile_picture_url | TEXT | NULLABLE | ✅ | |
| gender | VARCHAR(10) | NULLABLE | ✅ | |
| date_of_birth | DATE | NULLABLE | ✅ | |
| street1 | VARCHAR | NULLABLE | ✅ |  |
| street2 | VARCHAR | NULLABLE | ✅ |  |
| area | VARCHAR | NULLABLE | ✅ |  |
| city | VARCHAR | NULLABLE | ✅ |  |
| state | VARCHAR | NULLABLE | ✅ |  |
| postal_code | VARCHAR | NULLABLE | ✅ |  |
| country | VARCHAR | NULLABLE | ✅ |  |
| emergency_contact_name | VARCHAR(100) | NULLABLE | ✅ | |
| emergency_contact_phone | VARCHAR(20) | NULLABLE | ✅ | |
| language | VARCHAR(20) | NULLABLE | ✅ | UI preference language (shared across roles) |
| timezone | VARCHAR(50) | NULLABLE | ✅ | |
| created_at | TIMESTAMP | DEFAULT now() | ✅ | |
| updated_at | TIMESTAMP | NULLABLE | ✅ | |
| language_preference_id | INT | FK -> enum_value_catalog, NULLABLE | ➕ | NEW - Normalized from free-text language |
| legal_acceptance_version | VARCHAR(50) | NULLABLE | ➕ | NEW - T&C version user accepted |
| legal_accepted_at | TIMESTAMP | NULLABLE | ➕ | NEW - When T&C was accepted |
| default_saved_address_id | INT | FK -> saved_addresses, NULLABLE | ➕ | NEW - Rider's default pickup address |

---

### 1.4 `roles` ✅ EXISTS

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| role_id | SERIAL | PK | ✅ | |
| role_name | VARCHAR(50) | NOT NULL, UNQ | ✅ | rider / driver / admin / support |
| description | TEXT | NULLABLE | ✅ | |
| is_active | BOOLEAN | DEFAULT true | ✅ | |
| created_at | TIMESTAMP | DEFAULT now() | ✅ | |
| updated_at | TIMESTAMP | NULLABLE | ✅ | |

---

### 1.5 `user_roles` ✅ EXISTS

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | ✅ | |
| user_id | INT | FK -> users, IDX | ✅ | |
| role_id | INT | FK -> roles, IDX | ✅ | |
| is_active | BOOLEAN | DEFAULT true | ✅ | |
| created_at | TIMESTAMP | DEFAULT now() | ✅ | |
| updated_at | TIMESTAMP | NULLABLE | ✅ | |

> **Rule:** One user can hold both rider and driver roles (BR-19).

---

### 1.6 `user_refresh_tokens` ✅ EXISTS (CRITICAL)

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | ✅ | |
| user_id | INT | FK -> users, IDX | ✅ | |
| refresh_token | TEXT | NOT NULL | ✅ | |
| is_revoked | BOOLEAN | DEFAULT false | ✅ | |
| revoked_at | TIMESTAMP | NULLABLE | ✅ | |
| ip_address | VARCHAR(50) | NULLABLE | ✅ | |
| device_key | VARCHAR(255) | NULLABLE | ✅ | |
| user_agent | TEXT | NULLABLE | ✅ | |
| created_at | TIMESTAMP | DEFAULT now() | ✅ | |
| expires_at | TIMESTAMP | NOT NULL | ✅ | |
| role_id | INT | FK -> roles, IDX | ✅ | Per-role isolation |
| client_app_type_id | INT | NULLABLE | ✅ | |
| token_hash | VARCHAR(512) | NOT NULL, UNQ, IDX | ➕ | NEW - Hashed token for fast lookup |

> **Critical Constraint (BR-20):** UNIQUE index on `(user_id, role_id) WHERE revoked_at IS NULL` to enforce one active refresh token per user-role pair.

---

### 1.7 `user_notification_settings` ✅ EXISTS

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id | SERIAL | PK | ✅ | |
| user_id | INT | FK -> users, IDX | ✅ | |
| notification_type_id | INT | FK -> notification_types | ✅ | |
| is_enabled | BOOLEAN | DEFAULT true | ✅ | |
| channel | VARCHAR(20) | DEFAULT 'push' | ➕ | NEW - push / sms / whatsapp / in_app |
| category | VARCHAR(50) | NULLABLE | ➕ | NEW - operational / marketing / safety |
| is_mandatory | BOOLEAN | DEFAULT false | ➕ | NEW - Cannot be disabled by user |
| quiet_hours_start | TIME | NULLABLE | ➕ | NEW - User-defined quiet start |
| quiet_hours_end | TIME | NULLABLE | ➕ | NEW - User-defined quiet end |
| created_at | TIMESTAMP | DEFAULT now() | ✅ | |
| updated_at | TIMESTAMP | NULLABLE | ✅ | |

---

### 1.8 `user_referral_profiles` 🆕 (V2 BASE)

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| user_id | INT | PK, FK -> users | 🆕 | One referral profile per user |
| referral_code | VARCHAR(20) | NOT NULL, UNQ, IDX | 🆕 | Stable referral code |
| referral_status | VARCHAR(20) | DEFAULT 'active' | 🆕 | active / disabled |
| generated_at | TIMESTAMP | DEFAULT now() | 🆕 | |
| disabled_at | TIMESTAMP | NULLABLE | 🆕 | |

---


## MODULE B2 - PRIMARY WORKFLOW: RIDE LIFECYCLE

### 4.1 `ride_types` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| ride_type_id | INT | ✅ | PK |
| ride_type_name | VARCHAR(100) | ✅ | Ride type display name |
| description | TEXT | ✅ | NULLABLE |
| is_active | BOOLEAN | ✅ | DEFAULT true |
| created_at | TIMESTAMP | ✅ | |
| updated_at | TIMESTAMP | ✅ | NULLABLE |

---

### 4.2 `ride_pricing` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| id (pricing_id) | SERIAL | PK | ✅ | |
| ride_type_id | INT | FK -> ride_types, IDX | ✅ | |
| vehicle_type_id | INT | FK -> vehicle_types, IDX | ✅ | |
| base_fare | DECIMAL(10,2) | NOT NULL | ✅ | |
| per_km_rate | DECIMAL(10,2) | NOT NULL | ✅ | |
| per_min_rate | DECIMAL(10,2) | NOT NULL | ✅ | |
| minimum_fare | DECIMAL(10,2) | | ✅ | |
| waiting_charge | DECIMAL(10,2) | | ✅ | |
| created_at | TIMESTAMP |  | ✅ |  |
| updated_at | TIMESTAMP |  | ✅ |  |
| surge_enabled | BOOLEAN | DEFAULT false | ➕ | |
| surge_multiplier | DECIMAL(4,2) | CHECK > 1.0 | ➕ | |
| surge_start_at | TIMESTAMP | NULLABLE | ➕ |  |
| surge_end_at | TIMESTAMP | NULLABLE | ➕ |  |
| free_cancellation_window_min | INT | DEFAULT 5 | ➕ | Minutes after booking |
| post_accept_cancel_fee | DECIMAL(10,2) | DEFAULT 0 | ➕ | |
| post_arrival_cancel_fee | DECIMAL(10,2) | DEFAULT 0 | ➕ | |
| effective_from | TIMESTAMP | DEFAULT now() | ➕ | |
| effective_to | TIMESTAMP | NULLABLE | ➕ | |
| is_published | BOOLEAN | DEFAULT false | ➕ | Only published rules apply |

---

### 4.3 `ride_peak_hours` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| peak_hour_id | INT | ✅ | PK |
| peak_day | VARCHAR(50) | ✅ | Day name/category |
| start_time | TIME | ✅ | Peak window start |
| end_time | TIME | ✅ | Peak window end |
| is_weekend | BOOLEAN | ✅ | NULLABLE |
| is_holiday | BOOLEAN | ✅ | NULLABLE |
| peak_multiplier | DECIMAL | ✅ | NULLABLE surge multiplier |
| is_active | BOOLEAN | ✅ | DEFAULT true |
| created_at | TIMESTAMP | ✅ | |
| updated_at | TIMESTAMP | ✅ | NULLABLE |

---

### 4.4 `ride_requests` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| ride_request_id | SERIAL | PK | ✅ | |
| user_id | INT | FK -> users, IDX | ✅ | Rider |
| pick_up_location | TEXT | NOT NULL | ✅ | |
| drop_off_location | TEXT | NULLABLE | ✅ | |
| pick_up_latitude | DECIMAL |  | ✅ | |
| pick_up_longitude | DECIMAL |  | ✅ | |
| drop_off_latitude | DECIMAL |  | ✅ | |
| drop_off_longitude | DECIMAL |  | ✅ | |
| ride_type_id | INT | FK -> ride_types | ✅ | |
| vehicle_type_id | INT | FK -> vehicle_types | ✅ | |
| rental_hours | INT | NULLABLE | ✅ | Package rides |
| outstation_return | BOOLEAN | NULLABLE | ✅ | |
| status | VARCHAR(50) | NOT NULL | ✅ | |
| estimated_fare | DECIMAL/INT | | ✅ | |
| estimated_distance | DECIMAL/INT | | ✅ | |
| estimated_duration | DECIMAL/INT | | ✅ | |
| pick_up_datetime | TIMESTAMP |  | ✅ |  |
| drop_off_datetime | TIMESTAMP |  | ✅ |  |
| is_active |  |  | ✅ |  |
| created_at |  |  | ✅ |  |
| updated_at |  |  | ✅ |  |
| booking_source | VARCHAR(30) | DEFAULT 'rider_app' | ➕ | rider_app / admin / phone / support |
| assigned_driver_id | INT | FK -> users, NULLABLE | ➕ | Denormalized for fast read |
| scheduled_pickup_at | TIMESTAMP | NULLABLE | ➕ | Scheduled/advance booking |
| promo_code | VARCHAR(30) | NULLABLE | ➕ | Applied promo |
| points_redeemed | INT | DEFAULT 0 | ➕ | Loyalty points used |
| collection_mode_preference | VARCHAR(30) | NULLABLE | ➕ | cash / upi / other |
| idempotency_key | UUID | UNQ, IDX | ➕ | Prevent duplicate bookings on retry |
| current_sequence_no | INT | DEFAULT 0 | ➕ | Real-time bus ordering |

---

### 4.5 `ride_trips` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| ride_trip_id | SERIAL | PK | ✅ | |
| ride_request_id | INT | FK -> ride_requests | ✅ | |
| driver_id | INT | FK -> users, IDX | ✅ | |
| vehicle_id | INT | FK -> vehicle_details | ✅ | |
| start_time | TIMESTAMP |  | ✅ |  |
| end_time | TIMESTAMP |  | ✅ |  |
| distance | DECIMAL |  | ✅ |  |
| fare | DECIMAL |  | ✅ |  |
| status | VARCHAR(50) | NOT NULL | ✅ | |
| start_otp | VARCHAR(10) | NULLABLE | ✅ |  |
| end_otp | VARCHAR(10) | NULLABLE | ✅ |  |
| is_active |  |  | ✅ |  |
| created_at |  |  | ✅ |  |
| updated_at |  |  | ✅ |  |
| trip_type | VARCHAR(20) | NOT NULL | ➕ | local / outstation / package |
| estimated_fare_snapshot | DECIMAL(10,2) | NULLABLE | ➕ | Fare at booking time |
| fare_breakdown_json | JSONB | NULLABLE | ➕ | {base, distance, surge, tax, total} |
| cancellation_fee | DECIMAL(10,2) | DEFAULT 0 | ➕ | |
| driver_commission_amount | DECIMAL(10,2) | NULLABLE | ➕ | |
| package_deduction_amount | DECIMAL(10,2) | DEFAULT 0 | ➕ | |
| rider_rating_submitted_at | TIMESTAMP | NULLABLE | ➕ | |
| driver_rating_submitted_at | TIMESTAMP | NULLABLE | ➕ | |
| force_end_reason | TEXT | NULLABLE | ➕ | Admin force-end reason |
| no_show_reason | TEXT | NULLABLE | ➕ | |
| user_id | INT | FK -> users | ➕ | Rider; denormalized from request |

---

### 4.6 `ride_bookings` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| booking_id | SERIAL | PK | |
| ride_request_id | INT | FK -> ride_requests, UNQ | One booking per request |
| assigned_driver_id | INT | FK -> users, IDX | |
| assignment_time | TIMESTAMP | | |
| estimated_fare | DECIMAL(10,2) | | At time of assignment |
| booking_status | VARCHAR(30) | NOT NULL | pending / confirmed / cancelled |
| created_at | TIMESTAMP | DEFAULT now() | |

> **Design Rule (v1.2):** Keep `ride_bookings` lightweight. Reliability, reconfirmation, no-show and SLA state live in `ride_booking_reliability`.

### 4.6A `ride_booking_reliability` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| booking_id | INT | PK, FK -> ride_bookings | 1:1 side table |
| pre_assignment_status | VARCHAR(20) | NULLABLE, IDX | pending / assigned / failed |
| reconfirmation_status | VARCHAR(20) | NULLABLE, IDX | pending / rider_done / driver_done / complete |
| rider_reconfirmed_at | TIMESTAMP | NULLABLE | |
| driver_reconfirmed_at | TIMESTAMP | NULLABLE | |
| primary_driver_id | INT | NULLABLE, IDX | |
| backup_driver_id | INT | NULLABLE, IDX | |
| no_show_actor_type | VARCHAR(20) | NULLABLE | rider / driver |
| no_show_recorded_at | TIMESTAMP | NULLABLE | |
| sla_due_at | TIMESTAMP | NULLABLE, IDX | |
| sla_breach_at | TIMESTAMP | NULLABLE | |
| sla_breach_reason | VARCHAR(100) | NULLABLE | |
| reliability_state | VARCHAR(20) | DEFAULT 'on_track', IDX | on_track / at_risk / breached / recovered |
| updated_at | TIMESTAMP | NULLABLE | |

---

### 4.7 `ride_history` EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| ride_trip_id | INT | ✅ |  |
| user_id | INT | ✅ |  |
| driver_id | INT | ✅ |  |
| vehicle_id | INT | ✅ |  |
| pickup_location | TEXT | ✅ |  |
| dropoff_location | TEXT | ✅ |  |
| ride_type | VARCHAR | ✅ |  |
| status | VARCHAR | ✅ |  |
| cancellation_status |  | ✅ |  |
| cancelled_by |  | ✅ |  |
| cancellation_reason |  | ✅ |  |
| timestamps |  | ➕ |  |
| total_fare |  | ✅ |  |
| total_distance |  | ✅ |  |
| ride_duration |  | ✅ |  |
| is_active |  | ✅ |  |
| created_at |  | ✅ |  |
| updated_at |  | ✅ |  |
| booking_source | VARCHAR(30) | ➕ | |
| cancellation_actor_type | VARCHAR(20) | ➕ | rider / driver / admin / system |
| dispute_flag | BOOLEAN | DEFAULT false | ➕ | |
| package_impact_json | JSONB | NULLABLE | ➕ | Package deduction summary |

---

### 4.8 `ride_cancellations` EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| cancellation_id | INT | ✅ |  |
| ride_trip_id | INT | ✅ |  |
| user_id | INT | ✅ |  |
| cancellation_reason | TEXT | ✅ | |
| cancellation_timestamp | TIMESTAMP | ✅ | |
| is_active |  | ✅ |  |
| created_at |  | ✅ |  |
| updated_at |  | ✅ |  |
| cancelled_by_role | VARCHAR(20) | ➕ | rider / driver / admin / system |
| cancellation_stage | VARCHAR(20) | ➕ | pre_accept / post_accept / post_arrival |
| cancellation_fee_charged | DECIMAL(10,2) | ➕ | |

---

### 4.9 `ride_payments` EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| payment_id | INT | ✅ |  |
| ride_trip_id | INT | ✅ |  |
| user_id | INT | ✅ |  |
| driver_id | INT | ✅ |  |
| vehicle_id | INT | ➕ |  |
| payment_method |  | ✅ |  |
| payment_status |  | ✅ |  |
| payment_amount |  | ✅ |  |
| payment_timestamp |  | ✅ |  |
| transaction_id |  | ✅ |  |
| payment_gateway |  | ✅ |  |
| payment_details |  | ✅ |  |
| is_active |  | ✅ |  |
| created_at |  | ✅ |  |
| updated_at |  | ✅ |  |
| payer_type | VARCHAR(20) | ➕ | rider / driver / platform |
| purpose | VARCHAR(30) | ➕ | ride_fare / package_purchase / refund |
| promo_discount_amount | DECIMAL(10,2) | ➕ | |
| tax_amount | DECIMAL(10,2) | ➕ | |
| net_amount | DECIMAL(10,2) | ➕ | After discounts and tax |
| settlement_reference | VARCHAR(100) | ➕ | External settlement ID |

---

### 4.10 `trip_events` 🆕 (Reliability / Outbox Pattern)

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

### 4.11 `outbox_events` 🆕 (Reliability)

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

### 4.12 `outbox_delivery_attempts` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| attempt_id | SERIAL | PK | |
| outbox_id | INT | FK -> outbox_events, IDX | |
| provider | VARCHAR(50) | NOT NULL | azure_service_bus / redis |
| status | VARCHAR(20) | | |
| response_detail | TEXT | NULLABLE | |
| attempted_at | TIMESTAMP | DEFAULT now() | |

---


## MODULE C1 - DRIVER APP HIGH-END: DRIVER MANAGEMENT & ONBOARDING

### 3.1 `driver_details` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| driver_id | SERIAL | PK | ✅ | |
| user_id | INT | FK -> users, UNQ | ✅ | |
| known_language | TEXT | NULLABLE | ✅ | Languages for rider communication (driver-role only) |
| status | VARCHAR(50) | NULLABLE | ✅ | |
| approval_status_id | INT | Enum FK | ✅ | |
| vehicle_type_id | INT | FK -> vehicle_types | ✅ | |
| driver_type_id | INT | Enum | ✅ | |
| is_active | BOOLEAN | DEFAULT true | ✅ | |
| review_notes | TEXT | NULLABLE | ✅ | |
| created_at | TIMESTAMP |  | ✅ |  |
| updated_at | TIMESTAMP |  | ✅ |  |
| primary_zone_id | INT | NULLABLE | ➕ | Primary operating zone/city |
| ride_category_flags | JSONB | NULLABLE | ➕ | {local:true, outstation:false, package:true} |
| package_required_for_dispatch | BOOLEAN | DEFAULT false | ➕ | Dispatch blocked if no active package |
| restricted_reason | TEXT | NULLABLE | ➕ | Policy restriction reason |
| restricted_until | TIMESTAMP | NULLABLE | ➕ | Timed restriction |

---

### 3.2 `driver_status` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| driver_status_id | SERIAL | PK | ✅ | |
| user_id | INT | FK -> users, UNQ, IDX | ✅ | One status row per driver |
| vehicle_id | INT | FK -> vehicle_details, NULLABLE | ✅ | |
| status | VARCHAR(20) | NOT NULL | ✅ | online / offline / on_trip |
| created_at | TIMESTAMP |  | ✅ |  |
| updated_at | TIMESTAMP |  | ✅ |  |
| lock_reason | VARCHAR(50) | NULLABLE | ➕ | active_trip / policy_violation / no_package |
| current_trip_id | INT | FK -> ride_trips, NULLABLE | ➕ | |
| last_status_changed_at | TIMESTAMP | NULLABLE | ➕ | |
| status_source | VARCHAR(20) | NULLABLE | ➕ | driver_action / system / admin |

---

### 3.3 `driver_shift_log` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| shift_id | SERIAL | PK | ✅ | |
| user_id | INT | FK -> users, IDX | ✅ | |
| vehicle_id | INT | FK -> vehicle_details | ✅ | |
| check_in_time | TIMESTAMP | NOT NULL | ✅ | |
| check_out_time | TIMESTAMP | NULLABLE | ✅ | |
| check_in_latitude | DECIMAL | NULLABLE | ✅ | |
| check_in_longitude | DECIMAL | NULLABLE | ✅ | |
| check_out_latitude | DECIMAL | NULLABLE | ✅ | |
| check_out_longitude | DECIMAL | NULLABLE | ✅ | |
| shift_status | INT | Enum | ✅ | |
| is_active | BOOLEAN | DEFAULT true | ✅ | |
| created_at | TIMESTAMP |  | ✅ |  |
| updated_at | TIMESTAMP |  | ✅ |  |
| first_online_time | TIMESTAMP | NULLABLE | ➕ | First time driver went online in shift |
| final_offline_time | TIMESTAMP | NULLABLE | ➕ | Last time driver went offline |
| total_online_duration_min | INT | DEFAULT 0 | ➕ | Running total in minutes |
| total_trips_completed | INT | DEFAULT 0 | ➕ | |
| total_earnings | DECIMAL(10,2) | DEFAULT 0 | ➕ | |
| total_cancellations | INT | DEFAULT 0 | ➕ | |
| total_no_shows | INT | DEFAULT 0 | ➕ | |

---

### 3.4 `vehicle_details` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| vehicle_id | INT | ✅ | PK |
| vehicle_type_id | INT/ENUM | ✅ | Vehicle type reference |
| user_id | INT | ✅ | FK -> users, NULLABLE |
| vehicle_registration_number | VARCHAR(50) | ✅ | REQUIRED |
| vehicle_make | VARCHAR(50) | ✅ | NULLABLE |
| vehicle_model | VARCHAR(50) | ✅ | NULLABLE |
| vehicle_year | INT | ✅ | NULLABLE |
| seating_capacity | INT | ✅ | NULLABLE |
| fuel_type | VARCHAR(20) | ✅ | NULLABLE |
| color | VARCHAR(30) | ✅ | NULLABLE |
| registration_date | TIMESTAMP | ✅ | NULLABLE |
| insurance_number | VARCHAR(50) | ✅ | NULLABLE |
| insurance_expiry_date | TIMESTAMP | ✅ | NULLABLE |
| status | INT | ✅ | NULLABLE |
| approval_date | TIMESTAMP | ✅ | NULLABLE |
| rejection_reason | TEXT | ✅ | NULLABLE |
| is_active | BOOLEAN | ✅ | NULLABLE |
| created_at | TIMESTAMP | ✅ | |
| updated_at | TIMESTAMP | ✅ | NULLABLE |
| approval_status_id | INT/ENUM | ✅ | Approval status reference |

---

### 3.5 `vehicle_locations` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| vehicle_location_id | INT | ✅ | PK |
| vehicle_id | INT | ✅ | FK -> vehicle_details |
| user_id | INT | ✅ | FK -> users |
| latitude | DOUBLE/DECIMAL | ✅ | Live GPS |
| longitude | DOUBLE/DECIMAL | ✅ | Live GPS |
| timestamp | TIMESTAMP | ✅ | Location timestamp |
| status | INT/ENUM | ✅ | Driver status snapshot |
| is_active | BOOLEAN | ✅ | DEFAULT true |
| created_at | TIMESTAMP | ✅ | |
| updated_at | TIMESTAMP | ✅ | NULLABLE |

> **Note:** Redis mirrors this for real-time; DB is fallback/history only.

---

### 3.6 `vehicle_types`, `vehicle_models` NO CHANGE

#### `vehicle_types` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| vehicle_type_id | INT | ✅ | PK |
| vehicle_type_name | VARCHAR(100) | ✅ | Vehicle type name |
| description | TEXT | ✅ | NULLABLE |
| is_active | BOOLEAN | ✅ | DEFAULT true |
| created_at | TIMESTAMP | ✅ | |
| updated_at | TIMESTAMP | ✅ | NULLABLE |

#### `vehicle_models` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| model_id | INT | ✅ | PK |
| model_name | VARCHAR(100) | ✅ | REQUIRED |
| brand_name | VARCHAR(100) | ✅ | REQUIRED |
| vehicle_type_id | INT/ENUM | ✅ | Vehicle type reference |
| created_at | TIMESTAMP | ✅ | |

---

### 3.7 `driver_vehicle_link` EXTEND

| Column | Notes |
|--------|-------|
| link_id | FK |
| driver_id (user_id) | FK |
| vehicle_id | FK |
| approval_status_id |  |
| start_date |  |
| end_date |  |
| is_active |  |
| created_at |  |
| updated_at |  |
| link_type | Added in BRD delta |
| link_status | Added in BRD delta |
| effective_from | Added in BRD delta |
| effective_to | Added in BRD delta |
| approved_by | Added in BRD delta |
| approved_at | Added in BRD delta |
| change_reason | Added in BRD delta |

---

### 3.8 `driver_vehicle_link_history` EXTEND

| Column | Notes |
|--------|-------|
| Existing history columns | Keep as-is |
| event_type | Added in BRD delta |
| actor_user_id | Added in BRD delta |
| role | Added in BRD delta |
| old_vehicle_id | Added in BRD delta |
| new_vehicle_id | Added in BRD delta |
| old_link_status | Added in BRD delta |
| new_link_status | Added in BRD delta |
| event_reason | Added in BRD delta |
| event_at | Added in BRD delta |

---


## MODULE C2 - DRIVER APP HIGH-END: DRIVER PACKAGES & MONETIZATION

### 5.1 `driver_packages` 🆕

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
| created_at | TIMESTAMP |  |  |
| updated_at | TIMESTAMP |  |  |

---

### 5.2 `driver_package_purchases` 🆕

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

### 5.3 `driver_package_transactions` 🆕

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

### 5.4 `package_refund_claims` 🆕

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

### 5.5 `package_refund_evidence` 🆕

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


## MODULE D - RIDER APP HIGH-END: RIDER ENGAGEMENT (Plugin: Conditional)

### 6.1 `saved_addresses` 🆕

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

### 6.2 `referral_events` 🆕 (Plugin: referral_enabled)

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

### 6.3 `loyalty_ledger` 🆕 (Plugin: loyalty_enabled)

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


## MODULE E1 - ADMIN HIGH-END: SUPPORT & ESCALATION

### 7.1 `tickets` EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| ticket_id |  | ✅ |  |
| user_id |  | ✅ |  |
| subject |  | ✅ |  |
| description |  | ✅ |  |
| status |  | ✅ |  |
| priority |  | ✅ |  |
| assigned_to_user_id |  | ✅ |  |
| created_at | TIMESTAMP | ✅ |  |
| updated_at | TIMESTAMP | ✅ |  |
| user_type | VARCHAR(20) | ➕ | rider / driver |
| query_type_id | INT | ➕ | FK -> enum_value_catalog (ticket category) |
| linked_ride_trip_id | INT | ➕ | FK -> ride_trips, NULLABLE |
| linked_package_purchase_id | INT | ➕ | FK -> driver_package_purchases, NULLABLE |
| sla_due_at | TIMESTAMP | ➕ | Response SLA deadline |
| first_response_at | TIMESTAMP | ➕ | |
| resolved_at | TIMESTAMP | ➕ | |
| escalation_level | INT | DEFAULT 0 | ➕ | 0=none, 1=L1, 2=L2, 3=management |
| escalated_to_user_id | INT | ➕ | FK -> users, NULLABLE |
| resolution_code | VARCHAR(50) | ➕ | |
| resolution_note_public | TEXT | ➕ | Visible to rider/driver |

---

### 7.2 `ticket_comments` EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| comment_id |  | ✅ |  |
| ticket_id |  | ✅ |  |
| user_id |  | ✅ |  |
| comment_text |  | ✅ |  |
| created_at |  | ✅ |  |
| is_internal_note | BOOLEAN | ➕ | Hidden from rider/driver |
| attachment_url | TEXT | ➕ | NULLABLE |
| actor_role | VARCHAR(20) | ➕ | rider / driver / support / admin |

---

### 7.3 `ticket_actions` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| id | INT | ✅ | PK |
| ticket_id | INT | ✅ | FK -> tickets |
| ticket_type | INT | ✅ | Existing ticket type/category code |
| action_by | INT | ✅ | User ID who performed the action |
| action_type | VARCHAR(50) | ✅ | Action name/type |
| action_details | TEXT | ✅ | Existing action detail payload |
| created_at | TIMESTAMP | ✅ | Action creation timestamp |

---

### 7.4 `sos_alerts` 🆕

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

### 7.5 `policy_violation_alerts` 🆕

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


## MODULE E2 - ADMIN HIGH-END: GOVERNANCE & APPROVALS

### 8.1 `approval_history` EXTEND

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| id |  | ✅ |  |
| entity_id |  | ✅ |  |
| entity_type |  | ✅ |  |
| previous_status_id |  | ✅ |  |
| new_status_id |  | ✅ |  |
| changed_by |  | ✅ |  |
| changed_at |  | ✅ |  |
| review_notes | | ✅ | |
| workflow_type | VARCHAR(50) | ➕ | driver_onboarding / document / high_impact |
| workflow_step | INT | DEFAULT 1 | ➕ | Step # in multi-stage workflow |
| decision | VARCHAR(20) | ➕ | approved / rejected / verified |
| old_value_json | JSONB | ➕ | Pre-change snapshot |
| new_value_json | JSONB | ➕ | Post-change snapshot |
| change_request_id | INT | ➕ | FK -> high_impact_change_requests, NULLABLE |

---

### 8.2 `high_impact_change_requests` 🆕

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

### 8.3 `high_impact_change_approvals` 🆕

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


## MODULE E3 - ADMIN HIGH-END: QUOTE DESK

### 10.1 `quote_requests` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| quote_id | SERIAL | PK | |
| user_id | INT | FK -> users, NULLABLE | If raised by logged-in user |
| quote_type | VARCHAR(20) | NOT NULL | outstation / package |
| pickup_location | TEXT | NOT NULL | |
| drop_location | TEXT | NULLABLE | |
| pickup_lat | DECIMAL |  |  |
| pickup_lng | DECIMAL |  |  |
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

### 10.2 `quote_booking_links` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| id | SERIAL | PK | |
| quote_id | INT | FK -> quote_requests | |
| ride_request_id | INT | FK -> ride_requests | |
| linked_at | TIMESTAMP | DEFAULT now() | |

---


## MODULE E4 - ADMIN HIGH-END: REPORTING

### 11.1 `report_templates` 🆕

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

### 11.2 `report_schedules` 🆕

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

### 11.3 `report_runs` 🆕

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


## MODULE F1 - REMAINING PLATFORM: AUDIT LOGGING

### 9.1 `immutable_audit_logs` 🆕

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


## MODULE F2 - REMAINING PLATFORM: NOTIFICATIONS

### 12.1 `notification_types` NO CHANGE

| Column | Type | Status | Notes |
|--------|------|--------|-------|
| id | INT | ✅ | PK |
| name | VARCHAR | ✅ | Notification type name |
| description | TEXT | ✅ | NULLABLE |
| is_active | BOOLEAN | ✅ | DEFAULT true |
| created_at | TIMESTAMP | ✅ | |
| updated_at | TIMESTAMP | ✅ | NULLABLE |

### 12.2 `notification_inbox` 🆕

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

## SUMMARY COUNTS

Counts are intentionally not hard-coded in v1.2 because BRD delta sections are merged into the same document and are the source of truth for implementation.

Use this rule for execution planning:
- Treat base module sections + D1..D5 delta sections as the final schema scope.
- For identity and booking, keep core tables lightweight and place operational/audit detail in side tables.

---

## MIGRATION PHASE ASSIGNMENT

## BRD-Only Delta Update (Latest Finalized BRDs)

This delta is derived from source files whose names contain `BRD` (01 to 08) and should be merged into implementation migrations.

---

### D1. Driver-Vehicle Link Governance

#### `driver_vehicle_link` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| link_type | VARCHAR(30) | NOT NULL, IDX | ➕ | primary / acting / vendor_assigned |
| link_status | VARCHAR(20) | NOT NULL, IDX | ➕ | pending / active / inactive / rejected |
| effective_from | TIMESTAMP | NULLABLE | ➕ | |
| effective_to | TIMESTAMP | NULLABLE | ➕ | |
| approved_by | INT | NULLABLE | ➕ | Admin actor ID |
| approved_at | TIMESTAMP | NULLABLE | ➕ | |
| change_reason | TEXT | NULLABLE | ➕ | |

#### `driver_vehicle_link_history` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| event_type | VARCHAR(40) | NOT NULL | ➕ | link_created / switch_requested / switch_approved / switch_rejected / unlinked |
| actor_user_id | INT | NULLABLE, IDX | ➕ | |
| actor_role | VARCHAR(20) | NULLABLE | ➕ | driver / admin / system |
| old_vehicle_id | INT | NULLABLE | ➕ | |
| new_vehicle_id | INT | NULLABLE | ➕ | |
| old_link_status | VARCHAR(20) | NULLABLE | ➕ | |
| new_link_status | VARCHAR(20) | NULLABLE | ➕ | |
| event_reason | TEXT | NULLABLE | ➕ | |
| event_at | TIMESTAMP | DEFAULT now(), IDX | ➕ | |

#### `ride_trips` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| assigned_driver_snapshot_id | INT | NULLABLE | ➕ | Immutable assignment snapshot |
| assigned_vehicle_snapshot_id | INT | NULLABLE | ➕ | Immutable assignment snapshot |

---

### D2. Scheduled Ride Reliability (Package/Outstation)

#### `ride_bookings` EXTEND (lightweight only)

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| assigned_driver_id | INT | NULLABLE, IDX | KEEP | Core booking linkage |
| assignment_time | TIMESTAMP | NULLABLE | KEEP | |
| estimated_fare | DECIMAL(10,2) | NULLABLE | KEEP | |
| booking_status | VARCHAR(30) | NOT NULL, IDX | KEEP | pending / confirmed / cancelled |

#### `ride_booking_reliability` 🆕

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| booking_id | INT | PK, FK -> ride_bookings | 🆕 | 1:1 reliability side table |
| pre_assignment_status | VARCHAR(20) | NULLABLE, IDX | 🆕 | pending / assigned / failed |
| reconfirmation_status | VARCHAR(20) | NULLABLE, IDX | 🆕 | pending / rider_done / driver_done / complete |
| rider_reconfirmed_at | TIMESTAMP | NULLABLE | 🆕 | |
| driver_reconfirmed_at | TIMESTAMP | NULLABLE | 🆕 | |
| primary_driver_id | INT | NULLABLE, IDX | 🆕 | |
| backup_driver_id | INT | NULLABLE, IDX | 🆕 | |
| no_show_actor_type | VARCHAR(20) | NULLABLE | 🆕 | rider / driver |
| no_show_recorded_at | TIMESTAMP | NULLABLE | 🆕 | |
| sla_due_at | TIMESTAMP | NULLABLE, IDX | 🆕 | |
| sla_breach_at | TIMESTAMP | NULLABLE | 🆕 | |
| sla_breach_reason | VARCHAR(100) | NULLABLE | 🆕 | |
| reliability_state | VARCHAR(20) | DEFAULT 'on_track', IDX | 🆕 | on_track / at_risk / breached / recovered |

#### `ride_requests` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| rematch_attempt_count | INT | DEFAULT 0 | ➕ | |
| dispatch_timeout_at | TIMESTAMP | NULLABLE, IDX | ➕ | |
| no_driver_found_at | TIMESTAMP | NULLABLE | ➕ | |
| fallback_action_taken | VARCHAR(50) | NULLABLE | ➕ | retry / queue / fail_safe_exit |

---

### D3. Serviceability & Geo-Governance

#### `ride_requests` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| pickup_zone_code | VARCHAR(50) | NULLABLE, IDX | ➕ | |
| drop_zone_code | VARCHAR(50) | NULLABLE, IDX | ➕ | |
| serviceability_status | VARCHAR(20) | NULLABLE, IDX | ➕ | serviceable / out_of_service / restricted |
| serviceability_reason_code | VARCHAR(50) | NULLABLE | ➕ | |
| location_tamper_flag | BOOLEAN | DEFAULT false | ➕ | |

#### `service_zones` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| zone_id | SERIAL | PK | |
| zone_code | VARCHAR(50) | NOT NULL, UNQ | |
| zone_name | VARCHAR(100) | NOT NULL | |
| city_code | VARCHAR(50) | NOT NULL, IDX | |
| status | VARCHAR(20) | DEFAULT 'active' | |
| effective_from | TIMESTAMP | DEFAULT now() | |
| effective_to | TIMESTAMP | NULLABLE | |

#### `geo_fences` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| geofence_id | SERIAL | PK | |
| zone_id | INT | NULLABLE, IDX | Logical zone reference |
| geofence_type | VARCHAR(30) | NOT NULL, IDX | serviceable / restricted / no_pickup / no_drop |
| polygon_geojson | JSONB | NOT NULL | |
| is_active | BOOLEAN | DEFAULT true | |
| created_at | TIMESTAMP | DEFAULT now() | |

#### `serviceability_decisions` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| decision_id | BIGSERIAL | PK | |
| ride_request_id | INT | NOT NULL, IDX | |
| pickup_result | VARCHAR(20) | NOT NULL | serviceable / out_of_service / restricted |
| drop_result | VARCHAR(20) | NOT NULL | serviceable / out_of_service / restricted |
| reason_code | VARCHAR(50) | NULLABLE | |
| evaluated_at | TIMESTAMP | DEFAULT now(), IDX | |
| evaluated_by_source | VARCHAR(30) | NOT NULL | booking_api / dispatch_service |

#### `location_tamper_events` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| event_id | BIGSERIAL | PK | |
| user_id | INT | NOT NULL, IDX | |
| role | VARCHAR(20) | NOT NULL | rider / driver |
| event_type | VARCHAR(40) | NOT NULL | mock_location / gps_jump / impossible_speed |
| confidence_score | DECIMAL(5,2) | NULLABLE | |
| observed_at | TIMESTAMP | DEFAULT now(), IDX | |
| action_taken | VARCHAR(40) | NULLABLE | flag / block / escalate |

---

### D4. Driver Settlement & Payout Operations

#### `driver_wallet_ledger` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| ledger_id | BIGSERIAL | PK | |
| driver_id | INT | NOT NULL, IDX | |
| entry_type | VARCHAR(30) | NOT NULL, IDX | commission / package / adjustment / reversal / payout |
| amount | DECIMAL(12,2) | NOT NULL | |
| balance_after | DECIMAL(12,2) | NOT NULL | |
| reference_type | VARCHAR(40) | NULLABLE | ride_trip / package_purchase / settlement |
| reference_id | VARCHAR(50) | NULLABLE, IDX | |
| created_at | TIMESTAMP | DEFAULT now(), IDX | |

#### `driver_settlement_cycles` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| cycle_id | SERIAL | PK | |
| cycle_start_at | TIMESTAMP | NOT NULL | |
| cycle_end_at | TIMESTAMP | NOT NULL, IDX | |
| status | VARCHAR(20) | NOT NULL, IDX | draft / locked / posted / closed |
| total_drivers | INT | DEFAULT 0 | |
| total_payable_amount | DECIMAL(14,2) | DEFAULT 0 | |
| generated_at | TIMESTAMP | DEFAULT now() | |
| closed_at | TIMESTAMP | NULLABLE | |

#### `driver_settlement_items` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| item_id | BIGSERIAL | PK | |
| cycle_id | INT | NOT NULL, IDX | |
| driver_id | INT | NOT NULL, IDX | |
| gross_earning | DECIMAL(12,2) | DEFAULT 0 | |
| commission_due | DECIMAL(12,2) | DEFAULT 0 | |
| package_due | DECIMAL(12,2) | DEFAULT 0 | |
| adjustments | DECIMAL(12,2) | DEFAULT 0 | |
| net_payable | DECIMAL(12,2) | DEFAULT 0 | |
| settlement_status | VARCHAR(20) | NOT NULL | pending / approved / paid / held |

#### `driver_payouts` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| payout_id | BIGSERIAL | PK | |
| driver_id | INT | NOT NULL, IDX | |
| settlement_item_id | BIGINT | NOT NULL, IDX | |
| payout_status | VARCHAR(20) | NOT NULL, IDX | initiated / processing / paid / failed |
| payout_ref | VARCHAR(100) | NULLABLE, IDX | |
| initiated_at | TIMESTAMP | DEFAULT now() | |
| completed_at | TIMESTAMP | NULLABLE | |
| failed_reason | TEXT | NULLABLE | |

#### `driver_payout_failures` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| failure_id | BIGSERIAL | PK | |
| payout_id | BIGINT | NOT NULL, IDX | |
| failure_code | VARCHAR(50) | NOT NULL | |
| failure_message | TEXT | NULLABLE | |
| retry_count | INT | DEFAULT 0 | |
| next_retry_at | TIMESTAMP | NULLABLE, IDX | |
| resolved_at | TIMESTAMP | NULLABLE | |

---

### D5. Support Lifecycle Extensions

#### `tickets` EXTEND

| Column | Type | Constraint | Status | Notes |
|--------|------|------------|--------|-------|
| waiting_on_type | VARCHAR(20) | NULLABLE | ➕ | rider / driver / internal_team / system |
| reopened_count | INT | DEFAULT 0 | ➕ | |
| parent_ticket_id | INT | NULLABLE, IDX | ➕ | Parent-child linkage |
| duplicate_of_ticket_id | INT | NULLABLE, IDX | ➕ | Duplicate merge mapping |
| breach_risk_flag | BOOLEAN | DEFAULT false | ➕ | |

#### `ticket_status_history` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| history_id | BIGSERIAL | PK | |
| ticket_id | INT | NOT NULL, IDX | |
| from_status | VARCHAR(30) | NULLABLE | |
| to_status | VARCHAR(30) | NOT NULL | |
| changed_by | INT | NULLABLE | |
| changed_at | TIMESTAMP | DEFAULT now(), IDX | |
| reason_code | VARCHAR(50) | NULLABLE | |

#### `ticket_assignments` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| assignment_id | BIGSERIAL | PK | |
| ticket_id | INT | NOT NULL, IDX | |
| assigned_team | VARCHAR(50) | NOT NULL | |
| assigned_user_id | INT | NULLABLE, IDX | |
| assigned_at | TIMESTAMP | DEFAULT now(), IDX | |
| unassigned_at | TIMESTAMP | NULLABLE | |
| assignment_reason | VARCHAR(100) | NULLABLE | |

#### `ticket_sla_events` 🆕

| Column | Type | Constraint | Notes |
|--------|------|------------|-------|
| sla_event_id | BIGSERIAL | PK | |
| ticket_id | INT | NOT NULL, IDX | |
| event_type | VARCHAR(20) | NOT NULL | warning / breach / pause / resume |
| event_at | TIMESTAMP | DEFAULT now(), IDX | |
| metadata_json | JSONB | NULLABLE | |

| Phase | Tables Included |
|-------|----------------|
| **M1 - Foundation** | deployment_profiles, deployment_branding, deployment_feature_flags, deployment_business_rules, configuration_keys_catalog, enum_value_catalog, deployment_configuration_values, configuration_change_history + user_refresh_tokens token_hash + users auth columns |
| **M2 - Ride Lifecycle** | ride_bookings, ride_booking_reliability, trip_events, outbox_events, outbox_delivery_attempts + extend ride_requests, ride_trips, ride_payments, ride_history, ride_cancellations + extend ride_pricing |
| **M3 - Driver Packages** | driver_packages, driver_package_purchases, driver_package_transactions, package_refund_claims, package_refund_evidence + extend driver_details, driver_status, driver_shift_log |
| **M4 - Support & Governance** | sos_alerts, policy_violation_alerts, high_impact_change_requests, high_impact_change_approvals, immutable_audit_logs + extend tickets, ticket_comments, approval_history |
| **M5 - Growth Modules** | saved_addresses, referral_events, loyalty_ledger, quote_requests, quote_booking_links, report_templates, report_schedules, report_runs, notification_inbox + extend user_details, user_notification_settings, documents, coupons |




