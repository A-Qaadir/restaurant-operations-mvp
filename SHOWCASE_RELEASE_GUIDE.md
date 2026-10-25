# Showcase guide

## Product story

ServiceOS brings the restaurant floor and the service team into one view. Start with the overview, follow a table state through the floor plan, then show how staff tasks and shift reporting support the workflow.

This repository contains a local demo. Camera events are simulated, video labels are manual, and restaurant activity is sample data. Do not describe the demo as live camera detection or as a deployed service.

## Screenshots

Current captures are stored in `screenshots/`:

- `overview.png` and `overview-floor-task-panels.png`
- `floor-plan.png`
- `staff-tasks.png`
- `analytics.png`
- `menu-manager.png`
- `video-review.png`
- `inventory-outlook.png`

The guest-menu preview image is still to be added. Keep screenshots separate and use the demo state. Do not include private restaurant footage, credentials, or customer information.

## Walkthrough

1. Open `index.html` and select **Reset sample shift**.
2. On Overview, review the simulated camera suggestion and decide whether to accept or dismiss it. Acceptance updates only the demo floor and creates a local sample task.
3. On Floor plan, select a table and follow its service/reset actions.
4. On Staff tasks, start and complete a task, then return to the manager view to confirm it.
5. In Menu & availability, edit a dish or mark it unavailable and open the guest preview on the same device.
6. In Video review, select an authorized local clip and add manual labels. The clip is not uploaded by the demo.
7. In Inventory outlook, adjust sample stock, recipe, or supplier lead-time assumptions to review the estimate.

All changes stay in the current browser. The demo does not send staff notifications, publish a customer menu, connect to a camera, or create purchase orders.

## Release notes

The GitHub repository is private. Keep it private unless the owner chooses to change its visibility. Do not add real CCTV, passwords, API keys, customer information, or restaurant credentials. The security checklist in `REAL_WORLD_BUILD_PLAN.md` remains a required gate for any production release.
