# Fare Operations 2026 — GitHub Pages Dashboard

Static replacement for the Google Sheets flight dashboard. All logic (status, sort rank) is computed client-side against today's date, so it stays current without touching formulas.

## Deployment

Deploys automatically via GitHub Actions (`.github/workflows/pages.yml`) on every push — no Pages settings to configure. Live at:

**https://jpdupree.github.io/FlightTracking/**

## Install on your phone

The dashboard is a PWA (manifest + service worker + icons), so it installs as a home-screen app and works offline with the last-fetched data.

- **iPhone (Safari):** open the URL → Share button → **Add to Home Screen**.
- **Android (Chrome):** open the URL → ⋮ menu → **Add to Home screen** (or tap the install prompt).

It opens full-screen with the split-flap DFW icon. Data refreshes from the network on each open; if you're offline it shows the last cached board.

## Adding trips from the app

The **+ Add Trip** button opens a form for fare watches, reprice watches, and booked segments. Two ways to save:

- **One-tap commit (recommended):** create a [fine-grained personal access token](https://github.com/settings/personal-access-tokens/new) scoped to only this repo with **Contents: Read and write** permission, and paste it into the form's token field once. It's stored in your browser's localStorage on that device only, and the app commits straight to `flights.json` — the board updates immediately and Pages redeploys in ~1 minute.
- **No token:** the app copies the trip's JSON to your clipboard and opens the GitHub web editor — paste it into the right array and commit.

**Removing trips:** tap **Edit** in the header — each row's status tile becomes a ✕ Remove button. Confirm, and it commits the deletion the same way (token) or opens the web editor (no token). Tap **Done** to exit.

## Daily workflow (replaces editing the spreadsheet)

All data lives in `flights.json`. Edit it in the GitHub web editor (press `.` in the repo or click the pencil icon), commit, and Pages redeploys automatically in ~30 seconds. From your phone, the github.com mobile editor works fine for this.

**Log a fare observation:** update `currentPrice` on the watch row. Status flips to BUY NOW / REPRICE automatically if it's at or under target.

**New watch:** add an object to `watches`:

```json
{
  "route": "DFW ⇄ XXX",
  "dates": "Mon DD–DD",
  "type": "buy",              // "buy" for unbooked, "reprice" for booked fares you're watching
  "currentPrice": null,        // null until you observe one
  "buyTarget": 300,
  "startChecking": "2026-08-01",
  "buyWindowEnd": "2026-09-15",
  "notes": "optional"
}
```

**Book a trip:** move it from `watches` to `booked` (fields: `route`, `dates`, `sortDate` for chronological sorting and the FLOWN flag, `fare`, `notes`). If a reprice watch applies, add a new `reprice`-type entry to `watches`.

## Status logic (ported from the Sheets formula)

| Type | Condition | Status |
|---|---|---|
| buy | today < startChecking | TOO EARLY |
| buy | currentPrice ≤ buyTarget | BUY NOW |
| buy | today ≤ buyWindowEnd | WATCH |
| buy | past window | LATE |
| reprice | currentPrice ≤ target | REPRICE |
| reprice | past buyWindowEnd | CLOSED |
| reprice | otherwise | WATCH |

Sort: actionable (BUY/REPRICE) first, then WATCH/LATE, then TOO EARLY — same as the old Sort Rank column.

## Local testing

`fetch()` is blocked on `file://` URLs, so don't just double-click index.html. Run:

```
python3 -m http.server
```

and open http://localhost:8000.
