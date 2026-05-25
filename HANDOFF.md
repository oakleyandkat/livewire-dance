# Livewire Dance — Handoff

A single-page web app for dance teams. Daily training checklist, weekly schedule, Nationals countdown, custom plans, coach schedule + class roster, competitions log, multiple roles, 24+ themes (custom builder + matching fonts), team rooms, and shareable team codes — all in one HTML file with browser storage.

## How to open it

Double-click `index.html`. It runs locally in the browser. No install, no server, no internet needed.

Each user's data lives in their own browser's localStorage (private, per-device, per-browser, per-team).

## File structure

```
livewire-dance/
├── index.html       ← the app (everything is in this one file, ~110 KB)
├── README.md        ← user-facing intro
├── HANDOFF.md       ← this doc
└── (leftover Next.js scaffold files — app/, package.json, etc. — harmless, ignore)
```

## Welcome flow (first load)

1. Enter your **name**
2. Pick **role(s)** — Teacher, Assistant, Dancer — multi-select (any combo)
3. Enter your **team name** (defaults to "Livewire")
4. Hit "Let's go ⚡"

Switch user anytime via the name chip in the top-right. Changing the team name on switch = entering a different team's "room" with separate data.

## Top of the app

- **Logo** — `⚡ {TEAM} DAILY` (updates with team name)
- **Team banner** — big centered display: `✦ TEAM ✦` with gradient text and yellow sparkles
- **Topbar buttons** (right side):
  - 🔗 Team Code (share/import schedule)
  - 📅 My Week (opens external planner)
  - 🎨 Theme picker (24 themes + custom builder)
  - User chip (shows name + roles, click to switch user)

## Roles (additive)

Each role adds capabilities. Mix and match.

| Role | What they see/can do |
|---|---|
| **Dancer** | Daily training checklist (4 sections), progress bar, streak, Livewire Expectations, Reminder card |
| **Teacher** | Coach Schedule (with class students), Daily Tasks list, can edit Nationals/practice count, can edit any day of the weekly schedule, can create custom team theme |
| **Assistant** | Coach Schedule, can edit weekly schedule EXCEPT days containing "Blackout Class" (those show 🔒) |

Examples:
- **Teacher only** → Coach Schedule + Daily Tasks, no dance training stuff
- **Teacher + Dancer** → Everything (player-coach)
- **Dancer only** → Dance training + Custom Plan, no Coach Schedule
- **Assistant + Dancer** → Dance training + Coach Schedule

## Features

### Hero countdown
Days until Nationals (default July 1, 2026). Teachers change date via ⚙ Admin button.

### Stats row (dancers only)
- **Practices left** — set by teacher
- **Day streak** — consecutive days where the full Daily Training Checklist was completed

### 🏆 Competitions
- Add comps with name + date + location + placement + score + notes
- 🔜 **Upcoming** sorted soonest-first, with day countdown
- 🎉 **Today's comp** gets a special celebration emoji
- 🏆 **Past comps** sorted most-recent-first, show "memories" with placement badge + your notes in italics
- Click any comp to expand → edit / delete

### 📅 Weekly schedule ("This Week" card)
- 7 rows (Mon → Sun) with each day's activity
- Today's row highlighted with theme accent gradient
- ← Prev / Next → buttons flip between weeks (date-based history)
- Checkboxes saved per date
- **Defaults from the photo:** Blackout Mon/Tue, Daily Training Checklist + variations Wed–Sun
- **Editing:**
  - Teachers can edit any day (saves to their personal schedule, not team-wide)
  - Assistants can edit any day EXCEPT Blackout Class days (🔒 locked)
  - Dancers can only check off

### Today's Plan timeline
Combined sorted timeline of the 4 training sections + custom items, by scheduled time. "Up next" tag indicates what's coming. Tap "+ set time" chips on section headers to schedule them.

### Daily training checklist (dancers only)
- 4 sections, 14 exercises from the original photo
- Tap any exercise to expand "How to do it" + Do/Don't cues
- Checks reset each day at midnight (or Reset Today button)
- Completing all 14 increments the streak

### My Custom Plan / Daily Tasks
- Each user adds their own items with optional times
- Appears in Today's Plan timeline
- Title relabels by role: "My Custom Plan" for dancers, "Daily Tasks" for teachers
- Doesn't count toward main streak

