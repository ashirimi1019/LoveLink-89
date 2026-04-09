# LoveLink '89

A retro-futuristic AI love concierge app that helps couples plan unique date nights with a blend of generative AI, data from the real world, and a richly styled 1980s/early-90s interface. Designed and built by [ashirimi1019](https://github.com/ashirimi1019).

---

## 🏗️ Architecture Overview

LoveLink '89 is a full-stack, API-first web application with:

- **Frontend**: React.js + Tailwind CSS + Retro Synthwave UI
- **Backend**: FastAPI (Python3) with async REST, Supabase integration, Google Maps, and Ticketmaster APIs
- **Database**: Supabase Postgres (manages users, profiles, plans, restaurants, activities)
- **Authentication**: Supabase Auth + JWT for API security, with optional Google OAuth
- **Cloud API Integrations**: Google Maps (for food/places), Ticketmaster (live events), OpenAI/GenAI (optionally for suggestions)
- **State & Persistence**: User plans and profile saved both in localStorage (fallback) and database
- **CI/Dev**: .env, python-dotenv, PowerShell for local tests/debug

---

## 🌈 Tech Stack

- **Frontend**:
  - Language: JavaScript (React)
  - State: React hooks/context
  - Styling: Tailwind CSS with heavy use of custom themes, CRT/VHS effects, and pixel fonts (Press Start 2P, VT323, IBM Plex Mono, Orbitron)
  - API calls via fetch and custom service objects (`services/api.js`, `services/supabaseApi.js`)
  - Routing: `react-router-dom`

- **Backend**:
  - Language: Python 3.10+
  - Major framework: FastAPI (async, statically typed, OpenAPI auto-docs)
  - Supabase SDK: Python client for DB and user management
  - Database Access: All main entities (users, profiles, relationships, date plans, activities, restaurants, preferences) modeled in Supabase/Postgres; see [`database_schema.sql`](backend/database_schema.sql)
  - Auth: JWT tokens, passlib for password hashes, python-jose for token signing, Google Sign-In
  - External APIs:
    - **Google Maps** → places search, geocoding, and food/activity recs
    - **Ticketmaster** → live local event fetch for surprise ideas
    - **(Optionally) OpenAI/GenAI** → AI-powered tips, names, and surprise generation
    - Weather, time, etc. as "flavor text"
  - .env loading (python-dotenv)
  - CORS and security headers are aggressive, defaults to local dev safe

---

## 🗂️ Backend Services and Folder Structure

```
backend/
├── main.py             - FastAPI app factory, ties all routes/middleware together
├── routes/
│   ├── date_routes.py  - /api endpoints to generate food, activity, and surprises
│   ├── auth_routes.py  - /api/auth for register, login, Google OAuth, user profile
│   └── external_api_routes.py - endpoints for Google Maps, Ticketmaster
├── services/
│   └── providers/
│       ├── places.py   - Google Places integration (find food, attractions)
│       └── ticketmaster.py - Ticketmaster events integration
├── utils/
│   ├── supabase.py     - Supabase client setup, CRUD, auth helpers
│   ├── auth.py         - JWT, password hashing
│   ├── weather.py      - (optional) weather and pollen APIs
│   └── ...
├── requirements.txt    - All Python backend packages
├── database_schema.sql - Postgres/Supabase schema
├── test_api.py         - CLI smoke tests, dev runner
├── api_tester.py       - End-to-end API & integration test tool
└── test_supabase_connection.py - Supabase connection tester
```

---

## 🎨 Frontend Structure

```
frontend/
├── src/
│   ├── pages/                - Multi-step UI: Setup, Food, Activity, Surprise, Final Plan
│   ├── context/              - Global DatePlanContext (date state, history, persistence, user auth)
│   ├── services/             - API wrappers for backend and Supabase
│   ├── components/           - Reusable UI primitives (Retrowave cards, CRT effects, etc.)
│   └── ...
├── public/                   - Static files, wallpapers, fonts
├── tailwind.config.js        - Full custom neon/purple/CRT color themes
├── package.json              - All JS deps
├── index.html                - Loads glowy retro fonts, sets favicon/title
└── README.md                 - Thematic documentation; see [repo](frontend/README.md)
```

---

## 🗄️ Database ER Model (Supabase/Postgres)

Tables defined in [`backend/database_schema.sql`](backend/database_schema.sql):

- **users**: UUID id, username, email, hash, created/last login
- **profiles**: `user_id`, bio fields (name, birthday, hobbies, etc)
- **hobbies**: connect-to-profiles, many-to-one
- **relationships**: tie user and partner, status, anniversary
- **date_plans**: schedules, state, notes
- **activities/restaurants**: title, type, tags, details, rating
- **user_preferences**: budget, transit, dietary, etc.
- All activity/restaurant can be tagged (for searching/filter)

---

## 🔑 Authentication

- **Sign Up / Sign In**: Managed by Supabase (secure, email+password, password hash only stored in Supabase)
- **JWT / OAuth2**: Email and JWT for session routing, Python jose for serverside
- **Google Sign-In**: Optional, with token validation
- **Session & Preferences**: Both frontend and backend write to localStorage as a safety net (for offline or fallback after DB error)

---

## 🤖 Core Backend APIs

- `/api/auth/register`, `/api/auth/login` — User management, session creation
- `/api/auth/supabase-config` — FE gets Supabase API config
- `/api/generate-date` — Main endpoint, takes user setup and preferences, returns plan with: `restaurant`, `activity`, `surprise`
- `/api/events/search` — Uses Ticketmaster API for in-town events
- `/api/places/search` — Uses Google Maps Places/Geocode for dining/attraction recs
- `/api/save-date-plan` — Save a completed date night plan (JWT protected)
- `/api/debug` — Health check (`{"status": "API is running", ...}`)
- `/api/test-html` — Backend HTML test page

All major payloads are JSON in and out; OpenAPI docs auto-generated by FastAPI on `/docs` endpoint.

---

## ⚙️ Major Features

- **Step-by-step Date Planning:** Location, mood, budget, dietary needs, and transportation feed into rec system.
- **Dynamic Food & Activity Picker**: Combines user prefs with Google Maps and Ticketmaster events.
- **Retro CRT/VHS UI:** No modern UI elements; fonts, transitions, and colors are strictly 80s/90s synthwave.
- **AI Surprises:** Optional (can use OpenAI, Google GenAI, or locally seeded set if not configured).
- **Save, Share, and History:** All plans can be stored per user, and history loaded (API + local fallback).
- **Authentication:** Full JWT login flow, fallback to local if offline.
- **Extensible:** Easy to add more providers or integrate new APIs via FastAPI modular routers and providers.

---

## 🔗 Key Integrations / Environment Variables

- `.env` (both FE & BE):
  - **Supabase**: `SUPABASE_URL`, `SUPABASE_KEY`, `SUPABASE_SERVICE_KEY`
  - **Google**: `GOOGLE_API_KEY` (places, geocoding, weather)
  - **Ticketmaster**: `TICKETMASTER_API_KEY`
  - **JWT**: `JWT_SECRET_KEY`
  - **Frontend/Backend URLs** for CORS
  - (Optional) `GOOGLE_CLIENT_ID` for Google OAuth

All keys are loaded via dotenv and should NEVER be committed in source.

---

## 🧪 Local Development

**Backend:**
```bash
cd backend
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # Set all required keys
python main.py
# Visit http://localhost:8000/docs for OpenAPI
```

**Frontend:**
```bash
cd frontend
npm install
npm start
# App @ http://localhost:3000
```

---

## ⚡ Power Features & Debug

- `backend/api_tester.py`, `backend/test_api.py`: Full integration test for API and 3rd party services
- `backend/test_supabase_connection.py`: Verifies DB connection + list of tables
- Frontend can work in "demo" mode with fake/mock data if APIs unavailable

---

## 🚀 Deployment

- Deploy backend/ as a FastAPI app with Gunicorn/Uvicorn; static files served from `/static`
- Database: Supabase instance (or manual Postgres if desired, schema included)
- Environment variables configured on platform or manager
- Frontend: Build and host as static files on Netlify/Vercel or any modern host

---

## 👩‍💻 Attribution

- Original design, code, and theming by [ashirimi1019](https://github.com/ashirimi1019)
- Built for synthwave lovers and romantic hackers everywhere

---

## 📚 References

- [Supabase Python SDK](https://github.com/supabase-community/supabase-py)
- [FastAPI](https://fastapi.tiangolo.com/)
- [Google Maps Places API](https://developers.google.com/maps/documentation/places/web-service/overview)
- [Ticketmaster API](https://developer.ticketmaster.com/products-and-docs/apis/getting-started/)
