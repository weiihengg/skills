---
name: preclass-brief
description: >
  Use when the user wants to prepare for an upcoming class. Triggers: "prep me for class",
  "what do I need to know before {module} today", "generate my pre-class brief",
  "I have class soon", "brief me on week {N}", or when the schedule shows a class is within
  24 hours and no brief has been generated for it yet.
sources: [chat]
aliases: [brief, pre-class, prep, class prep]
---

# preclass-brief

## Purpose

Generate a **scannable HTML brief** the user reads before class. Short by default (3–5 min).
Deep only on explicit request (10–15 min).

The brief is a **launchpad**, not a lecture summary. Its job is to make the material land
faster when the lecturer covers it — not to replace attending.

**First-screen rule:** The first screen of the brief must be completable as a standalone.
If the user reads nothing else, that screen was enough. Everything below the fold is bonus.

## Inputs

| File | What it provides |
|---|---|
| `schedule.md` | Next session, week number, Drive file path |
| `learner-profile.md` | Proficiency, flags, calibration notes |
| `glossary.md` | Past weak spots — used to shape what gets emphasised |

If `schedule.md` is missing → run `schedule-sync` first.
If `learner-profile.md` is missing → run `learner-profile` (silent bootstrap), then continue.
If `glossary.md` is missing → continue. It will be created when the first gap is logged.

## Steps

### Step 1 — Identify the target session

Read `schedule.md`. Find the next upcoming session (closest future session from now).
If the user named a specific module or week, use that instead.
If multiple sessions are on the same day, ask which one.

Check `schedule.md` for drift check due date. If overdue, run the drift prompt before continuing.

### Step 2 — Pull lecture materials

**Primary source:** this week's Drive file from `schedule.md`.

**Fallback chain if Drive file is missing:**
1. Check if the file was uploaded late — search the Drive folder again
2. If still missing → web search: `"{module name} week {N} {topic from schedule.md}"`
   Use the most authoritative result (university site, textbook resource, lecture notes repository)
3. If web search yields nothing useful → generate from module name + topic title alone
   Mark the brief header: `⚠ Generated without lecture slides — verify after class`

**Context files (scan only — do not summarise):**
- Previous week's file → identify topics that carry forward; reference briefly, do not re-explain
- Next week's file → find one concept seeded this week; use as the forward-link line

Completion criterion: primary source retrieved or fallback explicitly applied and marked.

### Step 3 — Analyse and scope

Before writing, extract:

1. **Core concepts** (3–5 for short, up to 8 for deep)
2. **Why it matters** — one sentence connecting the topic to the degree or real world
3. **Prerequisite check** — what must the user already know. Cross-check `glossary.md`:
   any prerequisite that is a known gap gets a one-sentence refresher on the first screen
4. **Forward link** — one concept from next week that this week seeds
5. **Jargon list** — new terms introduced this week

**Apply profile flags:**

| Flag | Effect on brief |
|---|---|
| `lazy-reader` | Hook is the first thing above the fold. First screen does full work. |
| `last-minute-prone` | Add a `★ If you read one thing` callout at the very top |
| `time-pressured` | Bullets over prose throughout. Cut any section that isn't core. |
| `example-driven` | Every concept anchored to a concrete example |
| `novice` | Define all jargon. Use analogies. No assumed vocabulary. |
| `competent` | Skip definitions. Lead with application and nuance. |

### Step 4 — Generate the HTML brief

**Default: short** (3–5 min). Deep only when user says "deep", "full", "thorough", "15 min".

#### Structure

```
[Header bar]
  Module | Week N | {date} | {short | deep} | ~{N} min read

[★ If you read one thing]  ← only if last-minute-prone flagged
  One sentence. The single most important thing from this lecture.

[Hook — 2–3 sentences]
  Why this lecture exists. What problem it solves. Why it matters.
  If lazy-reader: compelling enough to stop a scroll.

[From last week — carry-forward]
  Max 3 bullets. One sentence each.
  If any bullet matches a glossary gap → bold it + add one-sentence refresher inline.

[This week's core concepts]
  For each concept:
    - Name (bold)
    - One-line definition
    - Concrete example (expand if example-driven)
    - [deep only] Connection to other concepts

[Jargon to know]
  Term | Plain-English definition  (real <table>, not a list)

[What's next week]
  One sentence. Seed curiosity, not content.

[⚠ Brief accuracy — after class]
  "Did this brief match what actually came up in class?"
  [Yes] [Mostly] [No — it missed the point]
  (Tappable. Logs to glossary on response.)

[⚠ Missing material — if fallback was used]
  Which files were missing and what source was used instead.
```

#### Visual design rules

- Dark background `#0f1117`, off-white text `#e8e8e8`
- Module colour accent derived from module name (consistent across runs for the same module)
- Large section headers, generous line height — readable on a phone
- Max 3 sentences per paragraph before breaking to bullets
- Jargon section is a `<table>` — not a list
- Duration estimate in header
- `★ If you read one thing` callout uses a distinct highlight colour, full-width, above the fold

Completion criterion: HTML renders without errors, all sections present, first screen is
standalone-complete, profile flags visibly applied.

### Step 5 — Log to glossary

Call `learner-profile` (Mode B):
- Log any glossary-gap concepts resurfaced this session
- Record: module + week briefed, brief mode (short/deep), Drive source or fallback used

### Step 6 — Deliver and prompt

Output the HTML artifact. Then in chat:

```
Brief ready — {Module} Week {N}. Short mode, ~{N} min.
After class, run `postclass-quiz` to lock in what you just heard.
```

## Brief accuracy feedback

When the user taps [Yes / Mostly / No] in the brief:
- **Yes** → log `brief_accuracy: on-point` for this module + week in glossary
- **Mostly** → ask "What did it miss?" → log the gap
- **No** → ask "What actually came up?" → log full gap, flag this module's Drive files
  for manual review (slides may be misnamed or misweekd in the folder)

This signal compounds. If a module consistently scores "No", the Drive mapping for that
module likely has a naming mismatch — prompt the user to re-run `schedule-sync` for it.

## Notes

- Brief is a launchpad. Do not reproduce the full lecture — synthesise and frame.
- Previous and next week files are for context scoping only. This week stays the focus.
- The first-screen rule is non-negotiable regardless of mode. Deep mode adds depth below
  the fold — it does not delay the hook.