### 🤝 Coach Schedule (teachers + assistants)
- Weekly events with **day + class/event name + time + optional students** (comma-separated)
- Students shown under class name with 👥 prefix
- Quick-add buttons:
  - 📋 Daily Training (every day, 4:00 PM)
  - 🤸 2 Team Practices (Tue + Thu, 6:00 PM)
- 📤 Send to My Week — copies formatted schedule (with students) to clipboard + opens My Week planner

### 🔗 Team Code (sharing)
- Click 🔗 in topbar
- **Share:** Generate code → produces `LIVEWIRE-{base64}` string containing weekly schedule + coach events (with students) + Nationals date
- **Import:** Paste code → confirms before overwriting → applies the schedule
- One-way: when teacher updates, they generate a new code and re-send

### 🎨 Themes (24 + Custom)
- 24 preset themes — each swaps the font to match the aesthetic
- **Make Custom tile** at the end of the grid: pick your own background, text, primary, secondary, highlight colors with native color pickers. Saves as the team's custom theme. Each team can have its own.
- Preset themes: Electric, Cotton Candy, Sunset, Forest, Ocean, Galaxy, Blossom, Y2K, Cottagecore, Cyberpunk, Minimalist, Vaporwave, Strawberry, Lavender, Coquette, Tangerine, Eras Tour, Sour, Espresso, Renaissance, Brat, Pink Pony, Sweetener, Disco

### Team banner
Big centered banner under the topbar: `✦ TEAMNAME ✦` in gradient text with sparkle highlights. Updates with team name.

### 📅 My Week link
Topbar button opens the external My Week planner: https://tdmffss033vb-production-0ubs7ten.us-central1.suga.run/

## Team rooms (data scoping)

All per-user data is namespaced by team. Switching teams on the same device = entering a separate "room" with its own data.

Key pattern: `livewire.{TEAM}.{kind}.{username}`

