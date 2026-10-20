# Changelog — MG CommerceFoundry AI

All notable changes to this product are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versions follow [Semantic Versioning](https://semver.org/).

## [1.0.0] — 2026-10-20

First commercial release.

### Added
- **Accounts & roles**: registration with a 12-character password policy, bcrypt
  hashing, signed HTTP-only session cookies, login/logout/me, email-verification
  tokens, forgot/reset password, change password, admin role with a role-gated
  admin area (stats + user management with self-lockout protection).
- **Stores**: full CRUD with owner scoping, cascading delete limited to the
  store's own data, public store discovery, currency/language/theme/tagline.
- **Visual builder**: 7 section types with add/remove/reorder/visibility,
  live preview, browser drafts, and **server save + publish with versioning**.
- **Catalogue**: product CRUD (SKU, compare-at price, inventory, SEO fields,
  draft/published), category CRUD, public catalogue with search, category
  filter and pagination.
- **Commerce**: server-side cart, percent/fixed coupons with limits and expiry,
  checkout with server-recomputed totals and stock control, order history and
  owner order queue, status lifecycle with stock restored on cancellation.
- **AI Product Studio**: Claude product descriptions and SEO metadata from
  supplied facts (English + Bengali, four tones), AI category descriptions, a
  built-in demo generator when no API key is configured, and an AI usage log.
- **Platform**: rate limiting (global/auth/AI), Helmet headers, origin checks on
  mutations, Zod validation throughout, Docker Compose stack (MongoDB + API +
  Web) plus VPS setup, demo seed (3 accounts, demo store, 6 products,
  `WELCOME10` coupon), unit tests, and the full documentation set
  (README, HowToSetup, CLAUDE, DEMO, API reference, ERD, security notes).

### Security
- Every mutating endpoint requires authentication; admin routes additionally
  require the admin role; store-owned resources are owner-scoped and answer
  404 to other users. Action tokens (reset/verify) are stored only as SHA-256
  hashes with a 1-hour expiry. No real credentials ship in the archive —
  configuration is via `.env` copied from `.env.example`.
