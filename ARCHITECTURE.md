# Architecture — CBT Practice

## Stack

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 14 (App Router), TypeScript | Native fit for Vercel, server + client in one codebase, good for this scale of app |
| Styling | Tailwind CSS | Fast to build a calm, custom look without fighting a component library |
| Database | PostgreSQL, hosted on Vercel Postgres (or Neon/Supabase — any Postgres works) | Relational data (users, cases, attempts) fits Postgres well; Vercel Postgres has the least setup friction from within Vercel |
| ORM | Prisma | Type-safe queries, easy migrations, schema doubles as documentation |
| Auth | NextAuth.js, Email (magic link) provider, restricted to `@keele.ac.uk` in the sign-in callback | No passwords to manage; domain restriction is a one-line check |
| AI | Anthropic API, called **server-side only** (API routes), never from the browser | Keeps the API key secret and keeps case system prompts hidden from students |
| Hosting | Vercel | Matches "push via GitHub" — Vercel auto-deploys on every push to `main` once connected |

## Why server-side AI calls matter here specifically

The case system prompts (the client personas) must never reach the browser — a student inspecting network requests shouldn't be able to read "Alex's" hidden persona instructions. All AI calls go through Next.js API routes (`/api/chat`, `/api/attempts`) that hold the Anthropic API key as a server-only environment variable and construct the prompt server-side.

## Folder structure

```
/app
  /(auth)/login/page.tsx
  /(student)/practice/page.tsx              — case library
  /(student)/practice/[caseId]/page.tsx      — chat session
  /(student)/progress/page.tsx               — my progress
  /(student)/checkin/[week]/page.tsx
  /(instructor)/dashboard/page.tsx
  /(instructor)/cases/page.tsx
  /(instructor)/checkins/page.tsx
  /(instructor)/audit/page.tsx
  /api/chat/route.ts                         — POST: one roleplay turn
  /api/attempts/route.ts                     — POST: end session, score, store
  /api/checkins/route.ts
  /api/instructor/*                          — protected, instructor-only routes
/lib
  auth.ts                                    — NextAuth config
  prisma.ts                                  — Prisma client singleton
  anthropic.ts                               — Anthropic API wrapper
  tiers.ts                                   — tier threshold logic
/prisma
  schema.prisma
```

## Data model

See `schema.prisma` for the authoritative version. Summary:

- **User** — id, email, name, role (`STUDENT` | `INSTRUCTOR`), tier, timestamps.
- **Case** — id, name, label, brief (shown to students), systemPrompt (hidden), week, active.
- **Attempt** — id, userId, caseId, transcript (JSON array of turns), scores (JSON: one entry per dimension with score + note), overallFeedback, createdAt.
- **WeeklyCheckIn** — id, week, question, rubricNote, active.
- **CheckInResponse** — id, userId, checkInId, response, createdAt.

## Route protection

- `/(student)/*` — any authenticated user.
- `/(instructor)/*` — authenticated **and** `role === 'INSTRUCTOR'`, enforced in a layout-level check, not just hidden nav links.
- `/api/instructor/*` — same check, server-side, on every request — never trust the client to only call this if it can't see the nav link.

## Tier calculation

Keep thresholds as named constants in `lib/tiers.ts`, not hardcoded inline, so the instructor's ability to "tune them later" (per the spec) is a one-file change, not a hunt through the codebase. Compute a student's tier on read (from their stored attempts), not as a separately stored field that can drift out of sync — the `tier` field on `User` can be a cached/denormalised copy updated after each new attempt, but the source of truth is the attempt scores.
