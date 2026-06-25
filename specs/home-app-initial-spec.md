# Home Inventory & Maintenance App — Build Specification

## Overview

Build a mobile-responsive single-file web application (`index.html`) for tracking home inventory and maintenance history. The app is for two users (husband and wife) sharing a single view of all data, with separate logins. It uses Supabase as the backend (database and authentication) and will be hosted on GitHub Pages.

---

## Tech Stack

- **Front-end:** Single `index.html` file, vanilla JavaScript, no frameworks
- **Back-end:** Supabase (Postgres database + Auth)
- **Hosting:** GitHub Pages
- **Cost:** $0

### Environment Variables

Do not hardcode credentials. Reference the following variables throughout:

- `SUPABASE_URL` — the project URL from Supabase Settings → API
- `SUPABASE_KEY` — the publishable (anon) key from Supabase Settings → API

Provide clear instructions in a `README.md` for how to set these before deploying.

---

## Authentication

- Two named users (John and Julie) with separate Supabase Auth accounts
- Email/password login
- Simple login screen on app load if no active session
- Logout button visible when authenticated
- No self-registration — accounts are created manually in the Supabase dashboard
- All data is shared between both users (no row-level separation by user)

---

## Data Model

Create the following tables in Supabase. Include the SQL to create them in `README.md` so the owner can run it in the Supabase SQL editor.

### Table: `inventory`

| Column | Type | Notes |
|---|---|---|
| id | uuid | primary key, default gen_random_uuid() |
| category | text | e.g. HVAC, Roofing, Plumbing, Electrical, Smart Home, Structure |
| name | text | e.g. "Water Heater", "Gas Fireplace" |
| make | text | nullable |
| model | text | nullable |
| serial | text | nullable |
| install_date | date | nullable |
| expected_lifespan_years | integer | nullable |
| service_interval_months | integer | nullable — blank for items with no recurring service |
| last_serviced_date | date | nullable |
| notes | text | nullable |
| created_at | timestamptz | default now() |
| updated_at | timestamptz | default now() |

**Computed display fields (calculated in the app, not stored):**
- `next_due` = last_serviced_date + service_interval_months
- `overdue` = next_due is in the past
- `end_of_life` = install_date + expected_lifespan_years
- `age_years` = today − install_date

### Table: `maintenance_log`

| Column | Type | Notes |
|---|---|---|
| id | uuid | primary key, default gen_random_uuid() |
| date | date | date of service/action |
| description | text | what was done |
| contractor | text | nullable — contractor name or "DIY" |
| cost | numeric | nullable |
| notes | text | nullable |
| inventory_id | uuid | nullable — foreign key to inventory.id |
| created_at | timestamptz | default now() |

---

## Application Structure

### Views

The app has three main views, navigable via a simple tab or nav bar:

1. **Dashboard**
2. **Inventory**
3. **Maintenance Log**

---

### View 1: Dashboard

Surfaces items needing attention without any push notifications — pull only.

Display two sections:

**Overdue / Due Soon**
- Lists inventory items where `next_due` is in the past (overdue) or within 60 days (due soon)
- Show: item name, category, last serviced date, next due date
- Visual distinction between overdue (e.g. red) and due soon (e.g. yellow)

**Aging Systems**
- Lists inventory items where `end_of_life` is within 3 years or already past
- Show: item name, category, install date, expected lifespan, end-of-life year
- Visual distinction between past end-of-life and approaching

---

### View 2: Inventory

**List view:**
- Grouped by category
- Each item shows: name, make/model, install date, age, next service due (if applicable)
- Overdue items visually flagged
- Add new item button
- Tap/click item to open detail view

**Detail / Edit view:**
- All fields editable inline
- Delete item (with confirmation)
- Shows linked maintenance log entries for this item, most recent first
- Button to add a maintenance log entry linked to this item (pre-fills the inventory_id)

---

### View 3: Maintenance Log

**List view:**
- Chronological, most recent first
- Shows: date, description, linked inventory item name (or "—" if freestanding), contractor, cost
- Filter by linked inventory item (optional, nice to have)
- Add new entry button

**Add / Edit entry:**
- All fields
- Inventory item selector (searchable dropdown) — optional, can be left blank for freestanding entries like gutter cleaning or exterior painting
- When adding a log entry linked to an inventory item, prompt: "Update last serviced date on [item name]?" — if yes, update `inventory.last_serviced_date` to the log entry date

---

## Seed Data

Populate the inventory table with the following initial records on first run (or include as a SQL insert block in `README.md`):

| Category | Name | Make | Model | Install Date | Lifespan (yrs) | Service Interval (mo) | Notes |
|---|---|---|---|---|---|---|---|
| Plumbing | Water Heater | Rheem | — | 2002-01-01 | 15 | — | Original to home, high priority for replacement |
| HVAC | Furnace | Carrier | — | 2004-01-01 | 20 | 12 | Filter replacement annually |
| HVAC | Central AC | Lennox | XC13N030 | 2015-01-01 | 15 | 12 | Annual service |
| Fireplace | Gas Fireplace | Montigo | ME36-DV-2 | — | — | 12 | Never professionally serviced, heavily sooted — priority |
| Roofing | Roof | Owens Corning | Duration Architectural | 2023-04-01 | 30 | — | Platinum lifetime warranty, Cloise & Mike Construction |
| Structure | Rear Deck | — | — | 2015-06-01 | — | — | Replacement project in progress |
| Irrigation | Irrigation Controller | Rachio | Iro Gen 1 | — | — | — | 5 zones |
| Smart Home | Thermostat | Google | Nest | — | — | — | — |
| Smart Home | Smoke/CO Detectors | Google | Nest Protect | — | — | — | Replacement due January 2031 |
| Electrical | Main Panel | Square D | QO 200A | 2002-01-01 | — | — | 40-space, garage-mounted, good condition |
| Fireplace | Gas Fireplace Insert | Montigo | ME36-DV-2 | — | — | 12 | Direct-vent |

---

## UI / UX Requirements

- Mobile-first, but fully usable on desktop (Mac and Windows)
- Clean, simple layout — this is a utility, not a showcase
- No external CSS frameworks required, but may use one if it keeps the file simple
- Confirmation dialogs before any delete action
- Form validation — required fields flagged clearly
- All monetary values displayed in USD with two decimal places
- Dates displayed in human-readable format (e.g. "Apr 2023"), stored as ISO 8601

---

## Deployment

Include a `README.md` covering:

1. How to create the two Supabase Auth user accounts (John and Julie) manually via the dashboard
2. SQL to create both tables (copy/paste into Supabase SQL editor)
3. SQL seed data inserts for initial inventory
4. How to set `SUPABASE_URL` and `SUPABASE_KEY` in the `index.html` before deploying
5. How to enable GitHub Pages on the repository
6. How to set up a free cron job at cron-job.org to ping the Supabase project every 5 days to prevent the free-tier inactivity pause (endpoint to ping: `SUPABASE_URL/rest/v1/inventory?select=id&limit=1`, with the publishable key in the `apikey` header)

---

## Out of Scope

- Push notifications or reminders
- Document storage
- Automatic maintenance schedule rules
- Mobile app (web only)
- Self-registration
- Per-user data separation

