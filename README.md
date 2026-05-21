# Bloom Raffle · SEDO 72nd Congress · Palma de Mallorca 2026

A two-page web app for capturing leads at the SEDO Congress booth via QR code:

- **`index.html`** — Public landing page. Visitors scan the QR, fill in name / email / specialization / clinic / country / phone, and get a raffle number (BL-001, BL-002, …).
- **`admin.html`** — Password-protected dashboard. Search, see live stats, draw a random winner, export everything to Excel.
- **`schema.sql`** — Supabase table + Row Level Security policies + the two RPCs the app uses. Run once.

**Venue:** Carrer de Felicia Fuster, 2, Llevant, 07006 Palma, Iles Balears, Spain

Tech stack: vanilla HTML + Supabase (Postgres + Auth) + SheetJS for the Excel export. No framework, no build step. Same pattern as the Bassoul-Heneine, payment-follow-up, and Kazakhstan-almaty apps.

---

## What's different vs. the Kazakhstan version

- **Default language is now Spanish.** Auto-detection still works: if the visitor's browser is set to English / French / Russian / Arabic, the page renders in that language instead. If detection fails, it falls back to Spanish (not English) because the event is in Spain.
- **IP-geolocation country mapping** now covers Spain + 20 Spanish-speaking countries (Mexico, Argentina, Colombia, Chile, etc.) — visitors from those countries are auto-switched to Spanish.
- **Phone input** defaults to Spain (+34) with `[es, pt, fr, it, de, gb, mx, ar]` as preferred countries.
- **Country form field** defaults to "Spain" and the dropdown is reordered with Spain + Iberia + Latin America at the top.
- **Excel export filename** is now `Bloom-Raffle-SEDO-Palma-YYYY-MM-DD.xlsx`.
- **All venue/event text** (HTML body + every language block) references SEDO 72nd Congress Palma de Mallorca 2026 and the Felicia Fuster address.

---

## Setup — 10 minutes total

### 1. Supabase (3 min)

You have two options:

**Option A — separate project (recommended):** Create a new Supabase project at <https://supabase.com> so SEDO leads don't mix with KazDentExpo leads.

**Option B — reuse the existing Bloom Supabase project:** entries will land in the same `raffle_entries` table as the Kazakhstan raffle. You can filter them later by `created_at` (everything during the SEDO Congress dates) or by `country`. If you go this way, you can skip step 1.2 because the table already exists.

1. (If Option A) Open the new project's **SQL Editor**, paste all of `schema.sql`, click **Run**. This creates the `raffle_entries` table, the RLS policies, and the two RPCs (`submit_raffle_entry` and `lookup_raffle_number_by_email`).
2. (If Option A) Open **Authentication > Users > Add user > Create new user**. Set the admin email + password. Tick **Auto Confirm User**. This is what you'll use to log into `admin.html`.
3. In **Project Settings > API**, copy:
   - `Project URL` (looks like `https://xxx.supabase.co`)
   - `anon public` key (a long string starting with `eyJ...` or `sb_publishable_...`)

The anon key is safe to put in the public HTML because RLS limits it to inserting only (via the `submit_raffle_entry` RPC).

### 2. Plug the Supabase config into the two HTML files (1 min)

Open `index.html` and `admin.html` and replace these two lines in each (top of the `<script>` block):

```js
const SUPABASE_URL = "https://YOUR-PROJECT.supabase.co";
const SUPABASE_ANON_KEY = "YOUR-ANON-KEY";
```

Same values in both files.

**Important:** the files currently contain the Supabase credentials from the Kazakhstan-almaty project. If you went with Option A (separate project), you must replace them or SEDO entries will land in the KazDentExpo table.

### 3. Deploy to GitHub Pages (3 min)

```bash
mkdir Bloom-SEDO-Palma && cd Bloom-SEDO-Palma
# copy index.html, admin.html, schema.sql, README.md here
git init
git add .
git commit -m "Bloom raffle — SEDO 72nd Congress Palma de Mallorca 2026"
git remote add origin git@github.com:bloomaligner-collab/Bloom-SEDO-Palma.git
git push -u origin main
```

