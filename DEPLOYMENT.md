# Deployment — GitHub + Vercel

## 1. Get the code onto GitHub

```bash
# from inside your project folder, once Claude Code has built Stage 1
git init
git add .
git commit -m "Initial scaffold"
```

Create a new empty repository on github.com (don't initialise it with a README — you already have one), then:

```bash
git remote add origin https://github.com/<your-username>/cbt-practice.git
git branch -M main
git push -u origin main
```

From then on, `git add . && git commit -m "..." && git push` sends new work to GitHub — do this after each build stage Claude Code completes, so you have a checkpoint to roll back to if a later stage breaks something.

## 2. Provision the database

Easiest path: Vercel Postgres, since it's one click from inside the Vercel dashboard once your project is connected (step 3). Alternatives (Neon, Supabase) work identically — you just get a `DATABASE_URL` connection string from wherever you provision it.

## 3. Connect the repo to Vercel

1. Go to vercel.com, sign in (GitHub login is simplest), click "Add New Project."
2. Select your `cbt-practice` repository.
3. Vercel auto-detects Next.js — leave the default build settings.
4. Before the first deploy, add environment variables (from your `.env.example`, with real values) under Project Settings → Environment Variables. At minimum: `DATABASE_URL`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL` (your real Vercel URL, not localhost, once deployed), `EMAIL_SERVER_*`, `EMAIL_FROM`, `ANTHROPIC_API_KEY`, `INSTRUCTOR_EMAILS`.
5. Deploy. Every future push to `main` auto-redeploys.

## 4. Run the first database migration against the live DB

Locally, with your real `DATABASE_URL` in `.env`:

```bash
npx prisma migrate deploy
npx prisma db seed   # if Claude Code set up a seed script for the three cases
```

## 5. Sanity check before real students touch it

- Sign in with your own `@keele.ac.uk` email — confirm you land as `INSTRUCTOR`, not `STUDENT`.
- Sign in (or ask a colleague to try) with a non-Keele email — confirm it's rejected.
- Run one full Practice Room session end to end and check the `Attempt` row actually landed in the database (Vercel Postgres has a built-in data browser, or use `npx prisma studio` locally against the live `DATABASE_URL`).

## Ongoing costs to keep an eye on

- **Anthropic API usage** — scales with number of messages sent across all students' sessions. Check usage in the Anthropic console periodically, especially in the first couple of weeks of real use.
- **Vercel** — free tier covers small projects; watch for bandwidth/function-execution limits if usage grows.
- **Database** — most providers have a free tier sufficient for 35 students' worth of data; keep an eye on storage if transcripts accumulate over a full term.
