# Project spec — CBT Practice

## Context

PSY-30170, Cognitive & Behavioural Therapy Practice — a 3rd-year Psychology with Counselling module at Keele University, 35 students, one instructor (module leader). The module's main assessment is a recorded 20-minute simulated client induction session. This app gives students private AI-simulated rehearsal before that assessment, and gives the instructor visibility into cohort readiness — **without** ever exposing one student's performance to another.

## Roles

- **Student** — signs up with a `@keele.ac.uk` email. Can use the Practice Room, Weekly Check-Ins, and see only their own progress.
- **Instructor** — one account (the module leader) for MVP. Sees cohort-level data, individual student drill-downs, manages case content and weekly check-ins, and audits AI-generated feedback.

## Feature 1 — Practice Room (AI-simulated client roleplay)

A library of client cases. Seed with three:

1. **"Alex" — Social phobia** (induction session). 21-year-old undergraduate, intense fear of judgement in seminars, avoidant, guarded but not hostile. Opens up gradually with good open/Socratic questions; closes down with leading or rushed ones.
2. **"Sam" — OCD** (early assessment session). Checking/contamination-related compulsions affecting daily functioning. Embarrassed, minimises at first. Responds well to non-judgemental curiosity; gives vague answers to blunt/direct questions.
3. **"Jordan" — Complex/comorbid presentation** (later-stage session). Anxiety with a secondary low-mood presentation and some emotional dysregulation. Tests the trainee's ability to structure a session under complexity; less linear than the other two.

Each case has: a display name, a one-paragraph clinical brief (shown to students), and a system prompt defining the persona in detail (**never shown to students**).

Flow:
- Student picks a case, sees the brief, starts a chat.
- Student types as the therapist; the AI responds in character, one turn at a time.
- The AI must never break character, never reference being an AI, and must mildly deflect — not escalate — if a message drifts toward self-harm, suicide, or acute crisis content. These cases are not designed for crisis roleplay.
- A "Reflect & get feedback" action ends the session. The full transcript is sent to a second AI call (a clinical-supervisor role) that scores the attempt 0–2 on four dimensions, each with a short note grounded in the transcript, plus one actionable tip:
  - Agenda Setting & Collaboration
  - Guided Discovery (Socratic Questioning)
  - Application of CBT Technique
  - Homework / Task Setting
- Every attempt (student, case, transcript, scores, timestamp) is stored. Students can retry any case any number of times.

## Feature 2 — Weekly Skill Check-In

A short single-scenario question released weekly (11 weeks total), tied to that week's teaching topic. Instructor can add/edit the question and a model-answer/rubric note. Lighter weight than a full Practice Room session — a few minutes, not a roleplay.

## Feature 3 — Certification tiers

Three tiers: **Trainee → Associate → Certified**, gated by a student's average dimension scores across their Practice Room attempts (not by raw activity count). Thresholds should be configurable, not hardcoded — the instructor will want to tune them after seeing real score distributions. A student sees only their own tier; it's never shown to other students.

## Feature 4 — My Progress (student-facing)

A private view per student: attempt history per case, a simple trend of the four dimension scores over time, current tier.

**No leaderboard, no ranking, no visibility into any other student's scores or names, anywhere in the student-facing app.** This is a deliberate, non-negotiable constraint — the design is explicitly avoiding public evaluative pressure on applied clinical skill.

## Feature 5 — Instructor Dashboard

- Cohort-level view: average dimension scores across all students, by case and by week — where is the cohort collectively weak.
- Individual student drill-down: full attempt history and transcripts for a named student.
- **AI feedback audit view**: recent AI-scored attempts shown with transcript and scores side by side, so the instructor can spot-check accuracy and flag/annotate attempts where the AI scored wrong.
- Case management: add/edit/retire cases and their system prompts, no code required after launch.
- Weekly Check-In management: add/edit weekly questions.

## Design tone

Calm, clinical-but-human — a therapeutic skills tool, not a game show. No slot-machine XP bursts, no competitive framing. Warm, quiet, professional; closer to a journal/case-notes feel than an arcade feel.

## Data & privacy

- Registration restricted to `@keele.ac.uk` addresses.
- Student transcripts and scores are never visible to other students, under any view.
- Instructor can export all attempt data as CSV, anonymised by student ID rather than name, for research/evaluation use.