Examples:
- `livewire.Livewire.checks.Mia` (Mia's exercise checks in team Livewire)
- `livewire.RoyalDance.checks.Mia` (a different Mia on a different team — separate data)

**Migration:** when loading, if a team-scoped key has no data, the loader falls back to the legacy un-scoped key (`livewire.checks.Mia`). So existing users keep their data.

**Dots in team names are stripped** (would break the key separator).

## Data model (localStorage)

All keys live under the `livewire.*` namespace.

| Key | Shape | Scoping |
|---|---|---|
| `livewire.dancer` | `{ name, roles, team }` | Device-wide (current user) |
| `livewire.theme` | string (theme id, or 'custom') | Device-wide |
| `livewire.{TEAM}.shared` | `{ nationalsDate, practicesRemaining }` | Per team |
| `livewire.{TEAM}.customTheme` | `{ bg, text, pink, cyan, yellow }` | Per team |
| `livewire.{TEAM}.checks.{name}` | `{ date, items: { itemId: true } }` | Per team + user |
| `livewire.{TEAM}.history.{name}` | `['YYYY-MM-DD', ...]` | Per team + user |
| `livewire.{TEAM}.schedule.{name}` | `{ sectionId: 'HH:MM' }` | Per team + user |
| `livewire.{TEAM}.custom.{name}` | `[{ id, name, time }]` | Per team + user |
| `livewire.{TEAM}.assist.{name}` | `[{ id, day, name, time, students }]` | Per team + user |
| `livewire.{TEAM}.weekly.{name}` | `{ Mon, Tue, Wed, Thu, Fri, Sat, Sun }` | Per team + user |
| `livewire.{TEAM}.weekchecks.{name}` | `{ 'YYYY-MM-DD': true }` | Per team + user |
| `livewire.{TEAM}.comps.{name}` | `[{ id, name, date, location, placement, score, notes }]` | Per team + user |

## Team Code format

```
LIVEWIRE-{base64-encoded JSON}
```

The JSON payload:
```json
{
  "v": 1,
  "team": "Livewire",
  "coach": "Ms. Lopez",
  "weekly": { "Mon": "...", ... },
  "events": [{ "id": "...", "day": "Tue", "name": "Beginner Ballet", "time": "16:00", "students": "Mia, Sara" }],
  "nationalsDate": "2026-07-01",
  "practicesRemaining": 30,
  "ts": 1748140000000
}
```

`buildTeamCode()` builds it. `parseTeamCode()` validates. `applyTeamCode()` overwrites the importer's `weekly` and `assistEvents`.

## Code map (inside `index.html`)

```
<style>     CSS — theming via CSS variables (--bg, --pink, --cyan, --font, --on-accent, etc.)
<body>      HTML: welcome modal, theme modal, custom-theme modal, team-code modal, dashboard cards
<script>
  CHECKLIST const         — 4 training sections, 14 exercises
  DEFAULT_WEEKLY const    — photo-based weekly defaults
  THEMES const            — all 24 theme color sets
  THEME_FONTS const       — font for each theme
  DAY_ORDER/DAY_FULL/MONTHS — date helpers
  TEAM() helper           — returns current team name (default "Livewire")
  KEYS const              — team-scoped localStorage key builders
  Storage helpers         — loadJSON (with legacy fallback), saveJSON, dateStr, today, fmtTime
  Role helpers            — userRoles(), hasRole()
  Color helpers           — hexToRgba, isDarkColor, shadeHex (for custom theme)
  showWelcome()           — first-load profile + role + team picker
  loadDancerData()        — pulls all per-user, per-team state from localStorage
  renderHeader()          — name, role badge(s), team logo + banner, countdown, stats, visibility logic per role
  renderComps()           — competitions list (upcoming + past)
  expandComp(id)          — inline edit form for a competition
  renderWeek()            — weekly schedule grid with edit/check + Blackout lockdown
  renderSections()        — main 4-section checklist + per-section time chips
  renderCustomPlan()      — user's custom exercises / daily tasks
  renderAssistSchedule()  — Coach Schedule (with students sub-line)
  renderTodayPlan()       — sorted timeline of scheduled things
  renderThemeGrid()       — 24 preset themes + "Make Custom" tile
  applyTheme()            — sets CSS vars + font on :root (handles 'custom' branch)
  applyCustomTheme()      — derives all CSS vars from 5 user-picked colors
  openCustomThemeModal()  — opens color picker
  buildTeamCode()         — encodes per-user state as LIVEWIRE- string
  parseTeamCode()         — decodes, validates
  applyTeamCode()         — overwrites recipient's local data
  setupCustomTheme()      — wires up color picker modal
  setupAssistSchedule()   — wires up form + preset buttons + Send to My Week
  setupTeamCode()         — wires up share/import modal
  setupComps()            — wires up Add Competition button
  init()                  — main entry: load data, apply theme, render, wire all events
```

## Known limitations

- **No cross-device sync.** Each browser/device is independent. Your phone and your laptop are separate islands.
- **No real-time chat.** Teachers and assistants can share schedules via team codes (async), but real instant messaging needs a backend.
- **Team code is one-way + manual.** Recipient pastes each time the sender updates. No auto-refresh.
- **Send to My Week is one-way + paste.** Real sync needs an API endpoint on the My Week project.
- **Streak only tracks the daily training checklist**, not weekly schedule completion.
- **No undo on deletes.** Custom items, coach events, and competitions are gone immediately.
- **Theme is per-device, not per-user.** Switching users on the same device keeps the theme.
- **Team rooms are local-only.** Different teams have separate data on this device, but data still doesn't sync between devices.
- **Team name can't contain dots** (would break key separator — they're stripped on save).

## What's next

Easy:
- Tagline / motto under the team banner ("Est. 2026", "State Champs 2025", etc.)
- "Reset weekly schedule to default" button for teachers
- Show streak based on weekly schedule completion too
- Print/export today's checklist as PDF
- Undo for deletes
- Notes field in team codes (async messaging — teacher includes a message recipients see)
- QR code generation for team codes (no more copy/paste — scan with phone)

Medium:
- Real-time chat (requires backend)
- URL-based team code sharing (`?code=LIVEWIRE-...` auto-applies)
- Notifications / reminders
- Roster: teachers add their dancers as a saved list, pick from it when adding classes

Bigger:
- **Real backend** — the abandoned Next.js scaffold is still in this folder with Drizzle + PGlite wired up. Would enable cross-device sync, real auth, shared team data, real chat.
- **Sync to My Week for real** — add POST endpoint inside My Week project.
- **OCR photo imports** — snap the team handout, app extracts the schedule.

## Credits

Built collaboratively with Oakley + Claude (Cowork mode) in May 2026.
