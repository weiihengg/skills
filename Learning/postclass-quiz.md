---
name: postclass-quiz
description: >
  Use after the user attends a class to quiz them on what was covered. Triggers: "quiz me",
  "test me on {module}", "I just got out of class", "postclass quiz", "drill me on week {N}",
  or when user asks to be tested on lecture material after a session.
sources: [chat]
aliases: [quiz, post-class, drill, test me]
---

# postclass-quiz

## Purpose

Run a **Socratic quiz** after a lecture. The goal is not to grade — it is to surface gaps
while the material is fresh, then write those gaps into `glossary.md` so future briefs and
quizzes are sharper.

Socratic means the user explains, not you. Resist explaining until they've genuinely tried.

## Inputs

| File | What it provides |
|---|---|
| `schedule.md` | Which session just ended, week, Drive file |
| `learner-profile.md` | Proficiency, habits, calibration notes |
| `glossary.md` | Past weak spots — priority targets for this quiz |

## Steps

### Step 1 — Identify the session

Find the most recently completed session in `schedule.md`.
If the user named a specific module or week, use that.

Confirm: "Quizzing you on **{Module} Week {N}** — {topic}. Ready?"
Do not start questions until the user confirms.

### Step 2 — Load material and prioritise

Retrieve the Drive file for this session. Also load the pre-class brief if one was saved
for this module + week.

Extract:
- Core concepts covered this week
- Jargon introduced
- Concepts flagged as forward-links from last week's brief

Cross-reference `glossary.md`:
- Concepts with `Recurrence ≥ 2` → **priority targets**, quiz these first
- Concepts marked `[persistent gap]` → must appear in this quiz, no exceptions
- `Explanation that worked` entries → use the same framing if re-explaining is needed

Completion criterion: concept list built, priority targets identified, persistent gaps flagged.

### Step 3 — Run the quiz (Socratic mode)

**One question at a time.** Never dump a list.

**Response protocol after each answer:**

| Answer quality | Response |
|---|---|
| Correct | Affirm briefly (one line). Go deeper or move on. |
| Partially correct | Follow-up: "You got the what — can you explain the why?" |
| Wrong | Ask a simpler scaffolding question. If still wrong after two attempts, explain, then re-ask a variant. |
| "I don't know" / blank | Ask a narrower scaffolding question. If still stuck, explain and re-ask. |

**Question types by proficiency:**

| Proficiency | Style |
|---|---|
| `novice` | "What is X?", "What does X do?" — recall and definition |
| `developing` | "Why does X happen?", "What's the difference between X and Y?" |
| `competent` | "If X changes, what breaks?", "When would you use X over Y?" |

**Apply habit flags:**

| Flag | Effect |
|---|---|
| `lazy-reader` | Open with the highest-stakes question. No warm-up. If two consecutive "idk" or one-word answers → end quiz, go to debrief, log all untouched concepts as gaps. |
| `example-driven` | After every correct answer: "Give me an example of that." |
| `time-pressured` | Hard cap at 5–7 questions. Offer to continue but do not extend by default. |
| `last-minute-prone` | Note in debrief which gaps are highest exam-risk. |

**Default length:** 5–8 questions. Stop and debrief unless user asks to continue.

Completion criterion: at least one question per core concept, all priority targets attempted,
all persistent gap concepts attempted.

### Step 4 — Debrief with forward momentum

Do not just list failures. Frame gaps as work-in-progress, not verdicts.

```
Quiz done — {Module} Week {N}

✓ Strong: {concepts answered well — one line each, what they showed}
↻ Gaps this session: {concepts missed — one line each}
★ Persistent gaps: {concepts at recurrence ≥ 3 — flag these explicitly}

What happens next:
- These gaps are now in your glossary and will be front-loaded in your next brief.
- {Specific gap} will come back in Week {N+1} — you'll see it again soon.
- Run `preclass-brief` before {next session date} to get ahead of it.
```

The debrief ends with a forward action, not a score. The student knows what to do next.

### Step 5 — Brief accuracy check

Ask one question after the debrief:

> "One more thing — did your pre-class brief cover what actually came up in class today?
> [Yes] [Mostly] [No — it missed the point]"

Log the response via `learner-profile` (Mode B):
- **Yes** → `brief_accuracy: on-point` for this module + week
- **Mostly** → ask "What did it miss?" → log the gap
- **No** → ask "What actually came up?" → log full gap, flag Drive mapping for this module

### Step 6 — Update glossary

Call `learner-profile` (Mode B) before exiting:
- Every concept missed or requiring scaffolding → log or increment recurrence
- Every explanation that visibly worked → update `Explanation that worked`
- Every concept answered correctly at `competent` level → consider marking resolved if
  recurrence was ≥ 2 and they nailed it cleanly

Completion criterion: glossary updated. Never exit without logging.

## Notes

- Tone is conversational, not interrogative. This is a study session, not an exam.
- Recurrence ≥ 3 on a concept is a signal to revisit proficiency rating for that module.
  Call `learner-profile` Mode C to downgrade if the pattern is consistent.
- If the user got out of class hours ago and says they've forgotten most of it — log that
  as a signal (`time-to-quiz: delayed`), run a compressed re-teach from the Drive file,
  then quiz. Delayed quizzing still beats no quizzing.
