# Showcase and GitHub release guide

## What the showcase should communicate

Lead with the camera-first restaurant floor workflow: a manager sees table states, reviews a clearly labeled sample camera event, and can create a service task. Make the camera boundary visible in every presentation: no live camera, computer-vision detection, or staff notification is connected in this prototype. Menu, guest preview, task, inventory, and analytics views support the main story; they do not replace it.

Keep the interface easy to scan. Use the Overview as the starting screen, show one primary action at a time, and leave advanced camera preparation, video review, and security requirements behind clear labels or expandable sections. Do not hide the demo-data or camera-offline labels to make screenshots look more impressive.

## Individual page images

Save one image per screen in `screenshots/`:

| File | Screen | What it should show |
| --- | --- | --- |
| `overview.png` | Overview | Camera-first sample scene, occupancy summary, task queue, camera health |
| `floor-plan.png` | Floor plan | Table states, floor tabs, selected table details |
| `staff-tasks.png` | Staff tasks | Queue with clear status and assignment actions |
| `analytics.png` | Management analytics | Shift metrics and clearly labeled sample history |
| `menu-manager.png` | Menu & availability | Dish editor, availability controls, and manager list |
| `guest-menu-preview.png` | Guest menu preview | Guest-facing dish details, ingredients/allergen notes, prices, and unavailable state |
| `inventory-outlook.png` | Inventory outlook | Recipe-based estimate with unknown values shown as unknown |
| `video-review.png` | Video review | Local/manual labeling controls with `LOCAL FILE · NO AI` visible |

Use a consistent browser size and zoom, show the complete page content without clipped dialogs, and capture a seeded sample shift. Do not include actual restaurant video, identifiable people, private menu data, credentials, or customer information. Keep the original page captures separate and link them from the project README. The available browser capture surface refuses access to this local `file://` page, so the images still need to be captured by the user from the visible local browser window and placed in `screenshots/`. Do not generate screenshot substitutes from HTML or CSS.

### Manual capture on Windows

1. Open `index.html` in the visible browser and use **Reset sample shift** so the pages show a clean demo state. Keep the sample-data and camera-not-connected labels visible.
2. Use the same browser window size and zoom for every image. Navigate to one page at a time from the sidebar; for `guest-menu-preview.png`, open **Menu & availability** and select **Preview guest menu**.
3. Capture the visible page with **Win+Shift+S** and save each image as PNG using the exact filename above. If a page is taller than the window, reduce zoom until its main content fits before capturing.
4. Save the eight separate PNG files in a `screenshots/` folder beside `index.html`. Use only seeded sample data. Leave real video unloaded on the Video review page.

The app currently runs from a local file, and the available Codex browser capture tool cannot access that `file://` page. These steps use a manual screenshot of the page already visible to the user; no capture images are generated from source code.

## GitHub handoff checklist

- Review the overview and every main page for readable spacing, responsive layout, clear action labels, and visible sample-data boundaries.
- Include the individual page images in `screenshots/` and link them from `README.md`.
- Keep the required security checklist in `REAL_WORLD_BUILD_PLAN.md`; do not present the local demo as production-secure or camera-connected.
- Check the intended repository contents before publishing. Keep secrets, local video files, synced project sources, user-specific configuration, and generated browser storage out of the repository.
- Add a license only after choosing the intended reuse terms.
- Choose public or private visibility before publishing. The workspace currently has no initialized Git repository or GitHub remote, so upload is a later release step.
