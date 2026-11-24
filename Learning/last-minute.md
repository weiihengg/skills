---
name: last-minute
description: >
  Use when the user has not studied for most or all of the semester and needs to prepare
  for an upcoming exam on a single module. Triggers: "I haven't studied all semester",
  "finals are coming and I'm screwed", "last minute mode", "I need to cram {module}",
  "exam is in {N} days and I haven't started", "help me survive {module} exam".
  This is not a pre-class brief — it is a semester-compression sprint.
sources: [chat]
aliases: [cram, last minute, exam cram, survival mode, panic mode]
---

# last-minute

## Purpose

Compress an entire semester of one module into the minimum viable understanding needed
to pass the exam. This skill is honest about what's achievable. It does not pretend the
student can master everything — it helps them **survive strategically**.

One module per session. Scope never expands beyond that.

## The core constraint

The student has a knowledge void, not a knowledge gap. Every step here is ordered around
that reality. Teaching comes last — triage and time math come first.

## Inputs

Collect before proceeding:

| Input | What to ask if missing |
|---|---|
| **Module name** | "Which module is this for?" |
| **Exam date** | "When is your exam?" |
| **Hours available per day** | "Realistically, how many hours a day can you study?" |
| **Drive folder / lecture files** | "Do you have your lecture slides? Share the folder or upload them." |
| **Past papers** | "Do you have any past exam papers? Even one helps a lot." |

If `learner-profile.md` exists, read it for proficiency and habit flags before proceeding.
If it doesn't exist, create a silent bootstrap via `learner-profile` (Mode A) using whatever
context is available, then continue.

## Steps

### Step 1 — Time math

Do this before anything else. Be honest.

Calculate:
- Days until exam: `{exam date} - today`
- Total available study hours: `days × hours/day`
- Estimated hours needed per topic: based on proficiency
  - `novice` on a topic → 3–4 hours to reach exam-answerable level
  - `developing` → 1–2 hours
  - `competent` → 0.5 hours (review only)

Print the honest summary:

```
Time available: {N} days × {H} hours = {total} hours
Module: {module name}

We'll figure out how many topics are coverable in Step 2.
```

Do not build the study plan yet. Topic triage comes first.

### Step 2 — Triage all semester topics

Scan all lecture files from the Drive folder (or uploaded files).
Build a topic list covering the full semester — every lecture, every week.

For each topic, estimate **exam weight** using this priority order:
1. **Past paper frequency** — if past papers provided, count how many times each topic
   appears as a question. This is the strongest signal.
2. **Slide density** — more slides + worked examples = lecturer considers it important
3. **Explicit signals** — words like "this is important", "exam favourite", "you will need this"
   found in slides or notes
