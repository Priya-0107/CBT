# Build brief for Claude Code

Read `docs/PROJECT_SPEC.md` and `docs/ARCHITECTURE.md` fully before starting — they define what to build and how. Use `docs/schema.prisma` as the database schema and `docs/.env.example` as the required environment variables.

Build this in the stages below, in order. After each stage, stop and let me review before continuing to the next — don't build the whole app in one pass.

## Stage 1 — Project scaffold
- Initialize a Next.js 14 (App Router, TypeScript) project with Tailwind CSS.
- Set up Prisma with the schema from `docs/schema.prisma`, placed at `/prisma/schema.prisma`.
- Set up NextAuth with the Email (magic link) provider, restricting sign-in to `@keele.ac.uk` addresses (reject anything else at the sign-in callback). On first sign-in, check the email against `INSTRUCTOR_EMAILS` (comma-separated env var) and set role accordingly; default everyone else to `STUDENT`.
- Get a basic login page and an authenticated "hello, {name}" home page working end to end before moving on — this stage is done when I can sign in and see my own email reflected back.

## Stage 2 — Case library and chat (Practice Room)
- Build `/practice` — a page listing active `Case` records (name + brief only, never the systemPrompt).
- Build `/practice/[caseId]` — the chat interface: message list, text input, send button.
- Build `POST /api/chat` — takes `caseId` and the message history, loads that case's `systemPrompt` server-side, calls the Anthropic API (`claude-sonnet-4-6`, low max_tokens, e.g. 250) with that system prompt and the conversation as messages, returns the reply. The system prompt must never be sent to or readable by the client.
- Seed the database (a seed script is fine) with the three cases from `PROJECT_SPEC.md` (Alex, Sam, Jordan), writing out their full system prompts per the persona descriptions given there — including the instruction to deflect from crisis content rather than escalate.
- Done when I can have a full back-and-forth conversation with a seeded case in the browser.

## Stage 3 — Feedback scoring and attempt storage
- Add a "Reflect & get feedback" action that ends the current session.
- Build `POST /api/attempts` — takes the full transcript, calls the Anthropic API with a clinical-supervisor system prompt (score 0–2 on agenda/guided/technique/homework, each with a note, plus one overall tip — return strict JSON only, per the shape in `PROJECT_SPEC.md`), parses the result, and stores it as an `Attempt` row.
- Render the returned scores as a simple bar-per-dimension result screen after ending a session.
- Handle the case where the model doesn't return valid JSON — retry once, then fail gracefully with a message rather than crashing.
- Done when ending a session produces stored, displayed feedback, and I can retry the same case and get a second independent attempt.

## Stage 4 — My Progress and tiers
- Build `/progress` — the signed-in student's own attempt history (case, date, four dimension scores) and current tier, computed from `lib/tiers.ts` threshold logic (average dimension scores across all their attempts).
- No student can query or see another student's attempts — enforce this in the query itself (always scope by `session.user.id`), not just in the UI.
- Done when my own attempt history and tier render correctly, and there is no route or API response anywhere that leaks another user's data.

## Stage 5 — Weekly Check-Ins
- Build `/checkin/[week]` (student-facing, single question + free-text answer, stored as `CheckInResponse`) and instructor CRUD for `WeeklyCheckIn` records.
- Done when the instructor can add a weekly question and a student can answer it.

## Stage 6 — Instructor Dashboard
- Build `/dashboard` (instructor-only, enforced server-side, not just hidden nav): cohort-average dimension scores by case and by week.
- Build `/audit`: a list of recent attempts with transcript + AI scores shown together, plus a way to flag/annotate one as inaccurate (a simple boolean + text note field is enough for MVP).
- Build `/cases` (instructor CRUD for `Case` records, including editing `systemPrompt`).
- Done when I can see cohort-level trends, drill into any one transcript, and edit or add a case without touching code.

## Stage 7 — CSV export
- Add an export button on the dashboard that downloads all `Attempt` data as CSV, with `userId` in place of name/email (anonymised).

## Throughout, regardless of stage
- Never call the Anthropic API from client-side code — API routes only.
- Every instructor-only page and API route must check `role === 'INSTRUCTOR'` server-side, every time — not just hide the link.
- No feature anywhere shows one student's data to another student. If you're ever unsure whether a query is scoped correctly, ask rather than guess.
