# DEMO — reviewer & buyer walkthrough

Use this file for the **Working live demo** step of your listing: deploy the
product (see `HowToSetup.md`, Target A — Docker), run the seed, enter the demo
URL on the listing, and provide the credentials below.

## Demo credentials (created by `npm run seed`)

| Role | Email | Password | What to try |
|---|---|---|---|
| **Admin** | `admin@commercefoundry.demo` | `AdminDemo1234` | Admin panel: platform stats, promote/deactivate users |
| **Store owner** | `demo@commercefoundry.demo` | `DemoOwner1234` | Owns the *Foundry Demo Store*: edit store, products, builder, orders |
| **Customer** | `customer@commercefoundry.demo` | `CustomerDemo1234` | Cart, `WELCOME10` coupon, checkout, order history |

Demo coupon: **WELCOME10** — 10% off, no minimum, created by the seed.

## 5-minute walkthrough

1. **Storefront (no login needed):** open the demo URL → scroll to *Shop the
   demo store* → the seeded store is pre-selected with 6 published products →
   search and filter by category.
2. **Customer flow:** sign in as the customer → *Add to cart* ×2 → apply
   **WELCOME10** (total drops 10%) → fill the checkout form → *Place order* →
   the order appears under **My orders**; stock decreased by the quantity bought.
3. **Owner flow:** sign in as the store owner → **Store orders** shows the new
   order → set its status to *paid*, then *shipped*. In **Products**, edit a
   product's price/stock; in the **Builder**, reorder a section and *Publish* —
   the version number increases.
4. **AI Studio:** as the owner, open **AI Product Studio** → enter a product
   name, category and a few factual features → *Generate copy* → review the
   draft (works with no API key via the built-in demo generator; with an
   Anthropic key it uses Claude) → *Apply to product form* → save as draft.
5. **Admin flow:** sign in as admin → platform stats (users/stores/products/
   orders/revenue) and the user table → promote the customer to admin, then
   demote them back. Try the same URL as the customer first: the API returns
   **403** and the panel explains the role gate.

## Things worth noticing (audit points)

- Sign out, then try to create a product via the UI — you can't; via the API it
  returns **401**. Creating a product in a store you don't own returns **404**.
- Register with a weak password (e.g. `weak`) — rejected with the policy
  message. Register properly and the response includes an email-verification
  token on demo installs (no mail server configured); the **Verify** tab in the
  Account panel consumes it. **Forgot password** works the same way.
- All prices/totals at checkout are recomputed on the server; editing the
  request in dev tools cannot change what you pay.

## Resetting the demo

Re-run `npm run seed` at any time — it is idempotent and restores the demo
accounts, store, products, page and coupon (it does not delete orders placed
during evaluation; delete the database volume for a completely clean slate).
