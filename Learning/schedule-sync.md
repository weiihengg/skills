---
name: schedule-sync
description: >
  Use when the user wants to set up or update their class schedule for the study skills system.
  Triggers: "set up my schedule", "sync my timetable", "add a new module", "my Drive folder is X",
  "here is my timetable", "update my schedule", or whenever schedule.md is missing and another
  skill needs it. Also triggers automatically every 3 weeks to check for timetable drift.
sources: [chat]
aliases: [schedule, timetable, sync schedule]
---

# schedule-sync

## Purpose

Parse the user's timetable and Drive folder into `schedule.md` — the single source of truth
all downstream skills read to know *what* is being studied and *when*.

## Inputs required

Collect both before proceeding. If either is missing, ask directly:

| Input | What to ask |
|---|---|
| **Timetable** | "Please paste your timetable or upload it. Include day, time, module name, and venue." |
| **Drive folder path or URL** | "What is the Google Drive folder path or shared URL where your lecture slides live?" |

Do not proceed to Step 1 until both inputs are in hand.

## Steps

### Step 1 — Parse the timetable

Extract for every class session:
- Module name (full name + short code if present)
- Day of week + time
- Venue / mode (physical, online)
- Semester start date (ask if not in timetable)
- Total number of weeks

Infer the **calendar date** of every session from semester start + week offset.
If semester start date is missing, ask: "When does your semester start? I need the date to calculate each week's class dates."

**Handle messy timetables explicitly:**
- Public holidays → mark session as `[cancelled — public holiday]`, do not shift week count
- Replacement classes → add as a separate row tagged `[replacement]` with the original session it covers
- Cancelled sessions → mark `[cancelled]` with reason if known
- Mid-semester rescheduling → user must re-run this skill; prompt them when drift is detected (see Step 5)

Completion criterion: every session has a resolved calendar date. Cancelled and replacement sessions are explicitly marked, not silently dropped.

### Step 2 — Map Drive folder structure

Access the provided Drive folder. List all files and subfolders. Infer the organisation pattern:
- Organised by week → map folder/file to week number
- Organised by module → map folder/file to module name
- Flat → group by filename keywords (week number, topic name, date in filename)
- Mixed → apply both rules, flag ambiguous files

**Missing file handling:**
- If a session has no matching Drive file, mark it `[no file — will fallback to web search]`
- Do not treat missing files as errors — the brief skill has a fallback chain

Flag files that cannot be mapped under `# Unmapped files` in `schedule.md`.

Completion criterion: every session either has a Drive file mapped or is explicitly marked for fallback. No session is silently unresolved.

### Step 3 — Write schedule.md

Overwrite if it already exists.

```markdown
# Schedule

Generated: {date}
Last drift check: {date}
Next drift check due: {date, 3 weeks from generated}
Semester start: {date}
Drive root: {folder path or URL}

## Modules

### {Module Name} ({CODE})
- Sessions: {day} {time}, {venue}
- Drive path: {path}

## Weekly Calendar

### Week {N} — {Monday date of that week}
- [ ] **{Module}** — {date} {time} | Topic: {inferred from filename or "TBC"}
      Drive: {file link or "[no file — fallback]"}
      Status: {active | cancelled — public holiday | replacement for Week X}
      Pre-class brief due: **{date, day before session at 20:00 local time}**
```

Pre-class brief deadline = the evening before each class (20:00). Monday class → Sunday 20:00.

Completion criterion: `schedule.md` written, every session populated, pre-class deadlines present, drift check dates set.

### Step 4 — Confirm with user

Print a compact summary:

```
Schedule built.
- {N} modules, {N} sessions total
- {N} mapped to Drive files | {N} will use web search fallback
- {N} cancelled sessions | {N} replacement sessions

Pre-class briefs due the evening before each class.
Next timetable drift check: {date}.
```

Ask: "Does anything look wrong or missing?" Apply corrections and re-run affected rows only.

### Step 5 — Drift check (recurring)

When any downstream skill runs, check `schedule.md` for the `Next drift check due` date.
If today is past that date, ask the user before proceeding with the main skill:

> "It's been 3 weeks since your schedule was last synced. Any cancelled classes, replacement sessions, or rescheduling since then?"

- No changes → update `Last drift check` and `Next drift check due` in `schedule.md`, continue
- Changes reported → apply them to affected rows only, update dates, continue
- User unsure → flag in `schedule.md` as `[drift unverified — {date}]` and continue

Do not block the downstream skill. Drift check is a prompt, not a gate.

## Notes

- `schedule.md` is the single source of truth. Never hardcode dates in other skills.
- Missing Drive files are not failures — they trigger the brief skill's fallback chain.
- Mid-semester rescheduling always requires a manual re-run of this skill. The drift check prompts for it but cannot detect it automatically.
