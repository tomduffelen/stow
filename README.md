# Stow

A small personal web app for the things you need to hand while travelling: door codes, plans, and notes. One tap from the home screen, works offline, and everything stays on your own phone.

Built because holiday information arrives as a mess of WhatsApp messages, printed cards, and screenshots, and none of it is ever to hand at the moment you're standing at a locked gate.

## What it does

- **Codes** — door codes, gate codes, wifi passwords. Tap a code to copy it.
- **Plans** — bookings and activities with a date picker, sorted chronologically, showing the day name because dates stop meaning much on holiday.
- **Notes** — titled notes for anything else.
- **Add from a photo** — snap or upload a photo of an info sheet, booking message, or screenshot. Claude reads it and suggests entries, which you review and edit before anything is saved.

## How data is stored

Everything lives in the browser's `localStorage`, on the device, tied to the site's address. Nothing is uploaded, synced, or backed up.

Consequences worth knowing:

- Clearing browser site data wipes it. So does uninstalling the browser.
- It does not sync between devices.
- There is no backup. If the phone is lost, so is the data.
- Anyone who unlocks the phone can read it. The app has no lock of its own.

The one exception is the photo feature: an image is sent to a Cloudflare Worker, forwarded to Anthropic's API, and the extracted text comes back. The image is not stored anywhere.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app: markup, styling, and logic |
| `manifest.json` | PWA metadata, makes it installable |
| `sw.js` | Service worker, caches the app for offline use |
| `icon-192.png` `icon-512.png` `icon-180.png` | App icons |
| `worker.js` | Cloudflare Worker source — deployed separately, not served from here |

## Setup

### Hosting

Served from GitHub Pages off the `main` branch. Any static host works; there is no build step.

### Photo extraction (optional)

The app works fully without this. To enable it:

1. Get an API key from [console.anthropic.com](https://console.anthropic.com).
2. Create a Cloudflare Worker and paste in `worker.js`.
3. In the Worker's **Settings → Variables and Secrets**, add a secret named `ANTHROPIC_API_KEY` with your key as the value.
4. Set `DEFAULT_AI_ENDPOINT` in `index.html` to your Worker's URL.

The key stays server-side and never reaches the browser.

### Installing on a phone

Open the site in Chrome, then **⋮ → Add to Home screen**. It launches full-screen with no browser chrome.

## Updating

Edit the files and commit. Bump the `CACHE` version string in `sw.js` on every change, otherwise phones keep serving the cached copy and the update appears to do nothing.

## Notes and caveats

- **Check what the AI extracts.** It misreads digits. A wrong door code is worse than no door code, which is why the review step exists and why it can't be skipped.
- The Worker URL sits in public code. Anyone who finds it could spend against the API key behind it. Acceptable for a personal tool with a small prepaid balance; not acceptable at any larger scale.
- Uses `localStorage` rather than IndexedDB. Fine for this volume, not for much more.
