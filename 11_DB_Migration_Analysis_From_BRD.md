# DB Migration Analysis from BRD/TSD (No Code Change)

## 1. Scope of this analysis

This document maps new BRD/TSD requirements to backend database changes for:
- Core API service in ZCarsAPI
- Existing PostgreSQL entities in ApplicationDbContext
- Existing API surface in Controllers and DriverAppController

This is analysis only. No code/entity/migration generation is done in this phase.

V2 policy for this document:
- This is a new, not-live schema baseline.
- No backward compatibility constraints apply.
- Legacy fields/tables can be removed where replaced by cleaner V2 design.

## 2. Inputs reviewed

- 01_BRD_Rekla_Ride_App.md
- 02_BRD_Rider_App.md
- 03_BRD_Driver_App.md
- 04_BRD_Admin_Portal.md
- 05_BRD_Driver_Pricing_Model.md
- 06_BRD_Referral_Reward_Coupon_Management.md
- 07_BRD_Rating_Feedback_System.md
- 08_BRD_Unified_Support_Ticketing_System.md
- Existing EF entities + ApplicationDbContext in ZCarsAPI
- Existing HTTP endpoint inventory in Controllers and DriverAppController

Input filtering rule:
- Source-of-truth business inputs in this analysis are restricted to files whose names contain `BRD`.
- TSD and checklist documents may provide context but do not override BRD requirements.

## 3. Current backend data model snapshot (high-level)

Current database already has major operational tables:
- Users and profile: users, user_details, roles, user_roles, user_refresh_tokens, user_notification_settings
- Driver and vehicle: driver_details, driver_status, driver_shift_log, vehicle_details, vehicle_locations, vehicle_types, vehicle_models, driver_vehicle_link, driver_vehicle_link_history
- Trip lifecycle: ride_requests, ride_trips, ride_history, ride_cancellations, ride_payments, ride_types, ride_pricing, ride_peak_hours
- Support and approvals: tickets, ticket_comments, approval_history, service_requests, ticket_actions
- Config/meta: coupons, notification_types, documents, document_types, feedback, banner, language

Key gap: BRD/TSD requires deployment-specific configuration, central enum/config catalogs, multi-role support (same mobile number as rider and driver), per-user-per-role refresh token isolation, package subscriptions, referral/loyalty plugins, advanced support/escalation, maker-verifier-approver workflow, immutable audit/event logs, quote desk, report builder, and reliability/outbox persistence.

Configuration policy update: Configuration is runtime value-only and independent. Configuration tables/records must not be implemented with FK dependencies to core transactional/domain tables.

## 4. Table and column change list

## 4.1 Existing tables: required column updates

1. users
- No additional business/audit columns on `users`; keep this table minimal as the hot shared identity table.
- Keep role-specific restriction data out of `users` (store in role-owned tables such as `driver_details`).
- Move login telemetry and referral profile into dedicated tables (`user_login_history`, `user_referral_profiles`).
- Drop legacy `users.role_id` (single-role model); use `user_roles` only.

2. user_details
- Add default_saved_address_id
- Add language_preference (normalize from free-text language)
- Add legal_acceptance_version, legal_accepted_at

2A. user_refresh_tokens (CRITICAL: NEW SCHEMA REQUIREMENT)
- Ensure schema includes: user_id, role, refresh_token, token_hash, issued_at, expires_at, device_id, ip_address, revoked_at
- **Mandatory constraint (BR-20):** Per user-role combination, enforce single active refresh token; issuing a new token MUST invalidate the previous token for that specific user-role pair.
- **Reason:** Supports multi-role capability (BR-19) where same mobile number can be both Rider and Driver; requires strict session isolation per role to prevent cross-role session leakage (NFR-15).
- unique constraint on (user_id, role) WHERE revoked_at IS NULL (ensures only one active token per user-role)
- Add index on (token_hash) for fast lookup during token validation
- Add index on (user_id, role, expires_at) for session cleanup

