# UC AthleTrack

A web-based sports training and athlete monitoring system with **predictive data analytics** for
University of the Cordilleras student-athletes.

UC AthleTrack turns daily training sessions, wellness check-ins and benchmark tests into a live
readiness index, an acute:chronic workload ratio, an explainable injury-risk score and a 14-day
readiness forecast — with separate dashboards for student-athletes, coaches and the sports office.

## Stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18, Vite, Tailwind CSS 3, React Router |
| Backend | Node.js, Express 4 |
| Auth | HMAC-signed session tokens, `scrypt` password hashing (`node:crypto`) |
| Data | SQLite (`server/data/athletrack.db`) through the built-in `node:sqlite` driver |
| Charts | Hand-built SVG components (no chart dependency) |

## Brand palette

The interface palette is sampled from the official University of the Cordilleras logo:

| Token | Colour | Use |
| --- | --- | --- |
| UC jade | `#00A86D` | Primary actions, active navigation, data series |
| Jade light | `#00BE7C` | Hover states |
| Mint text | `#8CEFC8` | Accent text on dark surfaces |
| Deep forest | `#0B291F` / `#04100A` | Panels, page background |
| Silver / white | `#E9F4EE`, `#93A99C` | Primary and secondary text |
| Caution / risk | `#FBBF24`, `#FB7185` | Semantic status only |

The UC Jaguars logo (`public/logo.jpeg`) and jaguar-fur background (`public/background-jaguar.jpeg`) are
bundled with the app, and `public/favicon.png` is generated from the logo.

## Quick start

Requires Node.js 22.5 or newer (the API uses the built-in `node:sqlite` driver).

```bash
npm install          # install dependencies
npm run dev          # database setup (first run) + API on :5001 + Vite on :3000
```

On the first run, `npm run dev` opens a small terminal setup wizard:

1. **Load demo data** — 20 athletes, 3 coaches, accounts, 90 days of sessions, schedules
   and analytics.
2. **Fresh database** — a blank SQLite database with no varsity programs yet, where you enter
   the administrator's name, email and password.

If a database already exists it is reused without prompting, so later runs go straight to the
dev servers. Run `npm run setup` at any time to open the menu again and replace the data
(you have to type `reset` to confirm). When no terminal is available, demo data is loaded
automatically.

Then open <http://localhost:3000>.

Other scripts:

```bash
npm run setup        # database menu: demo data, fresh database + own admin, keep existing,
                     #                 or toggle system maintenance mode (with a custom message)
npm run server       # API only
npm run client       # frontend only (expects the API on :5001)
npm run build        # production build into dist/
npm run preview      # preview the production build
npm run seed         # rebuild the demo database from scratch
npm run restore      # apply a backup staged from System oversight (API stopped)
npm test             # Vitest unit + isolated API tests (temporary database)
```

Environment variables in these scripts are set with `cross-env`, so `NODE_ENV=production` works in
cmd/PowerShell as well as in macOS/Linux shells. On Windows consoles without a UTF-8 code page the
setup wizard automatically switches to plain ASCII output, and `NO_COLOR=1` (or `FORCE_ASCII=1`)
forces plain text anywhere.

The database lives in `server/data/athletrack.db` and can be opened with any SQLite client. The
local API binds to `127.0.0.1` by default and remains available at `http://localhost:5001` through
the Vite proxy. Vite continues listening on `0.0.0.0:3000` for local network access. For a
production deployment, initialize the database first, set a high-entropy `AUTH_SECRET` (at least
32 random bytes), and set `HOST` deliberately; do not expose the API directly without HTTPS and a
trusted reverse proxy.

Optional integrations and data retention (the app works without any of them):

| Variable | Default | Purpose |
| --- | --- | --- |
| `KNOCK_API_KEY` | unset | Enables email through Knock workflows `athletrack-password-reset`, `athletrack-weekly-report` and `athletrack-notification` |
| `KNOCK_WORKFLOW_RESET` / `KNOCK_WORKFLOW_REPORT` / `KNOCK_WORKFLOW_NOTIFICATION` | names above | Override the Knock workflow keys |
| `APP_BASE_URL` | request origin | Public URL used in password reset links |
| `AUTO_REMINDER_MINUTES` | `30` | Background reminder, retention and weekly-email sweep interval; `0` runs once at boot only |
| `PROOF_RETENTION_DAYS` | `365` | Delete participation proof older than this; `0` keeps everything |
| `ATHLETRACK_DATA_DIR` | `server/data` | Alternate data folder, used by the test suite for isolation |

