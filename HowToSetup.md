# HowToSetup — MG CommerceFoundry AI

Install the product from scratch on a clean machine. Two targets are covered:
**Target A — Docker (fastest)** and **Target B — VPS / manual Node.js**.
Both end with the same seeded demo, the same first-admin login and the same
branding steps.

## 0. Prerequisites

| Target | You need |
|---|---|
| A — Docker | Docker Engine / Docker Desktop with Compose v2 |
| B — VPS | Node.js **20 or newer**, npm 10+, and MongoDB 7 (installed locally, in Docker, or a managed URI such as MongoDB Atlas) |
| Both | This archive extracted; a terminal; (optional) an Anthropic API key for live AI copy |

No real credentials ship in this archive. All configuration happens through
`.env`, copied from `.env.example` (placeholder values only).

---

## Target A — Docker (full stack)

```bash
# 1. From the extracted root (the folder containing docker-compose.yml):
cp .env.example .env

# 2. Edit .env and set a unique SESSION_SECRET (32+ characters), e.g.:
#    node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
#    Optionally set ANTHROPIC_API_KEY for live AI generation.

# 3. Build and start MongoDB + API + Web:
docker compose up --build -d

# 4. Create the demo data (accounts, demo store, products, coupon):
docker compose exec api npm run seed -w @commercefoundry/api

# 5. Open the app:
#    Web  → http://localhost:3000
#    API  → http://localhost:4000/health   ({"status":"ok"})  ·  /ready
```

Stop with `docker compose down` (add `-v` to also delete the database volume).

---

## Target B — VPS / manual Node.js

```bash
# 1. Start MongoDB 7 (any one of these):
docker compose up -d mongodb        # uses this archive's compose file, MongoDB only
# — or use your own MongoDB / Atlas connection string in MONGODB_URI below.

# 2. Configure environment (from the extracted root):
cp .env.example .env                # set SESSION_SECRET (32+ random chars),
                                    # MONGODB_URI and WEB_ORIGIN for your server

# 3. Install, build, seed (from the "Node Project" folder):
cd "Node Project"
npm ci                              # restores the pinned lockfile exactly
npm run build                       # builds shared → API → Web
npm run seed                        # demo data (idempotent, safe to re-run)

# 4. Run (development):
npm run dev:api                     # terminal 1 — API on :4000
npm run dev:web                     # terminal 2 — Web on :3000

# 4b. Run (production):
npm run start -w @commercefoundry/api     # serves apps/api/dist/server.js
npm run start -w @commercefoundry/web     # next start on :3000
```

**Production checklist:** put both apps behind HTTPS (e.g. Nginx/Caddy reverse
proxy), set `NODE_ENV=production`, a unique `SESSION_SECRET`, `WEB_ORIGIN` to
the exact public web origin, `TRUST_PROXY=true` only behind a trusted proxy,
keep MongoDB on a private network, and schedule backups of the database volume.

**Verify the install:** `curl http://localhost:4000/ready` returns
`{"status":"ready"}` when the database is connected; `npm run typecheck` and
`npm test` (from `Node Project`) should both pass.

---

## Database setup (what the seed creates)

- Schema: Mongoose creates collections and indexes automatically on first
  run. To provision explicitly: `mongosh "<MONGODB_URI>" database/schema.mongo.js`
  (relational reference: `database/schema.sql`).
- Demo data: `npm run seed` (or `database/seed.mongo.js` via mongosh;
  reference: `database/seed.sql`) creates the accounts below, the **Foundry
  Demo Store**, 4 categories, 6 published products, a published builder home
  page and the **WELCOME10** coupon (10% off).

## First admin login

| Role | Email | Password |
|---|---|---|
| **Admin** | `admin@commercefoundry.demo` | `AdminDemo1234` |
| Store owner | `demo@commercefoundry.demo` | `DemoOwner1234` |
| Customer | `customer@commercefoundry.demo` | `CustomerDemo1234` |

These are **published demo credentials** for evaluation. On a real
installation: sign in as admin, register your own account, promote it in the
Admin panel, then deactivate the three demo accounts (Admin → Deactivate).

To make any registered user an admin later: sign in as an existing admin →
**Admin** section → *Make admin*. (There is intentionally no public way to
self-promote.)

## Branding changes

| Change | Where |
|---|---|
| Product name in the UI header/footer | `Node Project/apps/web/src/components/CommerceApp.tsx` (search for `CommerceFoundry`) |
| Page title / meta description | `Node Project/apps/web/src/app/layout.tsx` |
| Colours (ink / paper / accent) and shadows | `Node Project/apps/web/tailwind.config.ts` + `src/app/globals.css` |
| Demo store name, products, coupon | `Node Project/apps/api/src/seed.ts`, then re-run `npm run seed`; or edit live in the Store/Product managers |
| Store name, tagline, theme, currency, language | In the app: **Store Manager → Edit store** (per store, no code change) |
| Session cookie / issuer names (advanced) | `Node Project/apps/api/src/auth.ts` (`cf_session`, `commercefoundry-api`) |

## AI configuration

- **No key (default):** the AI Studio uses the built-in demo generator —
  deterministic copy from the facts you enter, clearly labelled "Demo mode".
- **Live Claude:** set `ANTHROPIC_API_KEY` (and optionally `ANTHROPIC_MODEL`)
  in `.env` and restart the API. The key is only ever used server-side.
- Set `AI_DEMO_FALLBACK=false` to require a real key in production.

## Troubleshooting

| Symptom | Fix |
|---|---|
| API exits with "Invalid environment configuration" | Your `.env` is missing or `SESSION_SECRET` is under 32 chars — compare with `.env.example`. |
| API exits in production about `SESSION_SECRET` | You left the placeholder value; set a unique random secret. |
| Web says "The API is not reachable yet" | Start MongoDB, then the API; check `NEXT_PUBLIC_API_URL` (default `http://localhost:4000`). |
| Login works but actions return 403 "origin is not allowed" | `WEB_ORIGIN` must exactly match the web address in the browser (scheme + host + port). |
| Empty storefront | Run `npm run seed`, and make sure products are **Published** (drafts never appear publicly). |
| Port already in use | Change `API_PORT` / the web port (`npm run dev -w @commercefoundry/web` uses 3000) or stop the other process. |