3. driver_details
- Add primary_driving_city_id / zone_id
- Add ride_category_flags (local/outstation/package eligibility)
- Add package_required_for_dispatch (bool)
- Add restricted_reason, restricted_until (driver-role restriction state belongs here, not in users)

4. documents
- Add expiry_date
- Add renewal_required_from
- Add rejection_code
- Add source_channel (app/admin)

5. coupons
- Add applicable_ride_types (local/outstation/package)
- Add per_user_limit
- Add total_redemption_limit
- Add current_redemption_count
- Add default_for_new_driver (bool)
- Add target_audience (rider/driver/both)

6. ride_pricing
- Add surge_enabled, surge_multiplier, surge_start_at, surge_end_at
- Add free_cancellation_window_min
- Add post_accept_cancel_fee, post_arrival_cancel_fee
- Add fallback_no_package_policy
- Add effective_from, effective_to, is_published

7. ride_requests
- Add booking_source (rider_app/admin/phone/support)
- Add assigned_driver_id (if not maintained elsewhere)
- Add scheduled_pickup_at (explicit, nullable)
- Add promo_code
- Add points_redeemed
- Add collection_mode_preference (cash/direct_upi/...)
- Add cancellation_policy_snapshot_json
- Add idempotency_key (for reliability)
- Add current_sequence_no (real-time ordering)

8. ride_trips
- Add trip_type (local/outstation/package)
- Add estimated_fare_snapshot
- Add fare_breakdown_json
- Add cancellation_fee
- Add driver_commission_amount
- Add package_deduction_amount
- Add rider_rating_submitted_at, driver_rating_submitted_at
- Add force_end_reason, no_show_reason

9. ride_history
- Add booking_source
- Add cancellation_actor_type
- Add dispute_flag
- Add package_impact_json

10. ride_payments
- Add payer_type (rider/driver/platform)
- Add purpose (ride_fare/package_purchase/refund_adjustment)
- Add promo_discount_amount
- Add tax_amount
- Add net_amount
- Add settlement_reference

11. driver_shift_log
- Add first_online_time
- Add final_offline_time
- Add total_online_duration_min
- Add total_trips_completed
- Add total_earnings
- Add total_cancellations, total_no_shows

12. driver_status
- Add lock_reason (active_trip/policy_violation/no_package)
- Add current_trip_id
- Add last_status_changed_at
- Add status_source (driver_action/system/admin)

13. tickets
- Add user_type (rider/driver)
- Add query_type_id (mandatory BRD classification)
- Add linked_ride_trip_id
- Add linked_package_purchase_id
- Add sla_due_at, first_response_at, resolved_at
- Add escalation_level, escalated_to_user_id
- Add resolution_code, resolution_note_public

14. ticket_comments
- Add is_internal_note (hidden from user)
- Add attachment_url
- Add actor_role

15. approval_history
- Extend for multi-stage workflows:
- Add workflow_type, workflow_step
- Add decision (approved/rejected/verified)
- Add old_value_json, new_value_json
- Add change_request_id

16. user_notification_settings
- Add channel (push/sms/whatsapp/in_app)
- Add category (operational/marketing/safety)
- Add mandatory_flag
- Add quiet_hours_start, quiet_hours_end

## 4.2 New tables required (high-priority)

1. deployment_profiles
- deployment_id, deployment_code, deployment_name, status, created_at

2. deployment_branding
- deployment_branding_id, deployment_id, app_name, logos, theme tokens, legal links, support contacts

3. deployment_feature_flags
- deployment_id, referral_enabled, loyalty_enabled, promo_enabled, package_enabled, quote_desk_enabled, report_builder_enabled

4. deployment_business_rules
- deployment_id, rule_key, rule_value_json, version, effective_from, effective_to

