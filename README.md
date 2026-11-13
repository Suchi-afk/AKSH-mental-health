# MindBridge (prototype name: MINDLINK)

MINDLINK is a hybrid mental-health support demo. A calm AI support assistant
offers immediate emotional support, detects risk signals, and — with explicit
consent — connects people to a real therapist for a scheduled session. The AI
is assistive and never diagnostic.

Runs fully offline after `npm install`: SQLite via Prisma + the
`@prisma/adapter-better-sqlite3` driver, no API keys, no external services.

## Stack

- Next.js (App Router) + TypeScript (strict)
- Tailwind CSS v4 + shadcn-style UI components
- Prisma 7 + SQLite (driver adapter, generated client under `src/app/generated/prisma`)
- Cookie/JWT-style sessions (bcrypt-hashed passwords), route-guarded by role
- zod for every validation boundary; sonner for toasts

## Setup

```bash
npm install
npm run db:push        # sync schema into dev.db
npm run db:seed        # create demo users, therapists, slots, chat data
npm run dev            # http://localhost:3000
```

Reset everything whenever you want a clean demo:

```bash
npm run db:reset
```

## Demo accounts (password: `demo1234`)

| Role      | Email                          | What you'll see                       |
|-----------|--------------------------------|---------------------------------------|
| Client    | maya@mindlink.demo             | Chat, therapists, booking, insights, appointments |
| Therapist | anaya.therapy@mindlink.demo    | Dashboard, clients, sessions, availability |
| Admin     | admin@mindlink.demo            | `/admin` overview, add therapists, alerts |

Extra demo therapists: `kabir.therapy@mindlink.demo`, `priya.therapy@mindlink.demo`,
`neha.therapy@mindlink.demo`.

The demo client already has a conversation with the AI, a consented summary,
one completed session, one upcoming confirmed session, and therapists have
real availability slots to book.

## Explanation of these

- `.env` — `DATABASE_URL="file:./dev.db"` plus `AI_PROVIDER="mock"`.
  Commit `.env.example` with placeholder values; never commit real secrets.
- `prisma/seed.ts` — writes local SVG avatars to `public/uploads` at seed
  time so the app needs no internet at runtime.
- Role-based route groups: `(client)`, `(therapist)`, `(admin)`,
  `(marketing)`.

## Commands

```bash
npm run dev          # start dev server
npm run build        # production build
npm run lint         # eslint (must be clean)
npm run db:push      # apply schema to dev.db
npm run db:seed      # (re)seed demo data
```