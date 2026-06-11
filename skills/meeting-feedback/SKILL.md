---
name: meeting-feedback
description: Review a meeting transcript and deliver brutally honest performance feedback (strengths, weaknesses, action plan) in French for one participant — default William. Stores transcript + report on disk and tracks progress across meetings. Use when the user wants feedback on their performance in a meeting or asks to review/analyze a meeting transcript.
allowed-tools: Bash, Read, Write, Glob, Grep
argument-hint: "[person (default: William)] [transcript file path]"
---

# Meeting Feedback

Analyze a meeting transcript and coach one participant (default: **William**) with
**brutally honest**, evidence-based feedback: what worked, what didn't, and exactly how
to do better next time. The goal is **continuous improvement**: every analysis is saved
to disk and compared against previous reports so progress (or stagnation) is visible.

**Read [`RAPPORT.md`](RAPPORT.md) first** — it holds the fixed French report template.
Every report follows it exactly, with the same axes and sections, so reports stay
comparable across meetings.

## Arguments

`$ARGUMENTS` (optional) may carry, in any order:
- a **person name** — the participant to evaluate. Default: `William` (also match
  variants: Will, Willy, W., the user's speaker label, etc.).
- a **transcript** — a file path. If absent, use a transcript pasted in the
  conversation; if there is none, ask the user for it (in French).

## Instructions

### Step 1: Get the transcript

Resolve in this order: file path in `$ARGUMENTS` → transcript pasted in the
conversation → ask the user (in French). Keep the raw text untouched for archiving.

### Step 2: Verify speaker attribution — do NOT trust the labels

Transcription tools (Google Meet, Teams, Zoom, tl;dv, Fireflies…) **regularly mislabel
speakers**. Before judging anything:

1. Map the speaker labels to the target person (name variants, initials, email handle).
2. Sanity-check every segment attributed to the target:
   - **Self-reference**: does the speaker talk about the target in third person, or get
     addressed by the target's name mid-segment? → likely mislabeled.
   - **Adjacency**: someone says *"Will, qu'est-ce que t'en penses ?"* — the next
     segment is probably the target, whatever the label says.
   - **Role consistency**: does the content match the target's role and knowledge?
   - **Continuity**: a segment that finishes another speaker's sentence is often a
     split-attribution artifact.
3. Segments with doubtful attribution are **excluded from all judgments** and listed in
   the report's *Fiabilité de l'attribution* block. **Never base a criticism on a quote
   you are not sure the person said.**
4. If the target barely speaks, check it isn't a labeling artifact before treating low
   airtime as a finding.

### Step 3: Load history

Archive root: `~/Documents/meetings/<person-slug>/` (person-slug = lowercase name,
e.g. `william`).

- Glob `~/Documents/meetings/<person-slug>/*/rapport.md` and read the **3 most recent**
  reports.
- Extract their scorecards and *Plan d'action* items: did the person apply the previous
  feedback in this meeting? Which weaknesses recur?
- No previous report → this one is the baseline.

### Step 4: Analyze the performance

Evaluate the six fixed axes from `RAPPORT.md` (clarté & concision · structure &
préparation · écoute & espace donné · impact & influence · posture & assertivité ·
gestion du temps). Ground the analysis in observable signals:

- **Airtime**: rough share of words vs other participants, longest monologue.
- **Interruptions**: cut others off / got cut off and how they reacted.
- **Questions vs statements**, filler and hedging (*"je pense que peut-être…"*),
  jargon vs audience.
- **Outcomes**: did their interventions produce decisions, clarity, next steps — or
  noise? Who actually closed the decisions?
- **Preparation**: came with numbers/structure, or improvised?

### Step 5: Apply the feedback craft

This skill must itself model how good feedback is given:

- **SBI** (Situation – Comportement – Impact) for every point, strength or weakness.
- **Evidence or it doesn't exist**: every judgment is backed by a verbatim quote from
  the transcript. Never paraphrase a quote into something harsher than what was said.
- **Behavior, not personality**: "tu as coupé Sarah trois fois" — never "tu es
  impatient".
- **Actionable**: every weakness ends with **"👉 À la place :"** — a concrete
  alternative, ideally a rewritten line the person could have actually said.
- **Prioritized**: max **3 weaknesses**, the ones with the highest impact. A laundry
  list teaches nothing.
- **Brutally honest ≠ vague harshness**: no compliment sandwich, no softening — but
  "c'était mauvais" is banned. The standard is: *"tu as répondu à une question oui/non
  par 3 minutes de contexte (citation) — résultat : X a reposé exactement la même
  question après."*
- **No invented flaws**: if the performance was genuinely strong, say so with the same
  standard of evidence. Manufacturing weaknesses to seem tough is as useless as
  flattery.

### Step 6: Write the report

In **French**, following `RAPPORT.md` **exactly** — same sections, same order, same six
scorecard axes every time. Address the person as **tu**.

### Step 7: Persist transcript + report

1. Create `~/Documents/meetings/<person-slug>/<YYYY-MM-DD>-<meeting-slug>/`
   (`mkdir -p`). Date from the transcript if it carries one, else today;
   meeting-slug = short kebab-case meeting topic.
2. Write `transcript.md` (the raw transcript as received) and `rapport.md` (the report).
3. Print the **full report inline** in the response, then a one-line confirmation with
   the saved folder path.

## Rules

- **Language**: skill logic is English, but **all output is French** — the report and
  any question asked to the user.
- **Fixed format**: never restructure the report or change the scorecard axes — the
  whole point is comparability across meetings.
- **Verbatim quotes only** — never fabricate or "improve" a quote.
- **Doubtful attribution → excluded**, and disclosed in the report.
- **Brutal honesty is the contract**: the user explicitly asked for it. Flattery and
  diplomatic vagueness are failure modes here.
