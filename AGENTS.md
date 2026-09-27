# SehatIn — Agent Guide

## Stack
- **Frontend:** React 19 + Vite 8, JSX (no TypeScript), Tailwind CSS v4
- **Backend:** Supabase, 2 Deno Edge Functions (not Node.js)
- **Package manager:** npm
- **Language:** Indonesian (UI text, variable names, comments, DB tables)

## Commands
| Command | Purpose |
|---|---|
| `npm run dev` | Vite dev server |
| `npm run build` | Production build |
| `npm run lint` | ESLint flat config (`eslint.config.js`) |
| `npm run preview` | Preview production build |
| `npm run supabase` | Pass-through to `supabase` CLI |
| `npx supabase functions serve` | Run Edge Functions locally |
| `npx supabase functions deploy rule-engine` | Deploy rule-engine |
| `npx supabase functions deploy gemini-proxy` | Deploy gemini-proxy |
| `npx supabase secrets set GROQ_API_KEY=<key>` | Set Groq key on hosted |

Run **lint** before committing. No tests or typecheck commands exist.

## Routes (`src/App.jsx`)
| Path | Component | Purpose |
|---|---|---|
| `/` | Splash | Landing with logo & "Mulai" button |
| `/login` | Login | Email/password, validates `@gmail.com` |
| `/register` | Register | Name/email/password + upserts `profiles` |
| `/dashboard` | Dashboard | Greeting + "Mulai Asesmen Kesehatan" card |
| `/asesmen` | Asesmenn | **Local** BMR/TDEE/macro calculator (no API calls) |
| `/muscle-scan` | MuscleScan | Interactive body avatar + exercise library |
| `/profile` | Profile | View height/weight/age from `profiles` table |

All auth components use `supabase.auth.signInWithPassword()` / `signUp()` with `options.data.name`.

`Navbar` (bottom nav, 3 items) appears on Dashboard, Asesmenn, MuscleScan, Profile.

## Data flow (target architecture — backend supports it, frontend not yet wired)
1. `Asesmenn.jsx` should call `supabase.functions.invoke('rule-engine', ...)` with form data + auth header
2. `rule-engine` calculates TDEE/BMI/macros, saves to `macro_targets` (deactivates prior), upserts `profiles`, returns all `food_catalog` items
3. Frontend calls `gemini-proxy` with `macro_target` + `food_catalog` → **Groq Llama 3.3** returns structured meal recommendations
4. Backend tables + views ready for saving recs and auto-summing daily totals

Currently `Asesmenn.jsx` does local math only — no Edge Function calls.

## Tables
Created via **Supabase Dashboard SQL Editor** (except `sql/001_add_name_to_profiles.sql`):
- `profiles` — gender (enum), weight_kg, height_cm, age, goal (enum: bulking/cutting/maintain); column names are **English**
- `macro_targets` — tdee, calorie_target, bmi, protein_g, protein_per_kg, carbs_g, fat_g, fiber_g, sugar_max_g, water_liter; partial unique index on `is_active` per profile
- `food_catalog` — 50 seed foods per 100g (food_name, category, kalori, protein_g, carbs_g, lemak_g, serat_g); readable by authenticated users, write only by service_role
- `food_recommendations` — daily recs linked to macro_target & food_catalog
- `daily_food_totals` — view summing totals per profile per day

## Edge Functions (`backend/supabase/functions/`)
- `verify_jwt = false` in config.toml, but `rule-engine` manually verifies auth via Bearer token + `supabase.auth.getUser()`
- `SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY` auto-injected by runtime (do NOT set)
- `GROQ_API_KEY` must be set as a secret (or in `backend/supabase/.env` for local)
- `rule-engine` imports supabase client from `https://esm.sh/@supabase/supabase-js@2` (deno.json import map unused)
- `gemini-proxy` calls Groq REST API directly (no supabase client)
- `.npmrc` files in function dirs are placeholders (ignored by Deno)
- Deno VSCode settings in `backend/.vscode/settings.json` — enables `deno.enablePaths: ["supabase/functions"]`

## Environment
| Variable | Where | Notes |
|---|---|---|
| `VITE_SUPABASE_URL` | Root `.env` | Frontend Supabase client |
| `VITE_SUPABASE_ANON_KEY` | Root `.env` | Frontend Supabase client |
| `GROQ_API_KEY` | `backend/supabase/.env` (local) / secret (hosted) | For gemini-proxy |

`.env` files are gitignored; `VITE_SUPABASE_URL`/`VITE_SUPABASE_ANON_KEY` must exist locally.

## Caveats
- Root `supabase/` is CLI artifact (`.temp/` only); real project is `backend/supabase/`
- `src/App.css` is unused Vite scaffold (not imported)
- Tailwind v4 via `@tailwindcss/vite` plugin — no PostCSS config or `tailwind.config.js`
- `config.toml:71` references `./seed.sql` — ignored, local Docker not used
- `Asesmenn.jsx` filename has double-n (`Asesmenn` not `Asesmen`)
