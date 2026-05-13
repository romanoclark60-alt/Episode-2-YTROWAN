# Progressive Overload Coach

A single-file gym tracking app. Logs your sets, tracks bodyweight, watches your photos progress, and automatically tells you when to add weight or push for more reps.

Everything is **one HTML file** — no install, no build, no npm. Open it in a browser and start logging. Optionally add cross-device sync with a free Supabase account.

## Quick start (no sync — just use it locally)

1. Download `index.html`
2. Open it in your browser (double-click, or drag onto your browser window)
3. Start logging sets. Data saves to your browser's localStorage on this device.

## Deploy it as a website (still no sync)

Drag the `index.html` file (or the whole folder) onto [Netlify Drop](https://app.netlify.com/drop). You get a public URL in 10 seconds. Bookmark it on your phone home screen.

## Add cross-device sync (optional, 5 minutes)

If you want logging on one device to show up on the others, do this:

1. Make a free Supabase account at [supabase.com](https://supabase.com)
2. Click **New project**, give it any name, generate a password, pick a region close to you
3. Wait ~1 minute for it to provision
4. In the left sidebar: **Settings (gear) → API**
5. Copy two things:
   - **Project URL** (looks like `https://xxxxx.supabase.co`)
   - **Publishable key** (the one starting with `sb_publishable_`)
6. Open `index.html` in a text editor. Find these two lines near the bottom:

   ```js
   const SUPABASE_URL = 'PASTE-YOUR-SUPABASE-PROJECT-URL-HERE';
   const SUPABASE_KEY = 'PASTE-YOUR-SUPABASE-PUBLISHABLE-KEY-HERE';
   ```

   Replace the placeholders with your two values.

7. Back in Supabase, click **SQL Editor → New query**, paste this block, hit **Run**:

   ```sql
   create table if not exists public.app_state (
     key text primary key,
     data jsonb not null default '{}',
     updated_at timestamptz not null default now()
   );

   alter table public.app_state enable row level security;

   drop policy if exists "anon read"   on public.app_state;
   drop policy if exists "anon insert" on public.app_state;
   drop policy if exists "anon update" on public.app_state;

   create policy "anon read"   on public.app_state for select using (true);
   create policy "anon insert" on public.app_state for insert with check (true);
   create policy "anon update" on public.app_state for update using (true);

   alter publication supabase_realtime add table public.app_state;
   ```

8. Save `index.html`, redeploy (drag the folder onto Netlify Drop again).
9. Open the site on both devices. Make a change on one — it appears on the other within a second.

## Customize it

Open the file in a text editor and edit the `CONFIG` block at the top of the `<script>` tag:

- **`gyms`** — list of gyms you train at
- **`days`** — your training split (Push/Pull/Legs, Upper/Lower, whatever)
- **`splitRotation`** — order of days through the week, including rest
- **`upgradeAtReps`** — how many reps on the top set before the coach tells you to add weight
- **`defaultExercises`** — your starter exercise list

Once you log anything in the app, you can add/edit gyms, days, and exercises through the in-app buttons — the CONFIG block is just the seed.

## Notes

- Without Supabase keys, the app is **local-only**. Each device has its own copy. Data never leaves your browser.
- With Supabase keys, the app syncs to a tiny single-row JSONB blob in your own Supabase project. Anyone with your site URL **and** your publishable key embedded in it can read your gym data — so don't share the site URL publicly if that matters to you.
- Photos are compressed to ~80KB each before saving to avoid hitting localStorage limits.
