# ServiceOS: camera-first MVP build plan

## Product direction

**Primary value:** turn camera-derived table occupancy and dwell-time signals into timely, understandable floor and service actions.

**Supporting value:** estimate ingredient use and stock runway from menu mix, recipes, stock counts, order timing, and per-ingredient supplier lead times. Inventory forecasting supports restaurant operations; it is not the primary sales story.

The local browser demo is a user-flow and calculation prototype. It is not a deployed service, has no connected camera, and does not implement real authentication or restaurant isolation.

## First production pilot

Build for one pilot restaurant first, while keeping tenant identifiers and authorization in the data model from day one.

### Camera-to-action path

1. **Camera input:** connect a supported IP camera or NVR stream through a restaurant-side gateway. Keep camera credentials in a managed secret store; never place credentials in browser code, source control, or project documentation.
2. **Calibrated table zones:** map camera coverage to named tables on the restaurant floor plan. A manager can review and correct the mapping.
3. **Detection events:** produce timestamped table-level signals such as occupancy changed, dwell threshold crossed, or table appears ready for reset. Store confidence, camera/zone, model version, and event time for audit and tuning.
4. **Operations state:** combine camera signals with explicit staff updates. Show uncertain signals as suggestions for staff review; do not silently claim a table is ready based only on ambiguous video.
5. **Action and feedback:** create a clear follow-up task, assign it to a staff member, notify the appropriate device, and record started/completed/manager-confirmed states. Capture overrides to improve calibration.
6. **Management reporting:** calculate occupancy, service time, turnover, task response, and camera availability from persisted events, with freshness and data-quality indicators.

For privacy, prefer table-zone occupancy and timing over identifying people. Confirm the pilot's applicable privacy rules, retention period, signage, and camera placement before enabling any real feed.

### Table-state evidence rules

- **Occupied:** a sustained, calibrated table-zone occupancy signal; brief crossings and temporary occlusion should not create a new seating event.
- **Possibly clear:** the zone appears empty for a site-tested interval. This is a camera suggestion, not proof that guests have left permanently.
- **Resetting:** begins from a recorded staff start action or a manager-accepted clear suggestion that creates a reset task.
- **Ready:** staff confirms cleaning/reset completion. A camera can support that the zone is empty, but an empty image alone cannot verify cleanliness.
- **Service check:** derive this from a restaurant-approved rule. If it depends on when food was ordered or served, require POS/order timestamps; otherwise call it a dwell-time threshold, not a claim that a guest is waiting.

Use these definitions in manual video labels and model evaluation. Keep occupancy transitions, operational staff actions, and reviewer labels distinguishable in storage and reports. Do not use person identity or identity recognition for this workflow.

### Camera pilot acceptance gate

Do not call a camera integration â€œsupported,â€ â€œlive,â€ or ready to trigger staff work until the exact pilot device and the following evidence are recorded. Set numeric performance and freshness limits with the restaurant before collecting the evaluation set; do not reuse an unvalidated universal threshold.

| Gate | Evidence required before pilot sign-off |
| --- | --- |
| Device and stream qualification | Record camera/NVR make, model, firmware, protocol/profile, codec, stream profile, auth mode, and test date. Validate access and decode through the intended restaurant-side gateway. A listed protocol or vendor logo alone does not pass. |
| Table-zone accuracy | Have an authorized reviewer label a representative held-out recording with table ID and occupancy state. Report per-table precision/recall, missed transitions, false transitions per camera-hour, and performance during occlusion, lighting changes, and table service. Agree limits with the restaurant before evaluation. |
| Event freshness and health | Measure event age and recovery after stream loss. Show stale/offline or unknown when camera data is missing; never convert missing frames into an â€œemptyâ€ table. Record reconnect results and the agreed freshness limit. |
| Suggestion-to-task safety | A table-level suggestion must include timestamp, zone, confidence, and model version. Require the configured review policy, reject duplicate/stale events, preserve human overrides, and verify the source event remains linked to any task. Do not infer that an empty table is clean or ready. |
| Staff action and fallback | Verify assignment, delivery to the intended staff device, acknowledgment, completion, and manager confirmation. Test the manual floor/task workflow during camera or network outage and document who owns each fallback. |
| Privacy and security release | Approve camera purpose, placement, signage, access, retention, and deletion. Complete every mandatory security checklist item and its verification evidence before using production credentials or footage. |

Keep the evidence set and results versioned by exact device firmware, gateway build, zone-map version, and model version. A pass applies only to that tested combination; update the qualification record after any material change.
### Supporting inventory estimate

For each menu item, store ingredient quantities and units per serving. Estimate ingredient demand as:

`expected portions per day Ã— ingredient quantity per portion`

Estimate runway as:

`usable stock on hand Ã· estimated daily demand`

Compare each ingredient's runway against `time until order placement + that ingredient's supplier lead time`. The local demo supports a default lead time with per-ingredient overrides. Flag runway at or below the estimated delivery time as at risk and show the assumptions behind the estimate. Incorporate POS order mix, receiving, counts, waste, and recipe yield over time. Keep purchase orders manual or out of scope until the estimates are trusted and the restaurant approves a purchasing workflow.

## Service architecture

- **Restaurant gateway:** outbound-only connection to approved camera/NVR streams; local buffering for short connectivity outages; no public inbound camera ports.
- **Detection service:** table-zone event processing, health/freshness monitoring, configurable confidence and dwell thresholds, and event audit trail.
- **Application API:** authenticated users, restaurant-scoped data access, floor plans, tables, events, tasks, inventory assumptions, and analytics.
- **Database:** relational storage with restaurant ownership on every operational record, server-side tenant authorization, and tenant-isolation tests.
- **Web app:** responsive manager and staff experiences; live floor updates; accessible task actions; stale-feed and uncertain-detection states; no camera frame presented as live unless the backend confirms it is fresh.
- **Integrations:** camera/NVR first for the core product; POS/menu and stock-count imports for better inventory estimates after the camera workflow is useful.

### Dataset and 8-hour replay plan

The linked [CAFE project page](https://dk-kim.github.io/CAFE/) describes the CafÃ© dataset as research data recorded at six cafes, with multiple group activities and annotations. It links a dataset download labelled about 150 GB. The associated paper and official codebase describe group activity detection (grouping people and classifying group activities), not restaurant table-state detection, table turnover, or waiter dispatch. Treat it as a possible research/evaluation reference, not a drop-in trained system. The project's website license does not by itself establish the dataset's reuse rights; verify the dataset terms before downloading, training on it, or showing footage.

The [Roboflow Smart Restaurant Surveillance project](https://universe.roboflow.com/visionspace-adnsg/ai-smart-restaurant-surveillance-veh3i) is a closer supplementary reference: its page lists 3,000 images, six versions, nine object classes (including person, customer, staff, waiter, food, and cleaning item), and a CC BY 4.0 dataset license. The page lists no published model and no project description; its sample workflow calls a general hosted detection service. The inspected v1 page says it is train-only, with 9,000 augmented training images and zero validation/test images. That version therefore cannot establish held-out accuracy. These are image object labels, not time-sequenced table occupancy/clear/reset ground truth; the classes do not identify which table is occupied. It may be evaluated as a possible generic object-detection starting point after checking the version, annotations, attribution, source provenance, and privacy suitability, but it is not the camera-to-table-and-staff system. Do not download or train on it until the chosen evaluation need is defined. Do not send restaurant CCTV to a hosted inference service or put its API key in a browser; a hosted processor would need an explicit privacy, retention, security, and cost review under the mandatory checklist.

An eight-hour recording is useful as a **replay/evaluation session**, but one video alone is not enough to claim a production-ready model. Start by processing a short, authorized, privacy-reviewed segment offline. Mark table-level ground truth (occupied, cleared/reset, ready) and compare detections to those labels. Track missed events, false alerts per hour, and alert delay. Then test longer replay and day/night or viewpoint changes. Keep the dashboard in clearly labelled replay/sample mode until an actual camera integration is connected and verified. Do not display or transmit a full raw eight-hour recording by default; prefer local processing, brief authorized review clips, and event summaries with a defined retention policy.

The occupancy-detection reference clip reviewed for this prototype is about 43 seconds long. It is a visual reference, not an eight-hour replay recording. Its frames have not been used for model training or to create live detections. Before an eight-hour replay demonstration, select an authorized recording of that duration and review the intended segment locally; the evaluation workspace defaults to the clip duration but requires the annotated interval to be selected explicitly.

### Camera evaluation scorecard

Before an operational pilot, agree with the restaurant on acceptable error and delay limits. For each reviewed recording, keep table ID and timestamp labels, and split training, tuning, and held-out evaluation by whole recording/shift or camera view rather than neighboring frames from the same clip. Compare table-level transitions using a pre-agreed time-matching tolerance and report:

- Event precision and recall for occupied, clear/reset-start, and ready transitions.
- False staff alerts per operating hour and missed transitions per table-hour.
- Median and 95th-percentile time from the human-labelled transition to the proposed alert.
- Results by camera/view, table zone, shift, lighting, crowding, and occlusion, plus camera uptime and stale-feed detection.
- How often staff dismiss or override a suggestion, and whether a notification was delivered or failed.

Do not set thresholds based on training clips or a single shift. Keep auto-dispatch disabled until held-out results meet the restaurant-approved limits and human override/failure handling has been exercised. The local review tool can compare a separately generated table-event CSV against manual labels for basic event-level precision/recall, false alerts per video hour, and mean absolute timing error. It does not run inference or training, validate the full scorecard, or establish production readiness.

### Camera compatibility strategy

The product goal is broad camera compatibility, not a promise that every CCTV device connects automatically. Build a vendor-neutral camera adapter layer around common stream protocols such as RTSP and ONVIF, then validate each recorder/camera, codec, authentication mode, network path, and frame rate in a compatibility matrix. Use a restaurant-side gateway so credentials and video remain under the restaurant's control. A device without a supported accessible stream needs a vendor-specific adapter or cannot be supported in that pilot. Start with the existing IP cameras/NVR after identifying its model and available protocol; never place passwords, stream URLs, or recorder credentials in browser code or project documentation.

ONVIF is a useful interoperability path, not a blanket guarantee that every feature will work. The official ONVIF documentation describes Profile T as advanced video streaming with H.264/H.265 support and recommends it as the successor to Profile S. ONVIF says Profile S conformance submissions end on March 31, 2027, while existing conformant devices can still use it. Verify the exact camera/NVR model and firmware in ONVIF's conformant-products directory, then test the actual stream from the restaurant gateway. Record exact model, firmware, RTSP availability, ONVIF profile (if shown), codec, authentication, and reconnect behavior. Do not mark a camera compatible from a protocol checkbox alone.

Pilot compatibility gates: (1) the gateway can reach the selected stream without exposing it publicly; (2) credentials can be stored and rotated outside the browser; (3) the stream decodes with a supported codec at the intended sampling rate; (4) time stamps and reconnects behave reliably; (5) table-zone calibration covers the target tables; and (6) loss, stale frames, and authentication failures become visible health states. Cameras that are analog-only, isolated behind an inaccessible recorder, or available only through a vendor cloud path may need an encoder, an approved vendor adapter, or may be unsupported for the pilot.

Only after offline evaluation and a supervised pilot should a high-confidence, stable table event create a staff notification. Keep a human-confirmed workflow for uncertain signals, allow staff override, and record alert delivery/failure. Do not automatically dispatch staff based only on an unvalidated research model.

## Delivery phases

1. **Pilot definition:** confirm restaurant, cameras/NVR, table/floor mapping, staff roles, and data/privacy constraints.
2. **Secure operations foundation:** real sign-in, restaurant setup, role permissions, tenant-scoped API/database, and audit events. Carry the full security checklist into release-blocking acceptance criteria.
3. **Camera pilot:** connect one supported camera path, map table zones, surface occupancy/dwell suggestions, handle lost/stale feeds, and let staff correct detections.
4. **Service workflows:** shared floor state, task assignment/notifications/completion, turnover analytics, and manager dashboard derived from real events.
5. **Menu publishing:** let managers maintain customer-facing dish descriptions, ingredient/allergen notes, prices, and availability; publish approved versions to a restaurant-scoped HTTPS guest page and issue table QR codes only after the shared backend, public-read isolation, and accessibility/privacy review are in place. The MVP guest page is informational and does not take orders.
6. **Inventory estimate:** validate recipe quantities and order mix against the pilot's actual menu; link published menu items to separate structured ingredient recipes; ingest sales and stock counts; display uncertainty and reorder lead-time risk without automatic purchasing.
7. **Hardening and rollout:** complete security and privacy review, tenant-isolation and authorization verification, failure-mode checks, accessibility review, backup/restore, monitoring, and staff onboarding before broader rollout.

## Release gates

- Authentication and role permissions are enforced by the server on every request.
- Every read/write is scoped to the signed-in user's restaurant; cross-restaurant access tests must fail safely.
- Camera stream credentials are protected, feed access is restricted, and health/freshness is visible.
- Detection uncertainty, missing frames, and manual overrides do not produce misleading live status.
- Tasks and table changes have an actor and timestamp audit trail.
- Inventory assumptions and units are visible; unknown values remain unknown; estimates are not presented as exact counts or as purchase orders.
- Managers can publish only within their restaurant; public menu reads expose only approved guest fields and availability, and never expose stock counts, camera data, staff tasks, or another restaurant's menu.
- Before publication, an authorized manager reviews each dish name, price, ingredient/allergen statement, and availability; a public menu update is versioned and timestamped.
- Complete the consolidated security checklist and required verification tests before production release.

## Mandatory security checklist

These requirements are release-blocking. Implement them according to the selected technologies; do not treat this list as satisfied by HTTPS and login alone.

### Secrets and Git

- Keep API keys out of source code.
- Use environment variables or a secrets manager.
- Scan Git history and rotate exposed credentials.

### Authentication

- Secure login.
- Strong password hashing.
- Protected admin accounts.
- Rate-limited login attempts.

### Permissions

- Enforce roles and restaurant-level data isolation on the server.
- One restaurant must never access another restaurant's records.

### Database and APIs

- Parameterized queries.
- Input validation.
- Restricted database privileges.
- Row-level security where applicable.
- Minimal API responses.

### Web protection

- XSS prevention.
- CSRF protection where relevant.
- Restrictive CORS.
- Secure session cookies.
- Security headers.

### Uploads and files

- Validate file type and size.
- Restrict access.
- Store uploads safely.
- Prevent executable uploads.

### Infrastructure

- HTTPS.
- Disable production debug mode.
- Secure configuration.
- Dependency scanning.
- Protected administrative endpoints.

### Abuse and cost

- Rate limits.
- Bot protection where appropriate.
- Usage monitoring.
- Spending alerts or caps for paid services.

### Verification

- Automated security tests.
- Dependency checks.
- Access-control tests.
- Documented release checklist.

### Key-handling interpretation

A public database key may be acceptable only when explicitly designed for client-side use and paired with correctly enforced access policies. Privileged service credentials must never be exposed to the browser. These controls must be evaluated for the actual stack selected.

### Current local prototype security status

The checklist above remains a mandatory production release gate. The current artifact is a browser-only demonstration, so the safeguards below are limited to local behavior and do not constitute a security review or production approval.

| Checklist area | Current evidence | Release status |
| --- | --- | --- |
| Secrets and Git | The project is tracked in Git and the GitHub repository is private. A dedicated history scan and credential-rotation evidence have not been recorded. | Not verified; required before publishing. |
| Authentication | The role selector changes demo presentation only. There is no login, password storage, or protected admin account. | Not implemented. |
| Permissions and tenant isolation | Role checks in browser code are presentation safeguards. No server enforces roles or restaurant boundaries. | Not implemented. |
| Database and APIs | The prototype uses browser local storage; no database or application API is deployed. | Not implemented. |
| Web protection | Dynamic user-entered text is escaped in the local UI. No production session, CSRF, CORS, or security-header controls exist. | Partial local safeguard only. |
| Uploads and files | Client-side demo limits: menu CSV 1 MB / 100 dishes / 24 columns; prediction CSV 2 MiB / 50,000 rows / 32 columns; local video 16 GiB. Selected video and CSV content are not sent to a server. | Demo-only; server-side upload controls are unimplemented. |
| Infrastructure | No production service or HTTPS deployment exists. | Not implemented. |
| Abuse and cost | No deployed API, bot controls, service monitoring, or paid inference is connected. | Not implemented. |
| Verification | No automated security, dependency, access-control, or cross-tenant verification has been run. | Required before release. |

Do not describe this matrix as a passing security assessment. Reassess every control against the selected production stack and retain evidence for each release gate.

## Showcase handoff

The private GitHub repository contains the product demo, supporting design notes, and eight interface screenshots. The guest-menu preview screenshot is still outstanding. The root `.gitignore` excludes common video formats and local `local-data/`, `private-data/`, `recordings/`, and `uploads/` folders; synced `sources/`, local instruction files, credentials, and private CCTV must remain outside the repository. The local menu CSV import does not replace server-side file validation and role checks.

## Inputs needed to build against the real pilot

1. Existing camera/NVR make and model, stream protocol support (such as RTSP/ONVIF), camera count, and approximate placement. Existing IP cameras/NVR are available; the specific model and protocol are still needed. Use [the camera compatibility matrix](CAMERA_COMPATIBILITY_MATRIX.md) to record each device's test evidence. Do not send passwords or stream URLs with credentials.
2. Pilot floor plan with table IDs and the camera views covering each table.
3. Staff roles and who can change table state, assign tasks, and confirm completion.
4. Menu/recipe quantities with units, typical daily order mix, current stock counts, and supplier lead times for inventory estimates. These details are not ready yet; the current calculator remains sample-only.
5. The first pilot restaurant is not selected yet; keep using the sample venue until a site is chosen.
6. The consolidated security checklist above is a mandatory project requirement.
7. A real customer menu/QR needs a stable HTTPS host, restaurant slug, shared tenant-scoped backend, manager publishing workflow, and a decision on table-specific context. None has been selected or deployed.
