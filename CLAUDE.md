# CLAUDE.md — guide for AI coding tools (and new developers)

Product: **MG CommerceFoundry AI** — self-hosted e-commerce builder with AI product
descriptions. This file explains how the code is organised so an AI assistant (Claude
Code, Cursor, Copilot…) or a buyer's developer can extend it safely.

## Architecture at a glance

- **Monorepo** (npm workspaces) inside `Node Project/`:
  - `apps/web` — Next.js 15 (App Router) + React 19 + Tailwind CSS 3. A single-page
    dashboard (`src/components/CommerceApp.tsx`) composes seven panels.
  - `apps/api` — Express 4 + TypeScript + Mongoose (MongoDB 7). Entry: `src/server.ts`.
  - `packages/shared` — `@commercefoundry/shared`: shared TS types (builder sections,
    products, orders, AI input/output). Build this package **before** the apps.
- **Data**: MongoDB collections — users, stores, categories, products, storepages,
  carts, coupons, orders, aiusages. Models: `apps/api/src/models.ts`. Schema reference:
  `database/schema.sql` / `database/schema.mongo.js`; ERD: `docs/ERD.md`.
- **Auth**: bcrypt (cost 12) + JWT in a signed, HTTP-only, SameSite=Lax cookie
  (`cf_session`, 7 days). Guards live in `apps/api/src/auth.ts`.

## Folder map (API)

```
apps/api/src/
├── server.ts            # middleware (helmet, CORS, origin check, rate limits), router wiring, error handler
├── config.ts            # zod-validated env; loads .env from apps/api, "Node Project" root, or submission root
├── models.ts            # all Mongoose schemas + indexes
├── auth.ts              # password hashing, session cookie, requireAuth, requireAdmin, getAccessibleStore, slugify
├── ai.ts                # Claude call + built-in deterministic demo generator (generateDemoDescription)
├── seed.ts              # idempotent demo seed (npm run seed)
└── routes/
    ├── authRoutes.ts    # register/login/logout/me, verify-email, forgot/reset/change password
    ├── storeRoutes.ts   # store CRUD (delete cascades to that store only)
    ├── productRoutes.ts # product CRUD + public catalogue (search/filter/pagination)
    ├── catalogRoutes.ts # categories, coupons (+ public /validate), builder pages
    ├── commerceRoutes.ts# cart + checkout + orders (status changes restore stock on cancel)
    └── adminAiRoutes.ts # /api/admin (stats, users) + /api/ai (product & category descriptions)
```

Web components mirror the domains: `AccountPanel`, `StoreManager`, `ProductManager`
(includes the AI Studio form), `BuilderStudio`, `Storefront` (catalogue + cart +
checkout), `OrdersPanel`, `AdminPanel`. All API calls go through `src/lib/api.ts`
(`credentials: "include"`, normalised `id`/`_id`).

## Conventions — follow these when editing

1. **Security is not optional.** Every mutating route MUST chain `requireAuth`
   (plus `requireAdmin` for `/api/admin/*`). Any route touching a store-owned
   resource MUST resolve ownership with `getAccessibleStore(req, res, storeId)`
   and answer **404** (not 403) when the store isn't the caller's. Never trust
   prices, totals or stock from the client — recompute server-side (see checkout).
2. **Validate everything with zod** at the route boundary; reuse the schemas in
   each route file. Keep the shared error handler's response shapes:
   `{ data }` on success, `{ error }` / `{ error, details }` on failure.
3. **Money**: round with `roundMoney()` (auth.ts). Slugs: `slugify()`; unique per
   store (compound indexes already exist).
4. **Types first**: add cross-app types to `packages/shared/src/index.ts` and
   rebuild shared before typechecking the apps.
5. **No secrets in code or commits.** Config only via `.env` (see `.env.example`);
   the API key for Claude is server-side only.
6. **No giant files.** Keep routes in their domain file; new domains get a new
   file under `routes/` and one wiring line in `server.ts`.

## How to run tests and checks

```bash
cd "Node Project"
npm run build        # shared → api → web (production build)
npm run typecheck    # builds shared, then typechecks all three packages
npm test             # node:test unit tests in apps/api/test (slugify, AI demo generator EN/BN)
npm run seed         # (re)creates demo data — idempotent
```

Manual smoke test: seed → sign in as each demo account (see `DEMO.md`) → add a
product → publish → add to cart as the customer → apply `WELCOME10` → check out →
as the owner, mark the order paid. Negative checks: creating a product while
signed out returns 401; `/api/admin/stats` as a normal user returns 403;
creating a product in someone else's store returns 404.

## Extending the product (common requests)

- **New builder section type**: extend `SectionType` in shared, the zod enum in
  `catalogRoutes.ts`, and the render switch in `BuilderStudio.tsx`.
- **Real payments**: `orders.paymentMethod` is intentionally demo-only
  (`demo-card` / `cash-on-delivery`). Add a provider adapter beside
  `commerceRoutes.ts`, verify webhooks, and only then mark orders `paid`.
- **Email delivery**: password-reset and verification tokens are returned in API
  responses on non-production installs (`exposeActionTokens` in `config.ts`) so
  the demo works without a mail server. For production, send the same tokens by
  email from `authRoutes.ts` and keep `EXPOSE_ACTION_TOKENS=false`.
- **File uploads**: product images are URL-based by design. If you add uploads,
  store them outside MongoDB, scan them, and keep the URL validation.
