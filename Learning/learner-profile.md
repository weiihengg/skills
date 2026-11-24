---
name: learner-profile
description: >
  Use when setting up a new user, updating what is known about them, or logging a question
  or gap from any study session. Triggers: first time any study skill runs and learner-profile.md
  is missing, "update my profile", user reveals course or degree details, user asks a question
  that reveals a knowledge gap or strong prior, after every brief and quiz session to log gaps.
sources: [chat]
aliases: [profile, learner, student profile, glossary update]
---

# learner-profile

## Purpose

Maintain two files all study skills read before generating any content:

- **`learner-profile.md`** — who the user is, what they study, how they learn
- **`glossary.md`** — a living record of questions, gaps, and explanations that worked

These are the memory of the system. The profile builds silently over time — no upfront
interview gates access to other skills. Every session adds signal.

## Modes

**Mode A — Silent bootstrap**: `learner-profile.md` is missing. Create a minimal stub from
whatever the user has already said (course name, module, any visible context). Do not interview.
The stub gets enriched automatically as sessions accumulate.

**Mode B — Glossary update**: Called by another skill to log a question or gap. Skip to Step 4.

**Mode C — Profile enrichment**: New information emerges mid-session (user mentions their degree,
corrects an assumption, reveals a strong prior). Update the relevant fields only. No re-interview.

**Mode D — Explicit refresh**: User directly asks to update their profile. Run Step 2 as a
short, targeted conversation — only ask what isn't already known.

## Steps

### Step 1 — Detect mode and bootstrap if needed

Check if `learner-profile.md` exists:
- Missing → Mode A: create minimal stub (Step 2a), then continue with the calling skill
- Exists + called with gap to log → Mode B: go to Step 4
- Exists + new info surfaced mid-session → Mode C: go to Step 3 (partial update)
- Exists + user asked for update → Mode D: go to Step 2b

### Step 2a — Silent bootstrap (Mode A)

Do not ask the user anything. Infer from context available:
- Degree / course name (from anything they've said or uploaded)
- University (if mentioned)
- Module name (from the session that triggered this)
- Year (if stated)

Write a minimal `learner-profile.md` stub:

```markdown
# Learner Profile

Last updated: {date}
Bootstrap: silent — enriched from session context

## Identity
- Degree: {inferred or "not yet known"}
- University: {inferred or "not yet known"}
- Year: {inferred or "not yet known"}

## Modules this semester
- **{Module triggering this session}**
  - Proficiency: developing (default — will adjust from session signals)
  - Flags: none yet

## Study habits
Flags: none yet
Calibration notes: profile is new — signals accumulating

## Content calibration notes
Insufficient data. Using defaults until 2+ sessions completed.
```

Completion criterion: stub written, calling skill can proceed immediately.

### Step 2b — Targeted interview (Mode D)

Ask only what isn't already in the profile. One or two questions at a time, conversationally.
Never re-ask what's already known. When done, go to Step 3.

### Step 3 — Update profile fields

Update only the fields that changed or were newly inferred. Do not rewrite the whole file
unless the structure needs fixing.

**Proficiency guesstimate** — infer from behaviour, never ask directly:

| Signal | Inference |
|---|---|
| Vague questions, asks "what is X" | `novice` |
| Uses terms correctly, asks "how does X relate to Y" | `developing` |
| Challenges the explanation, asks edge cases | `competent` |
| Consistently answers quiz questions correctly | upgrade one level |
| Consistently misses questions marked easy | downgrade one level |

**Habit flags** — infer from behaviour:

| Signal | Flag |
|---|---|
| Mentions being rushed, cramming, no time | `time-pressured` |
| Asks for shorter explanations | `concise-learner` |
| Engages more after examples | `example-driven` |
| Skips readings, opens brief late, one-word answers | `lazy-reader` |
| Runs last-minute skill | `last-minute-prone` |

Flags accumulate. A student can be `lazy-reader` + `example-driven` simultaneously.

**`lazy-reader` note:** This flag makes content generators hook harder and front-load value.
It is a calibration tool, not a judgement.

**`last-minute-prone` note:** If this flag is set, `preclass-brief` will add a one-line
"what matters most if you read nothing else" callout to the top of every brief automatically.

Completion criterion: all changed fields updated, proficiency and flags reflect current signals.

### Step 4 — Update glossary.md (Mode B)

`glossary.md` is append-only except for `Explanation that worked` and `Recurrence`, which update in-place.

**Log an entry when:**
- User asks a question during any session
- User gets a quiz question wrong or asks for clarification
- An explanation visibly worked ("oh that makes sense", correct answer after explanation)
- User explicitly says they don't understand something

**Entry format:**

```markdown
## {Concept, written as the user asked it or as close as possible}

- Module: {module name}
- Week: {week number}
- Logged: {date}
- Type: {confusion | gap | worked-example}
- User phrasing: "{how they asked or how the gap presented}
- Explanation that worked: {fill if found; leave blank otherwise}
- Recurrence: {count}
- Last seen: {date}
```

**Update rules:**
- Concept already exists → increment `Recurrence`, update `Last seen`, append new date
- New explanation worked → fill or overwrite `Explanation that worked`
- Concept resolved → add `[resolved — {date}]` after heading. Never delete.
- Recurrence ≥ 3 → flag for proficiency review: add `[persistent gap — review proficiency]`

Completion criterion: every gap and question from the current session logged before skill exits.
The glossary is the most compounding asset in this system — no entry skipped.

### Step 5 — Confirm (Mode D only)

```
Profile updated.
Modules: {list}
Proficiency: {module: level}
Flags: {flags}
Glossary entries: {total count}
```

## Notes

- Never gate another skill behind a completed profile. The stub is enough to proceed.
- Proficiency shifts. Adjust it when behaviour consistently contradicts the current rating.
- The glossary compounds. An entry logged today will surface in briefs and quizzes for the
  rest of the semester. Completeness here is the highest-leverage action in the whole system.