## Demo accounts

| Role | Email | Password |
| --- | --- | --- |
| Administrator | `admin@uc.edu.ph` | `Admin123!` |
| Coach (Basketball, Football) | `coach@uc.edu.ph` | `Coach123!` |
| Coach (Volleyball, Swimming) | `coach.kalinga@uc.edu.ph` | `Coach123!` |
| Coach (Athletics, Badminton, Table Tennis) | `coach.montoya@uc.edu.ph` | `Coach123!` |
| Student-athlete | `athlete@uc.edu.ph` | `Athlete123!` |

Every seeded athlete has an account in the form `first.last@uc.edu.ph` with password `Athlete123!`.

## Features

### Athlete profiles
Academic and sporting record, jersey, position, course, emergency contact, coach notes, injury
history, season goals with progress, plus benchmark tests (vertical jump, sprint times, Yo-Yo
distance, VO₂ max, agility, smash speed and more).

### Training progress monitoring
- Session logging by athletes or coaches: type, duration, RPE → session load (AU).
- Team sessions logged once and applied to a selected group.
- Weekly session targets and 28-day compliance, streaks, minutes and average intensity.
- Daily wellness check-ins (sleep, sleep quality, soreness, stress, mood, hydration).
- **Training calendar** — coaches schedule sessions (sport, date, time, location, focus) and mark
  attendance (present, late, absent, excused with a note); athletes see their upcoming block.
- **Attendance tracking** — per-session registers feed the attendance rate used on every dashboard
  and in reports.
- **Weekly training attendance (Monday–Saturday)** — a dedicated weekly register at `/attendance`.
  Coaches cycle each athlete's day between present, late, absent and excused (saved on click,
  future days locked) and add a late/excused reason per day; athletes see their own Monday–Saturday
  record and can check in for today. **Athletes must upload a photo or video proof of attendance
  before a check-in is accepted**, and each day shows whether proof is attached. The week exports to
  CSV, and the weekly training attendance rate appears on both the coach squad panel and the athlete
  overview.
- **Assessment entry** — coaches record benchmark results from the athlete profile; the history
  table compares each result against the intake baseline and can be corrected or deleted.
- **Coach feedback and assigned activities** — notes and extra work are delivered to the athlete's
  dashboard and inbox, and athletes can mark activities complete.
- **Coach reports and exports** — coaches can open date- and program-filtered attendance, session,
  athlete and assessment summaries, print reports, and download CSV exports from the Reports workspace.
- **Notifications** — in-app bell for new schedules, feedback, assignments, assessments, broadcast messages and proof submissions.

### Messages and participation proof
- **Message all athletes and coaches** — a coach or administrator can post one message from the
  Participation Proof page. It appears as a dismissible banner on every athlete and coach screen,
  lands in their notification bell, and each person dismisses it for themselves. Authors and
  administrators can remove a message from the same page.
- **Photo and video proof** — athletes upload a photo or a short video (up to 30 MB; JPG, PNG, WebP,
  HEIC, MP4, MOV, WebM) with the participation date and an optional note. Coaches can upload on an
  athlete's behalf and see every submission for the athletes in their programs, filterable by athlete.
  Owners and coaches can delete a submission; files are stored on disk (`server/data/uploads/`,
  gitignored) and served only to signed-in users who can see that athlete.
- **Where proof shows up** — the coach dashboard has a “Participation proof” panel listing every athlete
  in the coach's programs with their status for the last seven days (this week, earlier, or missing),
  sorted missing-first, with a coverage bar and a link to the proof page. Each athlete profile shows
  their latest photo or video with its date, uploader and note, plus a link to the full proof page.

### Private messages
- **Direct messages** between accounts from the Messages page: conversation list with unread counts,
  searchable contacts, and a thread view that marks incoming messages read when opened. Enter sends,
  Shift + Enter adds a line; each message is capped at 2,000 characters.
- **Who can message whom** — administrators can start a conversation with any active athlete, coach or
  administrator; coaches can message the athletes in their own programs and the sports office;
  athletes can message the coach assigned to them and the sports office. Everyone else is hidden from
  the contact list, and the API rejects a message to anyone outside that scope.
