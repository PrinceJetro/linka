# Linka — AI-Powered African Partnership Network (MVP)

> **One-line pitch:** Discover the right people, businesses, resources, and opportunities
> across Africa — turning fragmented capabilities into connected partnerships.

## Run it (2 terminals)

**Backend — Django + DRF + JWT** (`http://127.0.0.1:8000`)

```powershell
cd backend
pip install -r requirements.txt
cd linka
python manage.py migrate
python seed_demo.py        # demo data (demo/demo12345, ~18 profiles)
python manage.py test      # 29 smoke tests (auth, profiles, matcher, requests, messaging)
python manage.py runserver 8000
```

**Frontend — Next.js** (`http://localhost:3000`)

```powershell
cd frontend
npm install
npm run dev
```

Set `NEXT_PUBLIC_API_URL=http://127.0.0.1:8000/api` in `frontend/.env.local`
if the API runs elsewhere (defaults to the URL above).

## AI (Gemini) — optional, graceful fallback

Without a key the matcher uses built-in intent detection and the brief uses
templates — everything works offline.

```powershell
$env:GEMINI_API_KEY = "your-key-here"   # + optional GEMINI_MODEL (default gemini-2.5-flash)
python manage.py runserver 8000
```

Or put `GEMINI_API_KEY=your-key-here` in `backend/linka/.env` (gitignored,
auto-loaded on startup — no extra package needed).

With a key: `POST /api/match/` extracts intent via Gemini (response has
`"ai": true` and the UI shows ✨ AI-understood chips); `POST /api/brief/`
generates the opportunity/benefits/verify sections via Gemini.

Rate limits: the free tier throttles rapid calls. The backend retries 429s with
backoff (5s, 15s), caches identical intent queries for 1h, and returns a clear
`429 wait ~30s` on voice when the quota is exhausted. If you hit limits often,
set `GEMINI_MODEL=gemini-2.5-flash-lite` (higher free-tier RPM).

## API cheat sheet

| Method | Endpoint | Auth | What |
|---|---|---|---|
| POST | `/api/auth/register/` | – | Register + auto-seeds first capability profile, returns JWT |
| POST | `/api/auth/login/` | – | Returns access + refresh JWT |
| POST | `/api/auth/refresh/` | – | Rotate access token |
| GET/PATCH | `/api/auth/me/` | ✅ | Current user |
| POST | `/api/auth/change_password/` | ✅ | Change password (old + new, min 8) |
| GET/POST | `/api/profiles/` | POST needs ✅ | List (paginated, `?country=&industry=&search=&mine=1`) / create |
| POST | `/api/profiles/:id/request_verification/` | ✅ owner | Request verification badge (approved in `/admin/`) |
| GET | `/api/profiles/map/` | – | Map counts by country (+ `?industry=` sector filter) |
| POST | `/api/match/` | optional | Matcher: `{query, country?, industry?, partnership_type?}` → scores + reasons + `intent` + `ai` flag (own profiles excluded when logged in; Gemini intent cached 1h) |
| GET | `/api/match/history/` | ✅ | Your recent searches (re-run from UI) |
| POST | `/api/brief/` | – | Partnership brief for two profile IDs |
| GET/POST | `/api/requests/` | ✅ | Inbox (sent + received, with profile names) / send request |
| POST | `/api/requests/:id/accept\|decline\|request_info/` | ✅ | Respond to a request |
| GET | `/api/metrics/` | – | Deck metrics: profiles, searches, avg top score, conversion % |
| GET | `/api/notifications/` | ✅ | Activity feed (received requests + updates on sent) |
| GET/POST | `/api/conversations/` | ✅ | Thread list (with unread counts) / start-or-reopen 1:1 thread |
| GET/POST | `/api/conversations/:id/messages/` | ✅ | Read (marks read) / send message |
| GET/POST | `/api/profiles/milestones/?profile=:id` | POST needs ✅ | Proof-of-capability timeline (owner posts, admin verifies) |
| GET/POST | `/api/profiles/broadcasts/` | POST needs ✅ | Intent wall (open broadcasts + supplier match counts) |
| GET/POST | `/api/endorsements/?profile=:id` | ✅ | Verified reviews on accepted deals (1–5, one per reviewer) |
| POST | `/api/mou/` | ✅ party | Draft MOU markdown for accepted requests (Gemini, else template) |
| POST | `/api/trade-info/` | – | AfCFTA lane: tariffs, certifications, payments (Gemini-enriched) |
| POST | `/api/voice-intent/` | – | Voice-note matcher: Gemini transcription → intent → matches |

## MVP coverage

1. **Capability profiles** — model, CRUD, target countries, verification request flow + admin approval
2. **AI matcher** — Gemini intent extraction when keyed, rule-based fallback; "understood" chips in UI
3. **Opportunity map** — Leaflet + OpenStreetMap live map with sector filter + counts
4. **Partnership requests** — full lifecycle with names, notifications bell
5. **Partnership brief** — Gemini-generated when keyed, template fallback

Plus: JWT auto-refresh, direct messaging, full `/profiles/[id]` pages with timelines +
badges, intent wall, MOU drafts, AfCFTA lane helper, voice-note + multilingual
matching, EN/FR/SW/PT interface localization, metrics strip ("we measure"),
36 tests, paginated lists, strict `tsc`.

## Production checklist

- Set `SECRET_KEY`, `DEBUG=0`, `ALLOWED_HOSTS`, `CORS_ALLOWED_ORIGINS`, `GEMINI_API_KEY` env vars

## Deploy to Vercel (two projects, one repo)

**0. Postgres first (sqlite can't persist serverless).**
Create a free Postgres at [Neon](https://neon.tech) (or Supabase) and copy the
connection string (`postgres://user:pass@host/db?sslmode=require`).

**1. Backend project** — Import the repo → set **Root Directory** to
`backend/linka` (this folder holds `api/index.py`, `vercel.json`, and its own
`requirements.txt` — keep it in sync with `backend/requirements.txt`).
Build runs `collectstatic` + `migrate` automatically. Env vars:

| Var | Value |
|---|---|
| `DATABASE_URL` | Neon/Supabase connection string |
| `SECRET_KEY` | long random string |
| `DEBUG` | `0` |
| `ALLOWED_HOSTS` | `your-api.vercel.app,.vercel.app` |
| `CORS_ALLOWED_ORIGINS` | `https://your-frontend.vercel.app` |
| `GEMINI_API_KEY` | your key (optional; offline fallback otherwise) |

Then point your **local** backend at prod once to seed + create admin:

```powershell
$env:DATABASE_URL = "postgres://..."
python manage.py migrate
python seed_demo.py
python manage.py createsuperuser
```

Remove `DATABASE_URL` from your shell afterwards to go back to sqlite.

**2. Frontend project** — Import the same repo → **Root Directory** `frontend`
(Next.js preset, defaults work). Env var:

| Var | Value |
|---|---|
| `NEXT_PUBLIC_API_URL` | `https://your-api.vercel.app/api` |

Redeploy after changing env vars. Both tiers run fine on Vercel's free plan;
watch Gemini free-tier RPM (see rate-limit note above).
"# linka" 