4. **Recency bias** — topics from the last 3 weeks of semester tend to appear on finals
5. **Foundational dependencies** — topics that unlock other topics (you can't answer Q2 without knowing this)

Assign each topic one of three tiers:

| Tier | Meaning |
|---|---|
| **Must** | High exam weight. Cannot skip. Answering this correctly = passing. |
| **Should** | Medium weight. Cover if time allows after Must topics. |
| **Skip** | Low weight, low frequency, or requires Must knowledge to even attempt. Cut. |

Print the triage table:

```
Semester triage — {Module}

MUST (cover these first):
  - {Topic} | Exam appearances: {N} | Est. hours: {N}
  - ...

SHOULD (cover if time remains):
  - {Topic} | Exam appearances: {N} | Est. hours: {N}
  - ...

SKIP (explicitly cutting):
  - {Topic} — reason: {low frequency | requires Must first | out of time}
  - ...

Realistic coverage with {total} hours:
  Must topics: {N of M} coverable
  Should topics: {N} if time remains
```

Ask: "Does this look right? Any topics you know are definitely on the exam that I ranked wrong?"
Adjust based on user input.

Completion criterion: every semester topic assigned a tier, honest coverage estimate given,
user has confirmed or corrected the triage.

### Step 3 — Past paper pattern analysis

If past papers are available, run this step. Skip if none provided.

For each Must topic, extract:
- **Question type** — calculation, definition, essay, short answer, MCQ
- **Typical marks** — how many marks does this question carry
- **Standard phrasing** — how the question is usually asked ("Explain the relationship between X and Y", "Calculate Z given...")
- **What the mark scheme rewards** — keywords, structure, specific values

Output per topic:

```
{Topic}
  Typical question: "{example phrasing}"
  Question type: {calculation | definition | essay}
  Marks: {N}
  What to include in your answer: {bullet list of mark-scheme keywords or expected steps}
  Past paper appearances: Weeks {list}
```

This is the most valuable output of this skill. A student who knows the question pattern
and the mark-scheme keywords can answer a question they don't fully understand.

Completion criterion: every Must topic has a question pattern. Should topics get patterns
only if time permits.

### Step 4 — Compressed topic briefs

For each Must topic, in priority order, generate a **compressed exam-answerable brief**.
This is not a teaching document — it is the minimum viable understanding to answer the
likely exam question correctly.

**Format per topic:**

```
## {Topic name} ★ MUST

The exam-answerable version:
{2–4 sentences. What it is, why it matters, what the key relationship or value is.
Written to be recalled under pressure, not to build deep understanding.}

Key terms to use in your answer:
  - {term}: {one-line definition}
  - ...

The likely question:
  "{standard question phrasing from past papers, or best estimate}"

What your answer needs:
  - {mark-scheme point 1}
  - {mark-scheme point 2}
  - {calculation steps if applicable}

Common mistake:
  {The one thing students get wrong on this topic. One sentence.}
```

**Apply profile flags:**
- `novice` → add one concrete analogy per topic
- `example-driven` → add one worked example per topic
- `lazy-reader` → the exam-answerable version must be the first thing. No preamble.
- `last-minute-prone` → this flag is already why they're here. Use direct language.
  No hedging. Tell them exactly what to write.

Do not generate Should topic briefs until all Must briefs are done.

Completion criterion: every Must topic has a compressed brief. Quality check: could a student
read this brief and write a passing answer to the likely question?

### Step 5 — Study schedule

Now build the schedule, not before. Use the time math from Step 1 and the tier assignments
from Step 2.

```
Study schedule — {N} days to exam

Day 1 ({date}): {Topic A} + {Topic B} — {H} hours
Day 2 ({date}): {Topic C} + revision of Day 1 — {H} hours
...
Day {N-1}: Full Must topic sweep — all topics, 10 min each
Day {N} (exam day): Morning only — jargon list + common mistakes. Stop by noon.
```

**Scheduling rules:**
- Must topics spread across the first 60% of available days
- Last day before exam → revision only, no new material
- Exam morning → jargon and common mistakes only. Hard stop.
- Interleave: do not spend two consecutive days on the same topic
- If total hours are fewer than Must topics require → be explicit:
  "You don't have enough time to cover all Must topics properly.
   Prioritise: {top 3 Must topics}. These are your exam."

### Step 6 — Compressed spaced repetition loop

At the end of each study session within this skill, run a mini-quiz on the topics just covered.
3–5 questions max. Exam-style, not Socratic.

**Format:**
- Ask the question the way the exam would
- User answers
- Reveal what the mark scheme expects
- Flag what was missing

Log every gap to `glossary.md` via `learner-profile` (Mode B).

After 2 sessions, re-quiz on Day 1 topics briefly before starting Day 2 material.
This is the entire spaced repetition system under time pressure — simple, not sophisticated.

### Step 7 — Deliver

Output as an HTML artifact:
- Full triage table
- Past paper patterns (if available)
- All Must topic briefs
- Study schedule
- Day-by-day checklist the user can tick off

Then in chat:

```
Last-minute pack ready — {Module}.

Must topics: {N} | Should (if time): {N} | Skipped: {N}
Days to exam: {N} | Realistic coverage: {coverage estimate}

Start with {first Must topic}. Run `last-minute` again after each study day
to log gaps and get the next day's focus adjusted.
```

## Honesty rules

This skill does not sugarcoat. If the student cannot cover all Must topics in the time
available, say so explicitly. Give them the best possible strategy given the constraint —
do not pretend the constraint doesn't exist.

If the student has fewer than 2 days and more than 4 Must topics:

```
Honest assessment: you don't have enough time to cover this properly.
Focus only on: {top 2–3 Must topics}.
For everything else: memorise the jargon list and the common-mistake line.
A partial answer with correct terminology scores more than a blank.
```

## Notes

- This skill runs per module, per exam. For multiple exams, run it separately for each.
- Past papers are the highest-signal input in this skill. If the user doesn't have them,
  ask if they can get them from their university portal, library, or a classmate.
- The compressed brief is not deep understanding. It is exam survival. Flag this clearly
  so the student knows what they're getting — and what they're trading away.
- Gaps logged here feed into `glossary.md` permanently. If the student uses the full
  system next semester, those gaps will already be tracked.