- Messages are private to the two participants: they are not shown on dashboards, message text is not
  recorded in the audit log, and the recipient gets a single in-app notification (bell) per message.
  A conversation stays readable by both participants even if a coach is later reassigned, but replies
  follow the current permissions.

### Predictive analytics
- **Readiness index (0–100)** — weighted wellness score adjusted for load spikes.
- **ACWR** — 7-day acute load against the 28-day weekly base, with an optimal 0.8–1.3 band and a
  reliability floor for athletes with almost no training history.
- **Injury-risk score (0–100)** — workload ratio, monotony, readiness, soreness, sleep and injury
  history, returned with named, human-readable drivers.
- **14-day readiness forecast** — damped linear regression over the last 21 days with a widening
  confidence band and model fit score.

### Role-based dashboards
- **Athlete** — readiness, forecast, load, risk, next sessions, attendance breakdown, assigned
  activities, coach feedback, sessions, goals, tests, wellness.
- **Coach** — squad KPIs, attendance, risk watchlist, weekly load, upcoming sessions with marking
  state, roster snapshot, program breakdown, assessments and feedback.
- **Admin** — institution KPIs, program table with attendance, risk distribution, load trend,
  priority athletes, full account management, reports and system oversight.### Reports and oversight

- Date-range and program filters across attendance, sessions, load and readiness.
- Per-program and per-athlete report tables, printable to a clean paper layout.
- CSV exports: attendance, sessions, athlete summary, assessments.
- **Varsity programs** — add or delete programs and see athlete, coach and schedule counts.
  A program that still has athletes, coaches or scheduled sessions cannot be deleted.
- **Audit log** — privileged actions (accounts, schedules, attendance, assessments, feedback,
  assignments, exports, programs) recorded with actor, role, time and detail.

### Data quality, recovery and operations

- **Automatic reminder sweep** — at boot and every `AUTO_REMINDER_MINUTES` (30 by default), missing
  wellness check-ins, today's session check-ins and due activities create one notification per athlete
  per day. The same sweep applies proof retention and, on Mondays when email is configured, sends the
  weekly summary to administrators.
- **Roster CSV import** — administrators upload or paste a CSV
  (`firstName,lastName,email,sport,position,jerseyNo,yearLevel,course`), preview every row with its
  problems, then create athlete profiles and accounts with one-time temporary passwords that can be
  downloaded as a CSV. Sport names must match an existing varsity program.
- **Schedule conflicts and recurrence** — creating or moving a session warns about overlapping sessions
  for the same program, coach or venue and asks before overriding. A session can repeat weekly for up
  to 16 weeks, optionally on chosen Monday–Saturday days. Each session's check-in QR opens the schedule
  and checks the signed-in athlete in.
- **Bulk register actions** — mark a whole program present, late, absent or excused for one day, or copy
  last week's marks into this week without overwriting days that are already marked.
- **Athlete context** — the athlete dashboard adds squad averages and percentiles for readiness,
  attendance and 7-day load, plus wellness and proof streaks and progress toward the weekly target.
- **Offline check-ins** — when the API cannot be reached the check-in is queued locally and retried when
  the browser comes back online; the service worker keeps the app shell available offline.
- **Self-service password reset** — “Forgot your password?” issues a single-use 30-minute link. It is
  emailed when email is configured; in local development without email the link is shown on screen.
- **Backup and restore** — System oversight downloads a consistent SQLite snapshot (`VACUUM INTO`). A
  backup can be validated and staged there, then applied with `npm run restore` while the API is
  stopped; the database it replaces is kept as a timestamped copy.
- **Eligibility review** — reports add a reviewable eligibility table and CSV combining session
  attendance, Monday–Saturday training and proof coverage.
- **Privacy and retention** — proof uploads have EXIF/GPS metadata stripped (JPEG and PNG) before they
  are stored, and proof older than `PROOF_RETENTION_DAYS` is deleted by the sweep.
- **Tests** — `npm test` covers the analytics models, auth/token handling, CSV validation, metadata
  stripping, eligibility maths, the offline check-in queue and the restore CLI. An isolated API instance
  exercises role boundaries, password reset, schedule conflicts/recurrence/check-in tokens, roster
  import, bulk attendance, reminders, squad context, data quality, proof privacy and self check-in,
  eligibility exports, email fallback and backup/restore staging.