4A. configuration_keys_catalog
- key_id, module, config_key, value_type (string/int/bool/decimal/json/enum), default_value, description, is_enum_catalog, is_editable, created_at

4B. enum_value_catalog
- enum_value_id, key_id, enum_code, enum_label, sort_order, is_active, metadata_json

4C. deployment_configuration_values
- value_id, deployment_id, key_id, config_value, effective_from, effective_to, version, changed_by, changed_at

5. saved_addresses
- address_id, user_id, label, full_address, lat, lng, landmark, last_used_at, is_default

5A. user_login_history
- id, user_id, ip_address, device_id, logged_in_at

5B. user_referral_profiles
- user_id, referral_code, referral_status, generated_at, disabled_at

6. ride_bookings
- booking_id, ride_request_id, assigned_driver_id, assignment_time, estimated_fare, booking_status, created_at

6A. ride_booking_reliability
- booking_id, pre_assignment_status, reconfirmation_status, rider_reconfirmed_at, driver_reconfirmed_at, primary_driver_id, backup_driver_id, no_show_actor_type, no_show_recorded_at, sla_due_at, sla_breach_at, sla_breach_reason, reliability_state

7. referral_events
- referral_event_id, referrer_user_id, referred_user_id, referral_code, qualification_status, reward_type, reward_value, credited_at

8. loyalty_ledger
- transaction_id, user_id, ride_trip_id, transaction_type, points_delta, balance_after, expiry_at, remarks

9. driver_packages
- package_id, deployment_id, package_name, price, validity_days, ride_categories_json, commission_benefit_json, activation_rules_json, status

10. driver_package_purchases
- purchase_id, driver_id, package_id, amount, promo_code, discount_amount, payment_mode, payment_ref, status, purchased_at, activated_at, expires_at, auto_renew

11. driver_package_transactions
- txn_id, purchase_id, event_type, actor_user_id, actor_role, remarks, created_at

12. package_refund_claims
- claim_id, purchase_id, driver_id, status, reason_code, created_by, created_at, final_decision_by, final_decision_at

13. package_refund_evidence
- evidence_id, claim_id, online_duration_min, dispatch_attempts, offers_received, accepted_count, declined_count, no_response_count, geo_zone_coverage_json

14. sos_alerts
- sos_id, user_id, ride_trip_id, triggered_at, lat, lng, status, escalation_history_json

15. policy_violation_alerts
- alert_id, driver_id, violation_type, severity, action_required, status, acknowledged_at, expires_at

16. high_impact_change_requests
- request_id, module, target_entity, old_value_json, new_value_json, rationale, impact_note, effective_time, status, created_by

17. high_impact_change_approvals
- approval_id, request_id, step_role (lead/manager/director), decision, decision_note, acted_by, acted_at

18. immutable_audit_logs
- audit_id, actor_user_id, actor_role, module, entity_name, entity_id, action_type, old_value_json, new_value_json, source, occurred_at

19. trip_events
- event_id (uuid), trip_id, event_type, sequence_no, payload_json, persisted_at

20. outbox_events
- outbox_id, aggregate_id, event_type, payload_json, status, retry_count, next_retry_at, dead_lettered_at

21. outbox_delivery_attempts
- attempt_id, outbox_id, provider, status, response, attempted_at

22. quote_requests
- quote_id, quote_type (outstation/package), pickup, drop, est_hours, est_days, est_km, estimated_amount_min, estimated_amount_max, validity_until, quoted_by, source

23. quote_booking_links
- id, quote_id, ride_request_id, linked_at

24. report_templates
- template_id, name, domain, filters_json, dimensions_json, metrics_json, visibility_scope, created_by

25. report_schedules
- schedule_id, template_id, frequency, next_run_at, recipients_json, output_format, enabled

26. report_runs
- run_id, template_id, started_at, completed_at, status, file_url, generated_by

27. notification_inbox
- notification_id, user_id, category, title, body, payload_json, is_read, created_at, expires_at

