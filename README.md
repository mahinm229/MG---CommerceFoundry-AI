# MG · CommerceFoundry AI

**AI-powered, self-hosted e-commerce software** — a visual store builder, complete commerce
operations (products, categories, carts, coupons, orders), store accounts with role-based
access, and Claude AI product descriptions in English and Bengali. Built with **Next.js, React,
Tailwind CSS, Express, TypeScript and MongoDB**. Docker supports full local deployment.

> One-line pitch: *Your store. Your style. AI at your side.* — run your whole shop on your own
> infrastructure, with source code you own.

![Version](https://img.shields.io/badge/version-1.0.0-limegreen) ![Node](https://img.shields.io/badge/node-%3E%3D20-green) ![License](https://img.shields.io/badge/license-commercial-blue)

---

## Who it's for

- **Independent retailers and D2C brands** who want a real store without monthly SaaS lock-in.
- **Agencies and freelancers** who deliver stores to clients and need white-label, self-hosted code.
- **Developers** who want a clean, typed, tested commerce foundation to extend (see `CLAUDE.md`).

## Feature list (everything in this archive is functional)

| Area | What's included |
|---|---|
| **Accounts & auth** | Email/password registration (12+ char policy), bcrypt hashing, signed HTTP-only session cookie (7 days), login/logout, current-user endpoint, **email verification tokens**, **forgot/reset password**, change password, account deactivation by admin |
| **Roles & security** | `user` / `admin` roles; **every mutating endpoint requires sign-in**; admin area is role-gated (403 test included); owner-scoping everywhere (another user's records return 404); origin check on mutations, Helmet headers, rate limits (global, auth, AI), Zod validation, parameterised Mongoose queries, scoped deletes |
| **Stores** | Create / edit / delete stores (delete cascades only to that store's own data), slug, currency, language (EN/BN), 6 themes, tagline; public store discovery for the storefront |
| **Visual builder** | 7 section types (hero, featured products, category grid, promo, testimonials, newsletter, rich text), add/remove/reorder/show-hide, live preview, per-section settings, **browser drafts + server save + publish with version numbers** |
| **Products** | Full CRUD, slug, SKU, compare-at price, inventory, draft/published, image URLs, features, SEO title/meta/keywords, public catalogue with **search, category filter and pagination** |
| **Categories** | Full CRUD per store |
| **Coupons** | Percent or fixed discounts, minimum subtotal, usage caps, expiry, public validate endpoint, usage counted at checkout |
| **Cart & checkout** | Server-side cart per user/store (prices and stock always re-checked server-side), quantity controls, checkout with shipping address, demo-card or cash-on-delivery, stock decrement, coupon application, order confirmation |
| **Orders** | Customer order history, store-owner order queue, order detail with access control, status lifecycle (pending → paid → shipped → delivered / cancelled / refunded) with **stock restored on cancellation** |
| **AI Product Studio** | Claude-powered product descriptions + SEO metadata from supplied facts only (EN + BN, 4 tones), **plus a built-in demo generator** so the feature works with no API key; AI category descriptions; every call logged to `ai_usage`; output always requires human review |
| **Admin** | Platform stats (users, stores, products, orders, AI calls, revenue), user list, promote/demote role, activate/deactivate (with self-lockout protection) |
| **Demo data** | Seed script creates 3 demo accounts, the *Foundry Demo Store*, 4 categories, 6 published products, a published builder home page and the `WELCOME10` coupon — see `DEMO.md` |
| **Deployment** | Docker Compose stack (MongoDB + API + Web) and a manual VPS path — both fully documented in `HowToSetup.md` |

## Screenshots

See [`screenshots/README.md`](screenshots/README.md) for a screen-by-screen tour of the product
(home/builder, account, store manager, product manager + AI Studio, storefront + cart/checkout,
orders, admin) and how to capture each screen from the seeded demo.

## Quick start

```bash
cp .env.example .env          # set SESSION_SECRET (32+ random characters)
cd "Node Project"
npm install
npm run build
# in three terminals, from "Node Project" (MongoDB via the root docker-compose, or your own):
npm run seed                  # demo data
npm run dev:api               # API  → http://localhost:4000/health
npm run dev:web               # Web  → http://localhost:3000
```

Full instructions — **Docker and VPS targets, database setup, first admin login, branding
changes** — are in **[HowToSetup.md](HowToSetup.md)**. Demo accounts and the reviewer
walkthrough are in **[DEMO.md](DEMO.md)**.

## Project layout

```
MG-CommerceFoundry-AI/
├── README.md  HowToSetup.md  CLAUDE.md  CHANGELOG.md  LICENSE-NOTE.md  DEMO.md
├── .env.example               # every variable, placeholder values only
├── docker-compose.yml         # MongoDB + API + Web
├── Node Project/              # the application (npm workspaces monorepo)
│   ├── apps/api/              # Express + TypeScript API (routes/, models, auth, AI, seed, tests)
│   ├── apps/web/              # Next.js + React + Tailwind web app (components/)
│   ├── packages/shared/       # shared TypeScript types (@commercefoundry/shared)
│   └── package-lock.json      # pinned dependencies
├── database/                  # schema.sql + seed.sql, schema.mongo.js + seed.mongo.js
└── docs/                      # API.md · ARCHITECTURE.md · ERD.md · SECURITY.md
```

## Verification (performed on v1.0.0 before release)

- `npm run typecheck` — passes for shared, API and web.
- `npm run build` — production build passes (Next.js + API + shared).
- `npm test` — unit tests for slug handling and the AI demo generator (EN + BN) pass.
- End-to-end smoke test against a MongoDB-compatible database: seed → login (admin/owner/
  customer) → public catalogue → cart → `WELCOME10` coupon → checkout (total re-checked
  server-side) → owner changes order status → AI description generation → admin stats;
  negative tests confirmed: unauthenticated mutation = 401, non-admin admin access = 403,
  cross-owner access = 404, weak password = 400, foreign `Origin` = 403.

## Licence

Commercial licence — see **[LICENSE-NOTE.md](LICENSE-NOTE.md)**. In short: one purchase covers
one production installation; you may modify the code for that installation; redistribution or
resale of the source is not permitted.

## Support

Sold and supported by **MG Digital Agency** (Sylhet, Bangladesh) via the marketplace listing
where you purchased this product. Built with AI assistance and human review.
