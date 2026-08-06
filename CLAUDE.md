# Airbnb — Basement Suite Cleaning Checklist

## What this is
A single-file, static web app: an Airbnb cleaning checklist PWA for a basement
rental suite. It shows two lists side by side — **Turnover** (full reset after a
guest checks out) and **Prep** (light refresh before a check-in when the unit has
sat). Hosts tick items off; a "Mark Complete" button notifies the host.

## Layout
- `index.html` — the entire app. HTML, CSS, and JS are all inline. There is no
  build step, no bundler, no dependencies, no framework. Open it in a browser and
  it runs.
- `README.md` — title only.

To work on it: edit `index.html` and reload in a browser. Nothing to install or compile.

## How it works
- **State is `localStorage`, per-device.** Checkmarks (`airbnb-checklist-v4`),
  collapsed sections (`airbnb-collapse-v4`), and theme (`airbnb-theme`) are saved
  only in the current browser. There is no server-side state or sync.
- Checkboxes are keyed by DOM order (`c0`, `c1`, …) and sections by order
  (`s0`, `s1`, …). **Reordering or inserting checklist items shifts every
  later key**, which silently invalidates saved checkmarks for existing users.
  If you restructure the lists, bump the `KEY`/`CKEY` version suffix (currently
  `-v4`) so stale state is discarded cleanly rather than mis-applied.
- **Completion notification:** the "Mark Complete" button POSTs (mode `no-cors`)
  to a Google Apps Script web app (`NOTIFY_URL`) with a shared `SECRET`, the list
  name, an optional note, and the count of missed items. The Apps Script side is
  what actually notifies the host.

## Known tradeoff: the notification secret is client-side
`NOTIFY_URL` and `SECRET` are hard-coded in the inline JS (`index.html`, in the
`sendComplete` block). Because this is a static page, anyone who views source can
read the secret and forge completion notifications. This is inherent to the
"static page → Apps Script" design and is accepted for a low-stakes personal tool.
Do **not** remove them — they authenticate the notification. If this ever needs to
be hardened, the fix is to move the call behind a server (or rotate the Apps Script
deployment + secret), not to delete it from the page.