Then on GitHub: **Settings > Pages > Source: main / root**. After a minute the public URL is live, e.g.:

- Public form: `https://bloomaligner-collab.github.io/Bloom-SEDO-Palma/`
- Admin: `https://bloomaligner-collab.github.io/Bloom-SEDO-Palma/admin.html`

### 4. Update the QR code on the booth artwork (1 min)

Send me the public URL once it's live and I'll regenerate the QR pointing at it, then re-export the booth artwork (PDF / SVG).

### 5. Test (2 min)

- Open the public URL on your phone in private/incognito mode. The page should render in Spanish if your phone is in Spanish, English if it's in English, etc. Fill in the form, submit. You should see `BL-001`.
- Open `admin.html`, sign in with the admin email + password from step 1.2. You should see your test entry. Click **Export to Excel** — it should download `Bloom-Raffle-SEDO-Palma-YYYY-MM-DD.xlsx`.
- Click **🎲 Draw a winner** to test the random pick.

You're done. The whole thing runs in the browser, costs nothing on Supabase's free tier (well under the 50k row / 500MB limit even with thousands of entries), and works for the full duration of the Congress.

---

## Language behaviour — how the auto-detection works

The page tries to pick the right language in this order:

1. **Browser language** — checks `navigator.languages` for `es`, `en`, `fr`, `ru`, `ar`. First match wins.
2. **If no match, defaults to Spanish.** (The original Kazakhstan version defaulted to English; this one defaults to Spanish because the event is in Spain.)
3. **IP geolocation refines this** — a background call to `api.country.is` returns the visitor's country. If they're in Spain, Mexico, Argentina, Colombia, Chile, Peru, etc., the page switches to Spanish. If they're in France, Belgium, Morocco, etc., it switches to French. Russian/CIS countries → Russian. Gulf countries → Arabic.
4. **Country dropdown override** — if the visitor manually picks a country in the form, language switches to that country's language.

So a Spanish dentist with Spanish browser sees Spanish immediately. A French dentist with French browser sees French immediately. An American with English browser sees English. An Italian with Italian browser sees Spanish (because Italian isn't in the supported list, it falls back to Spanish — the event default).

To add Italian, Portuguese, German, etc.: copy the `es` block in the `I18N` object, translate the strings, and the auto-detection picks it up automatically.

---

## What admin.html shows

- **Total entries · Countries · Today's entries · Specializations**
- **Search box** — instant filter across name, email, country, clinic, specialization, raffle number
- **📥 Export to Excel** — every row, all columns, ISO timestamps, named with today's date
- **🎲 Draw a winner** — picks one random entry uniformly at random, shows a modal with their raffle number, name, email, specialization. "Draw again" lets you re-roll if you want.
- **Refresh** — pulls latest entries (use this at the end of the last day before drawing)

---

## After the Congress

- Hit **📥 Export to Excel** one final time for your records.
- The data stays in Supabase — you can keep using it for follow-up campaigns. If you want it gone: Supabase Dashboard > Table Editor > raffle_entries > delete rows, or just drop the table.

---

## Notes / future tweaks (optional)

- **Prevent the same email entering twice**: already enforced — the `email` column has a unique index. If someone re-submits, the `submit_raffle_entry` RPC looks up their existing number and returns it (so they don't lose their original entry).
- **Add more languages** (Italian, Portuguese, German): add the language code to the `I18N` object in `index.html` — auto-detection picks it up. See the "Language behaviour" section above.
- **Add more questions**: add the column in `raffle_entries` (Supabase Table Editor) AND the input in `index.html` AND the column in `admin.html`'s table + Excel mapping. AND update the `submit_raffle_entry` RPC to accept the new parameter. Four small edits.
- **Custom domain**: GitHub Pages supports a custom domain like `sedo.bloomaligner.fr` — Settings > Pages > Custom domain.
- **Send "thanks" email automatically**: Supabase Edge Function on insert + your transactional email provider. Not wired up here, but easy to add.
