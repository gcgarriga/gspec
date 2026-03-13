# Example: Brownfield Project

A realistic example of using gspec on an existing codebase to plan a new feature.

---

**User:** gspec explore

**Agent:**

```
📋 gspec status:
No .gspec/ directory found. Starting fresh.
```

Scanning the codebase... Reading README, entry point, tracing a request through the stack...

✅ Wrote `.gspec/context.md`

```markdown
# Project Context

**Type:** brownfield
**Date:** 2025-03-12

## What this project does
E-commerce API built with Express.js and TypeScript. Handles products, orders,
users, and payments. ~15K lines of code across 120 files.

## Tech Stack
- **Language:** TypeScript 5.3
- **Runtime:** Node.js 20
- **Framework:** Express.js 4.18
- **Database:** PostgreSQL via Prisma ORM (12 models, 23 migrations)
- **Auth:** JWT with refresh tokens (custom middleware in src/middleware/auth.ts)
- **Validation:** Zod schemas co-located with routes
- **Testing:** Vitest + Supertest, ~78% coverage
- **Build:** tsup for production, tsx for dev

## Architecture
Layered monolith following service pattern:

Request → Route Handler → Validation (Zod) → Service → Prisma → PostgreSQL

- `src/routes/` — Express route handlers (products, orders, users, payments)
- `src/services/` — Business logic, one service per domain
- `src/middleware/` — Auth, validation, error handling (custom AppError class with HTTP codes)
- `src/utils/` — Shared helpers (pagination, date formatting, slug generation)

## Key directories and entry points
- Entry: `src/index.ts` — registers middleware, mounts routes, starts server
- Routes: products (CRUD), orders (create, status updates), users (register, login, profile), payments (Stripe integration)
- 47 test files in `src/__tests__/`, mirroring src/ structure

## Data Layer
Prisma with PostgreSQL. Key models: User, Product, Order, OrderItem, Payment.
Relations: User → Orders → OrderItems → Products. Payment 1:1 with Order.
Migrations in `prisma/migrations/`.

## How to build, test, and run
- Dev: `npm run dev` (tsx watch mode)
- Test: `npm test` (Vitest + Supertest, ~78% coverage, factories in `src/__tests__/factories/`)
- Build: `npm run build` (tsup)
- Deploy: Docker → fly.io (Dockerfile in root)
- CI: GitHub Actions (.github/workflows/ci.yml)

## Patterns and Principles
- Routes never access Prisma directly — business logic lives in services
- All input is validated with Zod at the route boundary before calling services
- Errors flow through the AppError + error middleware pattern
- Use camelCase for variables/functions and PascalCase for types/classes
- Follow the shared pagination pattern via `?page=&limit=`
- New features should extend the existing service-layer pattern, not introduce a new abstraction model

## Strengths and debt
**Strengths:** Consistent service pattern, good test coverage on routes, clean separation of concerns
**Debt:** No request logging/tracing, payment service is a 600-line monolith that should be split, no rate limiting, some Prisma queries in route handlers (bypassing service layer in 3 places)
```

I can also offer to generate or update `.github/copilot-instructions.md` from the patterns above once you confirm this context is accurate.

Anything missing or incorrect?

---

**User:** gspec specify — I want to add a wishlist feature. Users can save products to a wishlist, share wishlists with a link, and get notified when wishlist items go on sale.

**Agent:** Got it. Before I write the spec, let me probe some details:

