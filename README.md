# PGG Suite

Merges the three field-experiment PWAs — `pgg_moderator_app`, `pgg-field-app`,
`pgg_survey_app` — into one entry point with a single shared Setup step.

## How it works

- `index.html` (this folder) is the **only** new code. It shows a Character
  Select screen (Project Planner / Project Team Member → Moderator or
  Facilitator / Subject), then one Setup form that asks the shared fields
  (village number/name, run, village type, date, chief, etc.) **once**.
- `moderator/`, `field/`, `survey/` are the three original apps, copied in
  **unmodified** except for a one-line "⇦ Suite" link added to each app's Home
  screen. No other logic, state shape, CSV export, or screen was touched.
- On "Save & Continue", the shell writes that role's setup directly into its
  app's own `localStorage` key, in the exact shape each app already expects
  (`pgg_moderator_state_v1`, `pgg_field_state_v1`, `pgg_survey_state_v1`),
  then navigates to `moderator/index.html`, `field/index.html`, or
  `survey/index.html`. Because each app's own router is already
  `route = STATE.setup ? 'home' : 'setup'`, it lands straight on Home — its
  own Setup screen is never shown.
- If a role's app already has a saved session on that device (e.g. reopening
  mid-experiment), the shell skips its own Setup screen entirely and hands
  off immediately — no re-asking, and the in-progress session resumes exactly
  as before.
- Shared fields are also cached under `pgg_shared_setup_v1`, so if a second
  role is set up on the *same device* afterward, the common fields (village
  number/name, run, date, village type, chief, ward, group counts) are
  pre-filled and don't need retyping. This does **not** sync across different
  physical phones — each device still needs its own Setup pass once.
- Project Planner has no further screen: Setup **is** its destination. It
  only writes the shared fields for whichever role picks up the device next.

## Deploying

Push this whole folder as the root of one GitHub repo (e.g. replacing the
three separate `tsuning0905/pgg_*` repos, or as a new repo) and enable GitHub
Pages on `main` / root. The three sub-apps keep their own `manifest.json`,
icons, and `sw.js` (offline caching), so each still works offline once
visited; only their `manifest.json` `start_url`/`scope` were changed from
absolute (`/pgg_moderator_app/…`) to relative (`./…`) so they work correctly
inside a subfolder.

## What was intentionally *not* changed

- No round logic, timers, CSV export/column, validation rule, or survey
  question was touched in any of the three apps.
- Each app keeps its own `localStorage` key, archive key, and service
  worker/cache — resetting or exporting data in one app has no effect on the
  others.
- Facilitator's group **Colour** and the 4-digit **PIN** stay device-specific
  (asked in the shell only when that role is chosen) — they were never part
  of the other apps' setup and aren't shared.

## Verified before delivery

- `node --check` passed on the JS in all four HTML files.
- Headless render test (jsdom): pre-seeding each app's `localStorage` with a
  shell-built setup object and dispatching `load` correctly skips that app's
  Setup screen and renders its real Home screen with the right village/run/
  name/colour details.