28. configuration_change_history
- change_id, deployment_id, module, field, old_value, new_value, changed_by, changed_at

## 4.3 Delta updates from newly finalized BRDs (05-08)

The following additions are required so migration scope reflects latest finalized BRD updates.

### A. Driver-Vehicle link governance (03 + 05 BRD alignment)

1. driver_vehicle_link (extend)
- Add link_type (primary/acting/vendor_assigned)
- Add link_status (pending/active/inactive/rejected)
- Add effective_from, effective_to
- Add approved_by, approved_at
- Add change_reason

2. driver_vehicle_link_history (extend)
- Add event_type (link_created/switch_requested/switch_approved/switch_rejected/unlinked)
- Add actor_user_id, actor_role
- Add old_vehicle_id, new_vehicle_id
- Add old_link_status, new_link_status
- Add event_reason, event_at

3. ride_trips (extend)
- Add assigned_driver_snapshot_id (immutable snapshot field)
- Add assigned_vehicle_snapshot_id (immutable snapshot field)

### B. Scheduled ride reliability (Package/Outstation)

4. ride_bookings (extend)
- Keep `ride_bookings` lightweight (assignment and status only).
- Move reconfirmation/SLA/no-show reliability fields to `ride_booking_reliability`.

4A. new table: ride_booking_reliability
- booking_id, pre_assignment_status, reconfirmation_status, rider_reconfirmed_at, driver_reconfirmed_at, primary_driver_id, backup_driver_id, no_show_actor_type, no_show_recorded_at, sla_due_at, sla_breach_at, sla_breach_reason, reliability_state

5. ride_requests (extend)
- Add rematch_attempt_count
- Add dispatch_timeout_at
- Add no_driver_found_at
- Add fallback_action_taken

### C. Serviceability and geo-governance

6. ride_requests (extend)
- Add pickup_zone_code, drop_zone_code
- Add serviceability_status (serviceable/out_of_service/restricted)
- Add serviceability_reason_code
- Add location_tamper_flag

7. new table: service_zones
- zone_id, zone_code, zone_name, city_code, status, effective_from, effective_to

8. new table: geo_fences
- geofence_id, zone_id, geofence_type (serviceable/restricted/no_pickup/no_drop), polygon_geojson, is_active

9. new table: serviceability_decisions
- decision_id, ride_request_id, pickup_result, drop_result, reason_code, evaluated_at, evaluated_by_source

10. new table: location_tamper_events
- event_id, user_id, role, event_type, confidence_score, observed_at, action_taken

### D. Driver settlement and payout operations (driver-only monetization)

11. new table: driver_wallet_ledger
- ledger_id, driver_id, entry_type (commission/package/adjustment/reversal/payout), amount, balance_after, reference_type, reference_id, created_at

12. new table: driver_settlement_cycles
- cycle_id, cycle_start_at, cycle_end_at, status, total_drivers, total_payable_amount, generated_at, closed_at

13. new table: driver_settlement_items
- item_id, cycle_id, driver_id, gross_earning, commission_due, package_due, adjustments, net_payable, settlement_status

14. new table: driver_payouts
- payout_id, driver_id, settlement_item_id, payout_status, payout_ref, initiated_at, completed_at, failed_reason

15. new table: driver_payout_failures
- failure_id, payout_id, failure_code, failure_message, retry_count, next_retry_at, resolved_at

### E. Referral/Reward and Rating lifecycle completeness

16. referral_events (extend)
- Add campaign_id
- Add qualified_at
- Add rejected_reason_code
- Add reversed_at, reversal_reason

17. coupons (extend)
- Add coupon_status (issued/active/reserved/used/expired/cancelled)
- Add campaign_id
- Add reserved_at
- Add used_at

18. feedback / ratings (extend)
- Add moderation_status
- Add moderation_reason_code
- Add disputed_flag
- Add disputed_at, resolved_at

