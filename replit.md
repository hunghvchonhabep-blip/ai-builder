# AI Website Builder

## Overview

Vietnamese-friendly tool that turns a short text prompt into a full one-page website using AI. Inspired by tempi.vn. Users describe their business and instantly get a beautiful landing page they can preview, edit, refine with AI, and download (HTML/PHP/Node.js/WordPress). Includes a member system with points, top-up flow, and an affiliate (CTV) program.

## Stack

- **Monorepo**: pnpm workspaces (TypeScript 5.9, Node.js 24)
- **Frontend**: React + Vite (`artifacts/ai-builder`) — wouter, TanStack Query, framer-motion, shadcn/ui
- **Backend**: Express 5 (`artifacts/api-server`) with OpenAI SDK + Drizzle ORM + express-session (Postgres-backed) + bcryptjs
- **Database**: PostgreSQL — tables: `sites`, `users`, `point_transactions`, `topup_requests`, `commissions`, `session`
- **AI**: OpenAI gpt-5.4 via Replit AI Integrations proxy
- **API contract**: OpenAPI in `lib/api-spec/openapi.yaml`, codegen via Orval

## Member system

- **Auth**: email + password (bcrypt), session cookie via express-session (cookie path `/`, 30 days). First registered user is auto-promoted to admin.
- **Points**: 100 free on signup, generate site = -10 pts, refine = -5 pts. 402 returned when insufficient.
- **Top-ups**: tier prices (50k=50pt, 100k=120pt+20%, 300k=400pt+33%, 1tr=1500pt+50%). Methods: VNPay / Momo / Bank transfer (placeholder UI — admin manually approves to credit points).
- **Affiliate (CTV)**: each user has a `referralCode`. New signups using a referral code earn the referrer 10% commission on every approved top-up. Admin marks commissions as paid.

## Key endpoints

Auth/account: `/api/auth/{register,login,logout,me}`, `/api/account/{transactions,topups,affiliate,pricing}`
Sites: `/api/sites` (CRUD), `/api/sites/generate`, `/api/sites/:id/refine`, `/api/sites/recent`, `/api/sites/stats` — all scoped to current user (admins see everything)
Admin: `/api/admin/users`, `/api/admin/users/:id/adjust`, `/api/admin/topups`, `/api/admin/topups/:id/decision`, `/api/admin/commissions`, `/api/admin/commissions/:id/pay`

## Pages

- `/` — hero with prompt input, format selector, templates, recent sites, stats
- `/login`, `/register` — auth (register supports `?ref=CODE`)
- `/sites`, `/sites/:id` — library and split editor with iframe preview
- `/templates` — gallery
- `/account` — profile + points + transaction history
- `/account/topup` — pricing tiers and request form (placeholder for VNPay/Momo)
- `/account/affiliate` — referral code, link, commission summary
- `/admin` — tabs for users, top-up approval, commission payouts (admin only)

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks/Zod from OpenAPI
- `pnpm --filter @workspace/db run push` — push DB schema changes
- `SESSION_SECRET` env var is required by the api-server.
