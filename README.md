# Basement Suite Cleaning Checklist

A single-file, offline-friendly web app for cleaning and prepping a basement
Airbnb suite between guests. Open `index.html` in any browser (or add it to a
phone home screen) and work through two side-by-side checklists. Progress is
saved on the device, and a "Mark Complete" button can notify the host when a
clean is finished.

## Features

- **Two checklists**
  - **Turnover** – the full reset performed after a guest checks out.
  - **Prep** – a light refresh before a check-in when the unit was already
    turned over but has been sitting.
- **Progress tracking** – per-section and per-list counts plus a progress bar.
- **Collapsible sections** – tap a section header to collapse/expand it, or use
  the *Expand all* / *Collapse all* buttons.
- **"All" toggle** – check or clear every item in a section at once.
- **Dark / light theme** – toggle button, remembered per device.
- **Local persistence** – checkmarks and open/closed sections are saved in the
  browser's `localStorage` on that device only (nothing is uploaded).
- **Completion notification** – *Mark Complete* posts a summary (list name,
  optional note, done/total counts, and any missed items) to a configurable
  endpoint so the host gets pinged when a clean is done.
- **Print-friendly** – a dedicated print stylesheet expands all sections and
  strips interactive controls for a clean paper copy.
- **Installable (PWA-ish)** – includes a web app manifest and iOS home-screen
  meta tags so it can be launched full-screen from a phone.

## Usage

1. Open `index.html` in a browser, or host it on any static web host
   (GitHub Pages, Netlify, etc.) and open the URL.
2. Optionally add it to your phone's home screen for a full-screen, app-like
   experience.
3. Work through the **Turnover** or **Prep** list; checkmarks save automatically.
4. When finished, add an optional note and tap **Mark _\<list\>_ Complete** to
   notify the host. You'll be asked whether to clear the checkmarks so the next
   clean starts fresh.

Because state is stored in `localStorage`, checkmarks are specific to the
browser and device you use. Clearing site data (or using a different device or
browser) starts with an empty checklist.

## Completion notifications

The *Mark Complete* button sends a form-encoded `POST` to a notification
endpoint. This is wired up in the `<script>` block near the bottom of
`index.html`:

```js
var NOTIFY_URL = 'https://script.google.com/macros/s/…/exec';
var SECRET     = '…';
```

`NOTIFY_URL` points at a [Google Apps Script](https://developers.google.com/apps-script)
web app (the URL ends in `/exec`) that receives the payload and forwards it
(for example, by email). The request is sent with `mode: 'no-cors'`, so the
page never reads the response — it just fires the notification.

The payload fields are:

| Field    | Description                                     |
| -------- | ----------------------------------------------- |
| `secret` | Shared secret used by the endpoint to auth      |
| `title`  | Page title                                      |
| `list`   | `Turnover` or `Prep`                            |
| `note`   | Optional free-text note (max 300 chars)         |
| `done`   | Number of checked items                         |
| `total`  | Total number of items                           |
| `missed` | ` \| `-joined list of unchecked item labels     |

> **Security note:** `NOTIFY_URL` and `SECRET` are embedded in client-side
> JavaScript, so anyone who can view the page (or this repository) can read the
> secret and post to the endpoint. Treat the endpoint as low-trust: keep it to
> harmless notifications, and rotate the secret if the value is ever exposed
> publicly.

## Customizing the checklists

All checklist items are plain HTML inside the two `.col` blocks
(`#turn` for Turnover, `#prep` for Prep). To change an item, edit the text
inside its `<label>`; to add one, copy an existing line:

```html
<label><input type="checkbox"><span>Your new task</span></label>
```

Sections are `<section>` elements; the checkbox keys and section state are
assigned automatically by the script, so no IDs need to be maintained by hand.

> **Heads up:** checkbox save keys are index-based (`c0`, `c1`, …), so inserting
> or removing items shifts the saved state of the items after it. Adding items
> at the end of a section avoids disturbing existing saved checkmarks.

## Project structure

```
.
├── index.html   # The entire app: markup, styles, and script in one file
└── README.md    # This file
```

There is no build step and there are no dependencies — it's one self-contained
HTML file.