19. new table: rating_disputes
- dispute_id, ride_trip_id, raised_by_user_id, against_user_id, reason_code, status, sla_due_at, resolved_by, resolved_at

### F. Unified support ticket lifecycle and SLA controls

20. tickets (extend)
- Add waiting_on_type (rider/driver/internal_team/system)
- Add reopened_count
- Add parent_ticket_id
- Add duplicate_of_ticket_id
- Add breach_risk_flag

21. new table: ticket_status_history
- history_id, ticket_id, from_status, to_status, changed_by, changed_at, reason_code

22. new table: ticket_assignments
- assignment_id, ticket_id, assigned_team, assigned_user_id, assigned_at, unassigned_at, assignment_reason

23. new table: ticket_sla_events
- sla_event_id, ticket_id, event_type (warning/breach/pause/resume), event_at, metadata_json

## 5. API impact list (as per BRD)

Architecture clarification: because each customer deployment has its own isolated backend/database, `deployment_id` is not required on core/business tables. Deployment identity remains relevant only inside the independent configuration module where configuration values are stored and resolved at runtime.

## 5.1 Existing APIs likely to change

1. Account and User APIs
- api/Account/send-otp, verify-otp, resend-otp, me
- api/User/GetUserDetails, upsertUserDetails
Changes needed: referral input, plugin-aware profile fields, stronger login audit metadata.

2. Taxi and Trip APIs
- api/taxi/book-ride, calculate-fare, update-booking, cancel-ride
- api/ride-trips/update-status, complete, update-distance
Changes needed: booking source, idempotency keys, sequence handling, policy snapshot persistence, package eligibility checks.

