# Camera-first MVP API and data contract

This is the implementation contract for the first shared pilot service. It is intentionally vendor-neutral until the camera/NVR model is known. Camera credentials stay at the restaurant-side gateway and are never sent to browser clients.

## Operating rules

- Every operational record has a server-assigned `restaurant_id`.
- The API derives allowed restaurant IDs from the authenticated membership. A request-supplied restaurant ID is never sufficient authorization.
- Camera events arrive through a device identity issued to the restaurant gateway. Browser users cannot submit trusted camera events.
- Detection events are observations, not authoritative human identity or proof of service completion.
- A new, ambiguous detection becomes a review suggestion. Only accepted events affect the shared floor state; a staff override is attributed and timestamped.
- The UI may show “live” only while the event/feed freshness check passes. Offline and stale feeds are separate visible states.
- Inventory is an estimate. It must expose its recipe, order-volume, stock-count, and lead-time assumptions and must not place orders automatically in this MVP.

## Core records

| Record | Required fields | Notes |
|---|---|---|
| Restaurant | `id`, `name`, `timezone`, `currency_code`, `created_at` | Tenant boundary; `currency_code` is an ISO 4217 currency configured per restaurant. |
| Membership | `user_id`, `restaurant_id`, `role`, `status` | Server-enforced role and tenant authorization. |
| Floor/Table | `restaurant_id`, `floor_id`, `table_id`, `seats`, `zone`, `status` | `table_id` unique within a restaurant, not globally. |
| Camera | `restaurant_id`, `id`, `display_name`, `protocol`, `health`, `last_seen_at` | Secret reference only; never store a stream password in browser-readable data. |
| Camera Zone | `restaurant_id`, `camera_id`, `table_id`, `zone_geometry`, `calibration_version` | Table mapping can be reviewed and versioned. |
| Detection Event | `restaurant_id`, `event_id`, `camera_id`, `table_id`, `event_type`, `observed_at`, `confidence`, `model_version` | Idempotent event ID; avoid raw video or person identifiers in the event store. |
| Manual Review Label | `clip_ref`, `timestamp_seconds`, `table_id`, `observed_state`, `reviewer_id`, `created_at` | Human ground truth for offline evaluation; never a trusted camera event or live table update. |
| Table Event | `restaurant_id`, `table_id`, `previous_state`, `new_state`, `source`, `source_event_id` (optional), `actor_id`, `occurred_at` | Keep the evidence source explicit: accepted camera suggestion, staff reset-task start, manager-confirmed reset completion, manual action, or override. Link accepted camera suggestions to their event ID for a joined audit trail. |
| Task | `restaurant_id`, `id`, `title`, `table_id/area`, `assignee_id`, `priority`, `status`, optional `source_event_id`, `created_by`, timestamps | Camera-created tasks retain their accepted source event for audit; keep assignment, start, completion, confirmation, and override history. |
| Menu Item | `restaurant_id`, `menu_item_id`, `name`, `category`, `description`, `guest_ingredients`, `price`, `currency_code`, `is_available`, `published_at`, timestamps | Manager edits are restaurant-scoped; guest reads expose only published menu fields and availability, never staff, stock counts, or camera data. |
| Menu Recipe | `restaurant_id`, `menu_item_id`, `ingredient_id`, `quantity_per_portion`, `unit`, `yield` | Keep operational recipe quantities separate from guest-facing ingredient/allergen text; convert compatible units explicitly and reject incompatible units. |
| Stock Snapshot | `restaurant_id`, `ingredient_id`, `quantity`, `unit`, `counted_at`, `counted_by` | Timestamped physical count or receiving adjustment. |
| Demand Assumption | `restaurant_id`, `menu_item_id`, `expected_portions`, `period`, `source` | Clearly distinguish manual estimate from POS-reported demand. |
| Supplier Lead-Time Assumption | `restaurant_id`, `ingredient_id`, `supplier_id`, `lead_time_days`, `source`, `updated_at` | Lead time may vary by ingredient/supplier; retain provenance and compare runway against order delay plus this item's lead time. |

## Minimum endpoints

### Signed-in web application

