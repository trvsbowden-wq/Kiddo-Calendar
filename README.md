# 📅 Bowden Family Calendar

A personal family calendar web app for tracking school events, Kido activities, dress-up days, and holidays for **Evelyn** 🦋 and **Violet** 🌸. Hosted on GitHub Pages and designed to work as an iPhone home screen app.

---

## Features

- **Monthly calendar view** with color-coded dots per event type
- **Today snapshot** — morning summary of the day's events at a glance
- **Prep reminders** — tomorrow's action items surfaced automatically
- **Coming Up strip** — scrollable list of the next 28 days' special events
- **Dress-up day highlighting** — 👗 badge on calendar days, auto-surfaced in reminders
- **Event drill-down** — tap any event for full details (time, teacher, location, action)
- **Search** — find any event across the full calendar
- **Filter by kid** — view Everyone, just Evelyn, or just Violet
- **School holidays** — Leander ISD 2025–26 and 2026–27 built in
- **Bank holidays** — US federal holidays shown separately
- **Live sync** — pulls new events nightly from a Google Sheet (via Apps Script)
- **🔄 Force refresh button** — clears cache and re-fetches instantly, sits next to the sync status indicator
- **6-hour localStorage cache** — works offline after first load
- **PWA-ready** — add to iPhone home screen for full-screen app experience

---

## Kids & Schools

| Kid | School | Program | Teacher(s) |
|-----|--------|---------|------------|
| 🦋 Evelyn | River Place Elementary (RPE), Leander ISD | Kindergarten | Ms. Jevnikar |
| 🦋 Evelyn | Kido River Place (afterschool) | Innovators | Ms. Rebecca Russell |
| 🌸 Violet | Kido River Place (full day) | Experimenters | Ms. Chiann, Ms. Dinara, Ms. Caitlin |

**RPE:** 6500 Sitio Del Rio Blvd, Austin TX  
**Kido:** 10625 Bonaventure Dr, Austin TX · 512-354-2808

---

## How It Works

### Static calendar data (built into the app)
- Leander ISD school holidays and staff development days (2025–26 and 2026–27)
- US bank holidays
- Known events hard-coded: 100th Day, Valentines, Read Across America, Holi, Rodeo Day, STEAM Night, etc.

### Live event sync (Google Sheet → Calendar)
A **Google Apps Script** (`school-email-processor.gs`) runs nightly and:
1. Scans incoming emails from Kido (`@famly.co`) and RPE (`parentsquare.com`)
2. Sends them to the Claude Haiku API to extract structured event data
3. Writes events to a Google Sheet

The calendar fetches that sheet data on load via two methods (in priority order):

| # | Method | URL |
|---|--------|-----|
| 1 | Apps Script Web App (JSON) | `https://script.google.com/macros/s/…/exec` |
| 2 | Published Sheet CSV (fallback) | `https://docs.google.com/spreadsheets/d/e/…/pub?output=csv` |

Fetched events are merged with the built-in static data. The result is cached in `localStorage` for 6 hours. A colored status indicator in the header shows sync state, with a 🔄 button to force-clear the cache and re-fetch immediately:

- 🟢 **Live** — freshly fetched
- 🟢 **Cached** — served from local cache (still fresh)
- 🟡 **Syncing…** — fetch in progress (shows ⏳)
- 🟡 **Stale cache** — fetch failed, showing older cached data
- 🔴 **Error** — fetch failed, no cache available

### Deduplication

The Apps Script deduplicates at two levels before writing to the sheet:

1. **Email-level** — skips any email whose `subject + date` has already been processed
2. **Event-level** — normalizes titles (strips emojis, lowercases, removes punctuation) before comparing, so `"🤠 Rodeo Day!"` and `"Rodeo Day 🤠"` are treated as the same event

The calendar also deduplicates at render time when merging sheet events with static events, using `title + kid` per date.

### Sheet event schema

Events in the Google Sheet / JSON response use these fields:

| Field | Description |
|-------|-------------|
| `date` | `YYYY-MM-DD` |
| `title` | Event name |
| `time` | Display time string |
| `kid` | `evelyn`, `violet`, or `both` |
| `type` | `school`, `kido`, `event`, or `holiday` |
| `description` | Short description shown in list view |
| `action` | 📌 Action item (shown in orange) |
| `location` | Location string |
| `teacher` | Teacher name |
| `dress` | `TRUE` / `FALSE` — flags as a dress-up day |
| `source` | Source email domain |
| `extracted` | ISO timestamp of when it was extracted |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| UI | React 18 (UMD, via CDN) + Babel standalone |
| Styling | Inline styles — no CSS framework |
| Fonts | Google Fonts: Nunito + Baloo 2 |
| Hosting | GitHub Pages (static) |
| Data pipeline | Google Apps Script + Claude Haiku API |
| Data storage | Google Sheets (published CSV + JSON web app) |
| Client cache | `localStorage` (6-hour TTL) |

No build step. No bundler. Single `index.html` file.

---

## Installation & Deployment

### GitHub Pages (current hosting)

1. Fork or clone this repo
2. Go to **Settings → Pages**
3. Set source to **Deploy from branch → main → / (root)**
4. Visit `https://YOUR-USERNAME.github.io/bowden-calendar/`

### Add to iPhone home screen

1. Open the URL in **Safari** on iPhone
2. Tap the **Share** button → **Add to Home Screen**
3. Tap **Add**

The app opens full-screen with no browser chrome, behaving like a native app. The following meta tags enable this:

```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="Family Calendar">
```

### Local development

No server needed — just open `index.html` in a browser. Note that the Google Sheet fetch may be blocked by CORS in some local environments; use the live GitHub Pages URL for full functionality.

---

## Updating Events

### Manual (one-off events)
Edit the `SE` object in `index.html` directly. Events use this shape:

```javascript
[dk(2026, 4, 15)]: [{
  title: "🎨 Art Show",
  time: "6:00 PM",
  kid: "evelyn",           // "evelyn" | "violet" | "both"
  type: "school",          // "school" | "kido" | "event" | "holiday"
  desc: "Annual art show in the gym",
  action: "🎨 Arrive by 5:45!",
  loc: "RPE — Gym",
  t: "Ms. Jevnikar",
  dress: false,
  prep: "Pick up printed photos"
}]
```

### Automatic (via email pipeline)
The Apps Script processor handles this. New emails from Kido and RPE are picked up nightly, parsed by Claude, and written to the sheet. The calendar will reflect new events within 6 hours (or immediately on a forced reload that bypasses cache).

---

## File Structure

```
/
├── index.html          # Entire app — React calendar, all static data, fetch logic
└── README.md           # This file
```

The Apps Script source (`school-email-processor.gs`) lives in Google Drive attached to the sheet, not in this repo.

---

## Privacy

This is a private family app. The repo is public only because GitHub Pages requires it for free hosting. The calendar contains no sensitive personal data — only school schedule information that would appear in any school newsletter.
