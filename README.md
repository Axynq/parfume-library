# ÉCHO — Scent Journal

A responsive, bilingual (Ukrainian / English) perfume collection website. Built with plain HTML, CSS, JavaScript and Supabase. No framework, backend or build step required. Deployable to Vercel.

## Included features

- Public landing page with favorite fragrances, images, ratings and recent discoveries
- Search your existing imported `public.perfumes` catalog by name/brand
- Save fragrances into your personal journal; optionally create a catalog item manually
- Perfume details, pyramid of top / heart / base notes and accords
- Your 1–5 star rating and written personal impressions
- Editable top list with arrow controls
- UK / EN translation; iPhone responsive layout
- Supabase Authentication sign-in and Row Level Security for owner-only editing

## Setup (do this in order)

### 1. Keep the existing perfume catalog

Your imported `public.perfumes` table stays intact. It is expected to contain:

- `id` (text, primary key or unique), `name` (text), `brand` (text), `year` (text), `image` (text)
- `notes` (jsonb: `{ "top": [], "middle": [], "base": [] }`)
- `accords` (text[])
- `search` (text containing normalized searchable name and brand, including a generated column if already in use)

The website searches `search` using Supabase `ilike`. The previous `index.html` also relied on this column. If your table has no `search` column, adjust `app.js` or add a suitable generated search column first.

### 2. Create the only editor account

In Supabase Dashboard > Authentication > Users, create a user with your email and password. Copy the user's **UUID** (not email). Disable public registration in Authentication settings (Allow new users to sign up) if you don't want anyone else to create accounts.

### 3. Run the database script

Open `schema.sql`. **Replace** `00000000-0000-0000-0000-000000000000` on the `insert into private.site_owner` line with your user's UUID.

Run it in Supabase SQL Editor. It creates `entries`, owner verification, and RLS policies without deleting rows from `perfumes`.

**If you already have an `entries` table or existing policies**, inspect them before running: `create table if not exists` won't change incompatible columns, and old permissive policies may still grant unwanted access. Do not run the SQL blindly on a shared project. The script resets the `private.site_owner` list to a single person.

### 4. Configure the frontend

Edit `config.js`:

```js
window.SCENT_CONFIG = {
  supabaseUrl: 'https://YOUR_PROJECT.supabase.co',
  supabaseKey: 'sb_publishable_...'
};
```

Get a **publishable** key from Supabase > Project Settings > API Keys. A legacy `anon` key also works. **NEVER use `sb_secret_...` or `service_role` in browser code**, or commit them to GitHub. The publishable key is intended to be public; RLS enforces write permissions.

### 5. Deploy to Vercel

Upload these project files to a GitHub repository, then in Vercel choose Add New > Project > Import repository. Choose framework preset **Other**, no build command, and output directory `.` (project root). Deploy.

You can also test by serving the directory locally:

```bash
python -m http.server 8000
```

Open `http://localhost:8000`.

## Usage

1. Open the site publicly: all visitors see the home dashboard, your collection and ranking.
2. Click **Sign in** and log in as the Supabase owner.
3. Click **Add fragrance**, search the imported catalog, select an item.
4. Open a perfume card to rate it, write notes and add it to your top list.
5. Open **My ranking** to move entries up or down.
6. Switch between Ukrainian and English in the header.

## Notes and limitations

- Catalog imagery uses your imported image URLs. Some third-party image links may stop working; a placeholder is shown if an image fails. Hosting images yourself in Supabase Storage can be a later enhancement.
- Public visitors can read your journal notes and ratings. Treat all saved descriptions as public.
- The interface is intended for a single owner now. Adding separate public profiles later requires changing the `entries` read policy and UI routing.
- The catalog and journal are different tables: deleting an entry does **not** delete the catalog perfume.
- Auth is provided by Supabase, not by the static site.
- The current collection query fetches the latest 1,000 journal entries. Add pagination if your personal list grows beyond that.
- If using a Supabase project on the free plan, inactivity can pause the project; check the project state if the site stops loading.

## SQL quick checks

```sql
select count(*) from public.perfumes;
select count(*) from public.entries;
select id, name, brand from public.perfumes limit 3;
```