- `GET /v1/me` — signed-in identity and authorized restaurant memberships.
- `GET /v1/restaurants/{restaurantId}/floor` — floor zones, tables, current state, state age, and camera freshness summary.
- `GET /v1/restaurants/{restaurantId}/camera-health` — camera connection/last-seen status; never returns credentials or a privileged stream URL.
- `GET /v1/restaurants/{restaurantId}/detection-suggestions` — pending suggestions with event time, confidence, and reason.
- `POST /v1/restaurants/{restaurantId}/detection-suggestions/{eventId}/review` — accept or dismiss a suggestion; record reviewer and reason.
- `POST /v1/restaurants/{restaurantId}/tables/{tableId}/overrides` — manual state update with actor, reason, and timestamp.
- `GET|POST /v1/restaurants/{restaurantId}/tasks` — list/create restaurant-scoped tasks.
- `PATCH /v1/restaurants/{restaurantId}/tasks/{taskId}` — permitted task transition; server validates role and current state.
- `GET /v1/restaurants/{restaurantId}/analytics/shift` — metrics derived from persisted table/task events, with period and data freshness.
- `GET /v1/restaurants/{restaurantId}/menu` — restaurant-scoped menu records and manager editing metadata.
- `POST /v1/restaurants/{restaurantId}/menu/items` — create a menu item after server-side field validation.
- `PATCH /v1/restaurants/{restaurantId}/menu/items/{menuItemId}` — update dish information or availability; record actor and timestamp.
- `POST /v1/restaurants/{restaurantId}/menu/publish` — publish an approved menu version for the restaurant's public guest page.
- `GET /v1/public/{restaurantSlug}/menu` — rate-limited public read of published customer menu fields only; return availability and a version/update timestamp, and never expose internal restaurant records. Do not accept orders through this MVP endpoint.
- `GET /v1/restaurants/{restaurantId}/inventory/estimate` — consumption, stock runway, delivery risk, and assumptions.
- `PUT /v1/restaurants/{restaurantId}/inventory/assumptions` — authorized update of recipes, demand assumptions, stock counts, or supplier lead times.
- `GET /v1/restaurants/{restaurantId}/events` — authenticated server-sent updates scoped to the membership's restaurant.

### Restaurant-side camera gateway

- `POST /v1/device/camera-events` — gateway-authenticated, schema-validated, idempotent event ingestion. Gateway credentials are distinct from user sessions and limited to assigned camera IDs and one restaurant.
- `POST /v1/device/health` — signed or mutually authenticated heartbeat reporting connection and processing health, not video credentials.

## Detection event shape

```json
{
  "event_id": "gateway-generated-unique-id",
  "camera_id": "cam-main-01",
  "table_id": "T13",
  "event_type": "possible_occupancy_change",
  "observed_at": "2026-10-23T19:58:00Z",
  "confidence": 0.83,
  "model_version": "detector-version",
  "calibration_version": "zone-map-version"
}
```

The gateway authentication context supplies `restaurant_id`; clients cannot override it in the event body. Reject unknown tables/cameras, invalid confidence ranges, stale/future timestamps outside an allowed clock window, and duplicate event IDs safely.

## Offline video review labels

The local prototype can play a user-selected video up to 16 GiB and export a CSV containing `video_file` (a reviewer-provided label, not the source filename), `timestamp`, `timestamp_seconds`, `table_id`, `observed_state`, and `label_source=manual_review`. This is a human annotation export for offline model evaluation, not an API request and not evidence of a camera detection. Detector prediction CSVs are limited to 2 MiB, 50,000 prediction rows, and 32 columns; menu CSVs are limited to 1 MB, 100 dishes, and 24 columns. The browser does not upload or persist the selected video. A future evaluation importer must associate labels with an approved, access-controlled clip reference, validate state/table/time values, keep reviewer attribution, and keep labels separate from live `Detection Event` and `Table Event` records. Do not build a raw-video upload endpoint until the pilot has approved a documented purpose, consent/privacy and retention rules, strict file validation, access controls, and the full mandatory upload/security controls.

