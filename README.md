# وصّل — WASSEL (Phase 1: auth + RBAC + foundations)

## Run
Requires Node 20+ and PostgreSQL.
```
cp .env.example .env   # edit DATABASE_URL
npm install
npm run db:setup       # migrate + append-only triggers + seed
npm run dev            # http://localhost:3000
```
## Demo accounts (public demo data — delete before production)
Password for all: `Demo#12345`

| Role | Phone |
|---|---|
| OWNER | +22210000001 |
| ADMIN | +22210000002 |
| SELLER | +22210000003 |
| DELIVERY_AGENT | +22210000004 |
| CUSTOMER | +22210000005 |

## Test checklist
1. Login with each account: lands on its own page (/owner, /staff, /seller, /agent, /account).
2. Logged in as seller, open /owner: redirected to /unauthorized.
3. Wrong password 5 times: account locked for 15 minutes.
4. Logged out, open /owner: redirected to /login.
5. `npx prisma studio`: AuditLog has login events. In psql, `UPDATE "AuditLog" SET action='x';` must fail (append-only).
6. `npm run typecheck`.

## Storefront (Phase 2)
Open http://localhost:3000 : home, /search (filters, sort, pagination), /product/books-1, /store/maktabat-chinguitt, city switcher in header, /login.

## Workers (real, no fake data)
Login as OWNER, open /owner then العمال (/owner/workers). Add a worker: a one-time password is shown once. Record income/payment/expense per worker; corrections use a reversal with a reason. Workers list starts EMPTY by design.
After pulling this update run: npm run db:migrate && npm run db:triggers

## Seller applications + settings
Public: /become-seller (form + optional GPS pin). Owner: /owner/applications (approve creates seller account + store; one-time password shown), /owner/settings (company info incl. WhatsApp, cities, zones). Run: npm run db:migrate

## Security (Phase 1 hardening)
Set APP_SECRET in .env (see .env.example). Run npm run db:migrate. New accounts created by Owner use a temp password and must change it at first login (/change-password). 2FA setup: /security (TOTP + 8 one-time recovery codes). Login asks for the code when 2FA is on.

## Storefront + operations (ported from the preview)
After pulling: `npm run db:migrate` then `npm run dev`.
Public: / (hero + sections), /sections, /section/[slug], /search, /product/[slug], /cart, /checkout (guest checkout, cash on delivery), /order/[token] (confirmation + tracking), /about /terms /privacy (text written by the Owner), /contact, /support, /become-seller, theme toggle in the menu.
Owner (/owner/settings hub): orders (status changes are logged), products, sections (with images), tickets, pages, company+delivery fees+cities, notifications, profile, appearance.
Prices and totals are always recomputed on the server; stock is reserved atomically at checkout.
IMPORTANT: images use STORAGE_DRIVER=local (files in ./storage). On serverless hosts (e.g. Vercel) local disk is NOT persistent: use a VPS/Docker host or add an S3-compatible StorageProvider before deploying.
Delivery fee defaults 50 (same city) / 100 (between cities) are provisional until you set them in Settings.

## Not built yet
Forced password change on first login, worker disable/enable (needs strong auth),
Cart + checkout (Phase 3), search autocomplete, TOTP 2FA, ratings display,
2FA enrollment/verification, re-auth form, user-management UI (Owner-protection guards in `src/lib/owner-guard.ts` are ready and fail closed), ledger posting service, fee resolver, file upload route.