### Onboarding and session safety

- A role-specific three-step guide opens once per account after sign-in and is remembered per account, so returning users do not see it again until they clear it.
- Each guide step links directly to the relevant entry point and jumps straight there when the user clicks it.
- Any logged-in user can reopen the guide from the header "?" button or from another part of the app that dispatches the same internal quick-guide event, so help is available on demand rather than only on first login.
- Athletes are guided toward the daily check-in and feedback loop; coaches toward schedule / attendance / triage; administrators toward reports / account access / system oversight.
- The schedule page labels past sessions as "Past sessions" and uses plain-language empty states, and helper tooltips explain readiness, ACWR, risk, compliance and load in everyday terms.
- Session tokens expire after 12 hours and refresh silently; a session guard warns five minutes before expiry and signs the user out if they leave the tab idle too long.


## First-use onboarding

New users automatically receive a role-specific welcome tour after their first sign-in. Athletes,
coaches and administrators see four short steps covering their main workspace, core actions and
Analytics Center. Users can move forward or backward, jump directly to a step, skip the tour, or
use the Quick Guide button to restart it later.

## Analytics notes

The Analytics Center is available from the sidebar for athletes, coaches and administrators. It centralizes
role-specific pie charts, trends and metric explanations and keeps the detailed formulas visible in plain
language. A terminology glossary explains RPE, AU, ACWR, readiness, risk score, wellness, monotony,
attendance rate, participation proof, compliance, benchmarks and forecasts.
Pie charts represent shares and category distributions; line charts are retained for readiness over time.

All models are deterministic and explainable, intended as coaching decision support rather than a
medical diagnosis. The seed deliberately includes athletes who are ramping load, under-training,
sleep-deprived and injured so every band and alert state is visible in the demo.

## Data & security

- Password reset links are single-use, expire after 30 minutes, and only a SHA-256 hash of the token
  is stored; using one bumps the account's token version so every existing session is signed out. A
  backup uses `VACUUM INTO` and restore is a separate CLI step so the live SQLite file is never swapped
  underneath the running API.
- Passwords are stored as `scrypt` hashes with per-user salts; new and reset passwords require
  12–128 characters. Administrator password resets automatically generate a secure 16-character temporary password,
  with regenerate, copy, and show/hide controls. Signed tokens expire after 12 hours and are invalidated when a password changes.
- Login attempts are throttled per client IP. API responses include a restrictive content security
  policy and browser security headers; JSON bodies are limited to 1 MiB, and browser API origins are
  restricted to same-origin, local development hosts, and the configured Freebuff development host.
- The API is bound to loopback in development, while Vite keeps its existing LAN host binding; set
  `HOST=0.0.0.0` explicitly only when a deployment requires network binding. This preserves the
  `localhost:3000` Vite-to-API proxy workflow.
- Private messages are stored in the `messages` table and returned only to the two participants; other
  accounts cannot read a thread through the API.
- Participation photos and videos are written to `server/data/uploads/` (gitignored) and streamed
  through an authenticated API route; only the athlete, their coaches and administrators can open a
  file. Upload bodies are limited to 30 MB and accepted only with image/video MIME types.
- This build uses local SQLite (`server/data/athletrack.db`) through Node's built-in `node:sqlite`.
  Production startup requires an initialized database and `AUTH_SECRET`; missing values fail closed.
- Security work reduces application-level risks but does not certify OWASP Top 10 compliance. Production
  use also requires HTTPS, secure deployment configuration, backups, dependency updates, access
  controls and a security review. The in-memory login throttle is per process; use a shared limiter
  at a trusted reverse proxy if multiple API processes are deployed.



Frontend
React 18
Vite
React Router
Tailwind CSS
JavaScript (ES Modules)

Backend
Node.js
Express.js
REST API architecture

Database
SQLite
Local file-based storage
Authentication & Security
HMAC-signed session tokens
scrypt password hashing
Per-user password salts
Token version invalidation
Role-based authorization

Analytics
Deterministic workload and readiness models
SVG-based charts:
Pie/distribution charts
Trend/line charts
Readiness and risk indicators
Plain-language calculation explanations

Development Tools
npm
Concurrently for frontend and backend development
Cross-env
PostCSS and Autoprefixer