The prototype's optional local comparison expects a separate prediction CSV with `table_id`, `state`, and `timestamp_seconds`; optional `confidence` must be a number between 0 and 1. State values are `occupied`, `reset`, `ready`, or `check`. Reviewers select the fully annotated clip interval; predictions and labels outside it are excluded, and false alerts per video-hour use only that interval's duration. This is an event-level, one-to-one match: a prediction matches at most one human label when table, state, and timestamp tolerance match; extra predictions count as false alerts and unmatched labels as misses. A confidence cutoff filters rows only when they include confidence; rows with no confidence remain included. The evaluator reports basic precision, recall, false predictions per video-hour, and mean absolute timing error. Its export contains run metadata, the optional external model/pipeline version label, and aggregate overall/per-state metrics only; it omits the video and raw prediction rows. This lightweight aid does not train a model, replace held-out evaluation, or establish production performance. Object-detection rows such as `person` or `waiter` must first be processed into table-specific state-transition predictions by a separate pipeline.

## Inventory calculation

For an ingredient, calculate expected daily use by summing `expected portions per day × quantity per portion` for available menu items that consume it; unavailable menu items are excluded until made available again. Expected portions and a reviewed structured recipe are maintained per menu item. A guest-facing ingredient string is not a substitute for structured per-serving quantities. Estimate days remaining as `usable on-hand quantity ÷ expected daily use`. Compare this runway with order delay plus supplier lead time. Surface missing recipes, unknown stock, unit conversions, zero/unstable demand, and uncounted stock as uncertainty; do not silently convert missing values to zero. The local demo preserves blank stock, recipe, demand, and default lead-time fields as unknown. CSV-imported dishes remain incomplete until a manager enters portions and confirms recipe mapping.

## Table timing, evidence, and turnover

- A camera event may establish sustained occupancy in a calibrated table zone or suggest that the zone has become clear. Apply a configurable dwell/debounce interval to reduce passers-by, brief absences, and occlusions. Keep ambiguous signals as suggestions for review.
- A clear image is not proof that a table is cleaned or ready. Transition to reset only from an attributable staff action or an explicitly accepted clear suggestion that creates a reset task; transition to ready only after staff confirms reset completion. Store the source and actor for both transitions.
- Camera dwell time is not the same as a guest waiting for service. A staff-check suggestion requires a restaurant-approved operating rule and, where the rule depends on order timing, POS/order context. Without that context, label it as a dwell-time threshold only.
- Start service time when occupancy is confirmed by an accepted camera event or staff action. Measure reset time from reset-start to staff-confirmed ready. Count a completed turnover only with a valid same-table occupied → reset → ready sequence.
- Exclude incomplete or ambiguous intervals from averages and show the sample count. Never substitute zero for a missing timestamp. The local demo uses illustrative timestamps; production values must come from persisted, attributable events.

## Security implementation mapping

The full user-provided security checklist in `REAL_WORLD_BUILD_PLAN.md` is mandatory. Apply it to this contract as follows:

- Authenticate human users and gateway devices separately; hash passwords with a modern password-hashing function, rate-limit login, protect administrators, and keep all secrets out of the browser and Git.
- Authorize every endpoint by role and restaurant membership on the server. Database row-level policies are an additional guard, not a substitute for correct API authorization.
- Use parameterized database access, validate every request and transition, restrict database/service privileges, and return only fields needed by the caller.
- Use HTTPS, secure and appropriately scoped session cookies, CSRF defenses where applicable, restrictive CORS, XSS-safe rendering, security headers, and protected admin routes.
- For uploads, validate type and size, restrict access, store outside executable/public paths, and scan as appropriate. The first pilot should avoid user-uploaded video unless a concrete need is approved.
- Keep production debug off; scan dependencies and source history; monitor rate limits, abuse, and paid-service spend; alert on abnormal usage.
- Release only after automated security/dependency checks, role and cross-tenant access-control tests, gateway authorization tests, and the documented checklist are complete.

## Camera rollout acceptance

1. A manager can map a supported camera view to named table zones and save a versioned calibration.
2. A fresh camera signal appears as a timestamped suggestion with confidence and a clear source label.
3. An authorized person can accept or dismiss an uncertain suggestion; no ambiguous suggestion silently marks a table ready.
4. Accepting a suggestion can update a shared table state and create a follow-up task with a traceable source event.
5. Stale/offline camera state is visible, and the product does not label stale or sample data as live.
6. Staff can override incorrect table state; the system preserves the camera event and the human override in the audit trail.
7. Cross-restaurant and unauthorized-role requests fail closed in automated tests.
