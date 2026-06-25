# Home Manager

A mobile-responsive single-page web app for tracking home inventory and maintenance history. Built with vanilla JavaScript and Supabase, hosted on GitHub Pages.

---

## Setup

### 1. Create a Supabase Project

1. Go to [supabase.com](https://supabase.com) and create a free account.
2. Create a new project and note your **Project URL** and **anon public key** from **Settings → API**.

---

### 2. Create the Database Tables

Open the **SQL Editor** in your Supabase dashboard and run the following:

```sql
-- Inventory table
CREATE TABLE inventory (
  id                      uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  category                text NOT NULL,
  name                    text NOT NULL,
  make                    text,
  model                   text,
  serial                  text,
  install_date            date,
  expected_lifespan_years integer,
  service_interval_months integer,
  last_serviced_date      date,
  notes                   text,
  created_at              timestamptz DEFAULT now(),
  updated_at              timestamptz DEFAULT now()
);

-- Maintenance log table
CREATE TABLE maintenance_log (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  date         date NOT NULL,
  description  text NOT NULL,
  contractor   text,
  cost         numeric,
  notes        text,
  inventory_id uuid REFERENCES inventory(id) ON DELETE SET NULL,
  created_at   timestamptz DEFAULT now()
);

-- Auto-update updated_at on inventory
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER inventory_updated_at
  BEFORE UPDATE ON inventory
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();

-- Enable Row Level Security
ALTER TABLE inventory ENABLE ROW LEVEL SECURITY;
ALTER TABLE maintenance_log ENABLE ROW LEVEL SECURITY;

-- Allow any authenticated user to read and write all rows
CREATE POLICY "auth_all_inventory"
  ON inventory FOR ALL TO authenticated
  USING (true) WITH CHECK (true);

CREATE POLICY "auth_all_log"
  ON maintenance_log FOR ALL TO authenticated
  USING (true) WITH CHECK (true);
```

---

### 3. Load the Initial Inventory

Run this in the Supabase SQL Editor to seed the inventory with your home's systems:

```sql
INSERT INTO inventory
  (category, name, make, model, install_date, expected_lifespan_years, service_interval_months, notes)
VALUES
  ('Plumbing',    'Water Heater',          'Rheem',         NULL,                    '2002-01-01', 15,   NULL, 'Original to home, high priority for replacement'),
  ('HVAC',        'Furnace',               'Carrier',       NULL,                    '2004-01-01', 20,   12,   'Filter replacement annually'),
  ('HVAC',        'Central AC',            'Lennox',        'XC13N030',              '2015-01-01', 15,   12,   'Annual service'),
  ('Fireplace',   'Gas Fireplace',         'Montigo',       'ME36-DV-2',             NULL,         NULL, 12,   'Never professionally serviced, heavily sooted — priority'),
  ('Roofing',     'Roof',                  'Owens Corning', 'Duration Architectural', '2023-04-01', 30,   NULL, 'Platinum lifetime warranty, Cloise & Mike Construction'),
  ('Structure',   'Rear Deck',             NULL,            NULL,                    '2015-06-01', NULL, NULL, 'Replacement project in progress'),
  ('Irrigation',  'Irrigation Controller', 'Rachio',        'Iro Gen 1',             NULL,         NULL, NULL, '5 zones'),
  ('Smart Home',  'Thermostat',            'Google',        'Nest',                  NULL,         NULL, NULL, NULL),
  ('Smart Home',  'Smoke/CO Detectors',    'Google',        'Nest Protect',          NULL,         NULL, NULL, 'Replacement due January 2031'),
  ('Electrical',  'Main Panel',            'Square D',      'QO 200A',               '2002-01-01', NULL, NULL, '40-space, garage-mounted, good condition'),
  ('Fireplace',   'Gas Fireplace Insert',  'Montigo',       'ME36-DV-2',             NULL,         NULL, 12,   'Direct-vent');
```

---

### 4. Create User Accounts (John & Julie)

1. In the Supabase dashboard, go to **Authentication → Users**.
2. Click **Add user** and create accounts for both John and Julie with their email addresses and passwords.
3. Under **Authentication → Settings**, disable **Email Confirmations** so accounts work immediately without clicking a confirmation link.

Both users share all data — there is no per-user data separation.

---

### 5. Configure the App

Open `index.html` and replace the two placeholder values near the top of the `<script>` block:

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';   // e.g. https://aovmbeqmdrjommnydaue.supabase.co
const SUPABASE_KEY = 'YOUR_SUPABASE_KEY';   // your anon/public key
```

The anon key is safe to include in a public file — Supabase Row Level Security restricts what it can access to authenticated users only.

---

### 6. Deploy to GitHub Pages

1. Push `index.html` (and `README.md`) to the `main` branch of this repository.
2. Go to **Settings → Pages** in the GitHub repository.
3. Under **Source**, select **Deploy from a branch**, choose `main` / `/ (root)`, and click **Save**.
4. Your app will be live at `https://highatomicyield.github.io/home-manager/` within a minute or two.

---

### 7. Keep Supabase Alive (Free Tier)

Supabase pauses free projects after 7 days of inactivity. Set up a free cron job to ping it every few days:

1. Go to [cron-job.org](https://cron-job.org) and create a free account.
2. Create a new cron job with these settings:
   - **URL:** `YOUR_SUPABASE_URL/rest/v1/inventory?select=id&limit=1`
   - **Schedule:** Every 5 days (or `0 12 */5 * *`)
   - **Request method:** GET
   - **Request headers:**
     ```
     apikey: YOUR_SUPABASE_KEY
     Authorization: Bearer YOUR_SUPABASE_KEY
     ```

Replace `YOUR_SUPABASE_URL` and `YOUR_SUPABASE_KEY` with your actual values.

---

## Features

- **Dashboard** — overdue and due-soon service items (within 60 days), plus aging systems approaching end of life (within 3 years)
- **Inventory** — grouped by category; tap any item to view/edit details and its full maintenance history
- **Maintenance Log** — chronological log with optional link to an inventory item; prompts to update "Last Serviced" date when you add a linked entry
- Mobile-first, works on desktop (Mac/Windows)
- Zero cost: GitHub Pages + Supabase free tier