3. Driver APIs
- api/DriverStatus/update
- api/driver-shift/shift-logs
- api/DriverDetail/*
Changes needed: lock states, package-linked eligibility, richer shift metrics, policy-violation state.

4. Admin and Document APIs
- api/Admin/approve, reject
- api/Document/Approve, UpsertDocument
Changes needed: maker-verifier-approver workflow transitions, multi-step decision log.

5. Support APIs
- api/Tickets/*
- api/service-requests/*
Changes needed: query-type classification, SLA timestamps, escalation, package-refund case linkage, SOS linkage.

6. Metadata/Pricing APIs
- api/metadata/*
- api/peak-hours/*
Changes needed: deployment-aware policy and pricing snapshots, publish/effective-time model.

## 5.2 New APIs required by BRD

1. Deployment and white-label configuration
- /api/deployments
- /api/deployment-branding
- /api/deployment-features
- /api/deployment-rules

2. Rider plugin APIs
- /api/referrals/*
- /api/loyalty/*
- /api/saved-addresses/*

3. Driver package monetization APIs
- /api/driver-packages/*
- /api/driver-package-purchases/*
- /api/package-expiry-reminders/*
- /api/package-refund-claims/*

4. Advanced admin workflow APIs
- /api/high-impact-change-requests/*
- /api/high-impact-change-approvals/*
- /api/permission-policy/*

5. Support escalation and safety APIs
- /api/sos-alerts/*
- /api/policy-violations/*
- /api/support-escalations/*

6. Quote and reporting APIs
- /api/quote-desk/*
- /api/report-templates/*
- /api/report-schedules/*
- /api/report-runs/*

7. Reliability and audit APIs
- /api/audit-logs/*
- /api/trip-events/*
- /api/outbox-monitoring/*

---

## 5A. Multi-Role Architecture Specification (BR-19, BR-20, NFR-14, NFR-15)

### Multi-Role Support Design (BR-19)

A single identity (mobile number / user account) in the platform can operate under multiple roles depending on the application context:
- Same person can be a **Rider** in the Rider app and a **Driver** in the Driver app within the same deployment.
- Each role represents a distinct operational context with separate authentication, session, permissions, and data visibility.

**Database implications:**
- `users` table maintains identity context (`user_id`, `mobile_phone`, `email`).
- `user_roles` table maintains role assignments per user (linking `user_id` to role codes e.g., 'rider', 'driver', 'admin').
- Profile and operational tables (`driver_details`, `user_details`) scope to the role context, not just the user.

### Per-User-Per-Role Refresh Token Isolation (BR-20, NFR-14)

**Requirement:** Only a single active refresh token shall be valid per user-role combination at any point in time. Issuing a new refresh token must invalidate any previously issued refresh token for that specific user-role pair.

**Database implementation:**
- `user_refresh_tokens` table enforces unique constraint on `(user_id, role) WHERE revoked_at IS NULL`.
- When a new token is issued for a user-role pair, the previous active token is revoked (set `revoked_at = now()`).
- Token validation logic must check:
  - Token is not revoked (`revoked_at IS NULL`)
  - Token has not expired (`expires_at > now()`)
  - Token matches the requested role (`role = requested_role`)
  
**Reason for this design:**
- Prevents unauthorized role-switching (session hijacking across roles).
- Ensures clean session cleanup when user logs out from one role.
- Supports device recognition logging (`device_id`, `ip_address`) for security audit without cross-role contamination.

**Example scenarios:**
1. User logs in as Rider → receives refresh token (user_id=123, role='rider', token_hash=abc123).
2. User switches app context to Driver → new login creates new token (user_id=123, role='driver', token_hash=def456).
3. Both tokens can coexist; each is independent and isolated.
4. If user logs out from Rider app → revoke token for (user_id=123, role='rider'); Driver token remains valid.
5. If user logs in again as Rider from a different device → create new token for (user_id=123, role='rider'), invalidating previous Rider token.

---

## 6. Recommended migration sequencing (documentation plan)

Phase M1 - Foundation and deployment configuration
- deployment_profiles, deployment_feature_flags, deployment_business_rules, deployment_branding
- configuration_keys_catalog, enum_value_catalog, deployment_configuration_values
- V2 cleanup drops: remove legacy identity/booking compatibility columns and deprecated artifacts

Phase M2 - Ride lifecycle integrity
- ride_requests/ride_trips/ride_history/ride_payments column additions
- ride_bookings, trip_events, outbox_events, outbox_delivery_attempts

Phase M3 - Driver package monetization
- driver_packages, driver_package_purchases, driver_package_transactions
- package_refund_claims, package_refund_evidence
- driver_status and driver_shift_log extensions

Phase M4 - Support and governance
- tickets/ticket_comments extensions
- sos_alerts, policy_violation_alerts
- high_impact_change_requests, high_impact_change_approvals
- immutable_audit_logs, configuration_change_history

Phase M5 - Product growth modules
- saved_addresses, referral_events, loyalty_ledger
- quote_requests, quote_booking_links
- report_templates, report_schedules, report_runs
- notification_inbox

## 7. Risks and constraints to handle in implementation phase

1. V2-only contract rollout
- No backward compatibility required; all mobile/admin clients must use V2 payload contracts.

2. Enum safety
- Central enums in shared abstractions must be versioned carefully to avoid breaking old app binaries.

3. Data migration
- Existing rows need defaults for deployment_id, booking_source, workflow status, and new mandatory fields.

4. Audit immutability
- Audit/event tables should be append-only with strict update/delete restrictions.

5. Performance
- New analytics/reporting tables and JSON fields require indexing strategy from day one.

## 8. Deliverables completed in this analysis phase

- BRD to DB delta mapping
- Existing table extension list
- New table proposal list
- New/changed API impact list
- Migration sequencing plan (M1-M5)

No code changes performed.

## 9. V2 explicit cleanup list (drop/remove)

1. Drop column `users.role_id` (single-role legacy).
2. Do not add compatibility placeholders in core tables for old app payloads.
3. Keep reliability/SLA details out of `ride_bookings` and in `ride_booking_reliability`.