1. Can users have multiple wishlists or just one?
2. Shared wishlists — view-only or can others add items?
3. Sale notifications — email, in-app, or push? (I don't see an email service in the current codebase, so email would be new infrastructure.)
4. Is there a max number of items per wishlist?
5. What defines "on sale"? A price drop from the product's original price, or a specific sale flag?
6. Should the wishlist show current prices and highlight price changes since items were added?

**User:** One wishlist per user. Shared = view only. Email notifications. No max items. "On sale" = any price drop. Yes, show price changes.

**Agent:** Note: email notifications will require adding an email service — this is new infrastructure for this project. I'll include that in scope.

✅ Wrote `.gspec/spec.md`

```markdown
# Specification: Wishlist Feature

## Overview
Users can save products to a personal wishlist, share it via a public link,
and receive email notifications when wishlist items drop in price.

## Requirements

### Wishlist CRUD
- POST /api/wishlist/items — add product to wishlist (creates wishlist if none exists)
- DELETE /api/wishlist/items/:productId — remove product from wishlist
- GET /api/wishlist — get current user's wishlist with products, current prices, and price-at-add

### Sharing
- POST /api/wishlist/share — generate or retrieve shareable link (idempotent)
- GET /api/wishlist/shared/:shareToken — view shared wishlist (public, no auth)
- DELETE /api/wishlist/share — revoke shareable link

### Notifications
- Background job: periodically check product prices against wishlist items' price-at-add
- Send email when price drops below price-at-add
- Don't re-notify for the same price drop (track last notified price)

## Constraints
- Wishlist queries should be <100ms for up to 500 items
- Email delivery should be best-effort (queue-based, don't block API)

## Out of Scope
- Multiple wishlists per user, collaborative wishlists, push/in-app notifications
- Wishlist analytics, recommendation engine

## Open Questions
- Which email provider? (SendGrid, AWS SES, Resend?)
- How often should the price check job run? (hourly? daily?)
```

---

**User:** gspec plan

**Agent:** Since this is brownfield, I'll follow the existing patterns (service layer, Prisma, Zod).

For the new email infrastructure, here are my recommendations:

**Email service options:**
1. **Resend (Recommended)** — Simple API, great DX, generous free tier (100 emails/day), TypeScript SDK
2. **SendGrid** — More mature, higher volume, but more complex setup
3. **AWS SES** — Cheapest at scale, but requires AWS account and more boilerplate

**Background job options:**
1. **BullMQ + Redis (Recommended)** — Battle-tested job queue for Node.js, supports cron schedules, retries, and monitoring
2. **node-cron** — Simpler but no persistence, retry, or monitoring. Fine for MVP.
3. **Trigger.dev** — Managed, but adds external dependency

Which email provider and job approach do you prefer?

**User:** Resend and BullMQ.

**Agent:** ✅ Wrote `.gspec/plan.md`

```markdown
# Implementation Plan: Wishlist Feature

## Architecture

Extends the existing service layer pattern:

```
Routes → Zod Validation → WishlistService → Prisma → PostgreSQL
                           NotificationService → Resend API
                           PriceCheckWorker (BullMQ) → WishlistService + NotificationService
```

New modules:
- `src/services/wishlist.ts` — wishlist CRUD + sharing logic
- `src/services/notification.ts` — email sending via Resend
- `src/routes/wishlist.ts` — API endpoints
- `src/workers/price-check.ts` — BullMQ worker for price monitoring
- `src/queues/` — queue definitions and connection

## Tech Stack Additions
- **Resend** (@resend/node) — email delivery
- **BullMQ** (bullmq) — background job processing
- **Redis** — required by BullMQ (add to docker-compose.yml)

## Data Model

New Prisma models:

Wishlist: id, userId (unique — one per user), shareToken (unique, nullable), createdAt
WishlistItem: id, wishlistId, productId, priceAtAdd (Decimal), lastNotifiedPrice (Decimal, nullable), addedAt

Relations: User 1:1 Wishlist, Wishlist 1:N WishlistItem, WishlistItem N:1 Product

## Key Interfaces

### API Endpoints
- POST /api/wishlist/items { productId } → 201 { wishlistItem }
- DELETE /api/wishlist/items/:productId → 204
- GET /api/wishlist → 200 { wishlist with items, current prices, price deltas }
- POST /api/wishlist/share → 200 { shareUrl }
- GET /api/wishlist/shared/:shareToken → 200 { wishlist } (no auth)
- DELETE /api/wishlist/share → 204

### Price Check Worker
- Runs every hour via BullMQ repeatable job
- Fetches all wishlist items, compares current product price to priceAtAdd
- For items where current price < priceAtAdd AND current price != lastNotifiedPrice: send email, update lastNotifiedPrice

## Implementation Approach
- Follow existing patterns: route → service → Prisma
- Zod schemas for request validation, consistent with other routes
- Share token: crypto.randomUUID(), stored on Wishlist model
- Email templates: simple HTML, no template engine needed for v1
- Worker runs in same process for simplicity (can extract to separate worker process later)

## Dependencies & Prerequisites
- Redis must be running locally (add to docker-compose.yml)
- Resend API key in environment variables
- New Prisma migration for Wishlist and WishlistItem models

## Risks
- **Redis as new dependency** — adds operational complexity. Mitigate: add to existing Docker Compose, document setup.
- **Email deliverability** — Resend handles this, but need to verify sender domain.
- **Price check at scale** — hourly check across all wishlists could be slow with many users. Mitigate: batch queries, paginate, and add index on WishlistItem.productId.
```

---

**Agent:** gspec is done. You now have:

- `.gspec/context.md` — codebase understanding and coding patterns
- `.gspec/spec.md` — feature requirements and scope
- `.gspec/plan.md` — implementation decisions and tech choices

Now hand off to Copilot's native planning/implementation:

```text
Implement the feature in .gspec/plan.md following the patterns in .gspec/context.md
```

Or provide the artifacts explicitly:

```text
@.gspec/context.md @.gspec/spec.md @.gspec/plan.md
Implement the wishlist feature.
```
