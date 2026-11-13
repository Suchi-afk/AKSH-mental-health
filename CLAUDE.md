# CLAUDE.md, MindBridge

This file is the standing brief for Claude Code on this project. It is read
automatically at the start of every session, before any prompt. It exists
so the app stays consistent, calming, and trustworthy across every prompt.

## What this project is

MindBridge is a hybrid mental health support platform: an AI chatbot provides
immediate emotional support, detects risk signals, and -- with explicit user
consent -- connects the user to a real therapist for a scheduled session. It
is not a medical tool: the AI is assistive, never diagnostic.

## Tech stack

- Next.js (App Router) + TypeScript, strict mode
- Tailwind CSS + shadcn/ui components
- Prisma ORM 7 + SQLite, via the @prisma/adapter-better-sqlite3 driver
  adapter, zero external services required
- Auth.js (NextAuth) v5, Credentials provider, JWT sessions, bcrypt hashing
- zod v4 for all validation
- sonner for toasts

Nothing in this stack requires an API key, a cloud signup, or an internet
connection at runtime. This is deliberate -- see "Offline-safety" below.

## Prisma 7 setup, exact shape, do not use an older pattern

Run `npm install prisma@7.10.0 @prisma/client@7.10.0 --save-exact`, pinned
exactly like that. The unpinned "latest" tag currently resolves to an
8.0.0 release candidate, not a stable release, and mixing prisma 8 with
@prisma/client 7 breaks the build.

The generator block in schema.prisma outputs the client to a project path,
not the default @prisma/client location:

    generator client {
      provider = "prisma-client"
      output   = "../app/generated/prisma"
    }

Import it as `from "@/app/generated/prisma/client"`, never the bare
`@prisma/client` package. PrismaClient requires a driver adapter, it
throws on construction otherwise. For SQLite:

    npm install @prisma/adapter-better-sqlite3

    import { PrismaClient } from "@/app/generated/prisma/client";
    import { PrismaBetterSqlite3 } from "@prisma/adapter-better-sqlite3";

    const adapter = new PrismaBetterSqlite3({ url: process.env.DATABASE_URL! });
    export const prisma = new PrismaClient({ adapter });

DATABASE_URL lives in .env as `file:./dev.db`, and AUTH_SECRET needs any
random string set in .env for Auth.js to run.

## Repository layout

app/                      route segments (App Router)
  (marketing)/             landing page
  chat/                     AI chat screen (first feature)
  therapists/               therapist listing and scheduling
  dashboard/                user dashboard
  login/ signup/
  generated/prisma/         generated Prisma client, do not edit by hand
components/               shared UI (shadcn primitives + app-specific)
lib/                       db client, auth config, validation schemas, AI logic
prisma/
  schema.prisma
  seed.ts
public/uploads/            locally-generated placeholder images

Co-locate a feature's server actions, zod schemas, and components inside
its own route folder rather than spreading them across shared "utils" files.

## Design system, read this before writing any UI

The bar is "calm, trustworthy startup," not "mental health template."
Deliberately not the default AI-generated look:

- Color: a single teal accent (#0e7c7b) for primary actions. No indigo,
  no violet, no purple-to-blue gradients. Zinc-50 for background, zinc-900
  for text. Emerald for success, rose for destructive, amber for warnings.
- Type: Fraunces (via next/font/google) for headings, warm serif, tight
  tracking, bold; IBM Plex Sans for body text. Not Inter -- it is the
  single most common tell that a UI came straight out of an AI tool.
- Shape: rounded-2xl on cards and inputs, rounded-full on badges and pills.
- Depth: soft shadows only (shadow-sm resting, shadow-lg + a subtle
  scale-[1.02] on hover for cards). No harsh drop shadows, no glow or
  bounce on hover. Purposeful motion only.
- Layout: left-align hero text, never centered. Vary section widths; never
  a uniform 3-card grid unless there really are exactly 3 things.
- Spacing: generous whitespace; sections use py-16/py-24.
- Every view needs three states, no exceptions: a loading skeleton, a
  friendly empty state (icon + message + CTA, never a blank page), and an
  error state with a retry action.
- Accessibility: semantic HTML, real <label>s, visible focus rings, AA
  contrast, alt text on every image.
- Mobile-first: design at 375px width first; nav collapses on small screens.

## Content tone

Write for one nervous person arriving alone. Calm, warm, human -- never
clinical, never authoritative, never corporate. The AI chatbot speaks like
a calm friend, not a therapist. The app never diagnoses, never prescribes,
never uses words like "patient," "symptom," or "treatment" outside of the
explicit disclaimer that it is assisting, not diagnosing.

## Data & backend conventions

- All mutations go through Server Actions co-located as actions.ts next to
  the route that uses them. Validate with zod before touching the database.
- Prefer Server Components for reads; drop to a Client Component only
  where interactivity is required (chat, scheduling).
- Server actions return { success: true, data } or
  { success: false, error }, never throw raw errors to the client.
- prisma/seed.ts must leave the app fully populated on first run. An empty
  database is a bug, not a valid starting state.

## Auth & security

- Every server action that mutates data re-checks the session server-side.
  Never trust a client-side "isLoggedIn" flag.
- Passwords are hashed with bcrypt; sessions are JWT-based via Auth.js.
- Secrets live in .env (gitignored); commit .env.example with placeholder
  values only.

## Offline-safety, important for live demos

This app must run end-to-end with no internet access after npm install:
no external image URLs, no third-party APIs, no OAuth providers. Seed data
writes real local SVG placeholder files to public/uploads at seed time,
referenced by a normal path, not a data URI.

## AI chatbot constraints

- The chatbot is a structured prompt flow, not a free-form LLM call in
  this demo version. Responses are generated server-side from a fixed
  empathetic prompt plus the user's messages. No external AI API key
  required for this demo.
- The chatbot NEVER provides medical diagnoses or prescriptions.
- The chatbot NEVER stores or transmits real personal health data beyond
  what lives in this SQLite database.
- When risk signals are detected, the bot suggests a therapist. It never
  diagnoses risk level. Consent is always explicit and explicit only.

## Code style

- No any. Components are PascalCase; utilities are camelCase.
- Comments explain non-obvious why only, never restate what the code
  already says.

## Definition of done, per feature

- Works at 375px and at desktop width
- Has loading, empty, and error states
- npm run build passes with zero type errors
- No console errors/warnings in the browser
- Server-side validation mirrors client-side validation
- You clicked through the happy path and one edge case yourself

## Local dev commands

npm install
npm run db:push && npm run db:seed
npm run dev            # http://localhost:3000

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->
