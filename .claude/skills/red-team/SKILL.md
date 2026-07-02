---
name: red-team
description: >-
  Stress-test an engineering plan, PRD, or design doc with 4 adversarial-roleplay
  passes (CIA Tradecraft Primer techniques) before it gets implementation
  approval. Trigger on: "red team this", "stress-test this plan", "kill this
  idea", "what am I missing", "poke holes in this", or when a plan/design is in
  context and the user wants an adversarial pass before committing to it.
---

# Red Team

Run a plan through 4 sequential adversarial passes — Key Assumptions Check, Pre-Mortem, Rival Implementation, On-Call Postmortem — then synthesize into a DEAD/FIXABLE verdict. Generative, not dialogic: each pass produces a full artifact, no back-and-forth. Complements `grill-me` (which resolves a decision tree interactively) rather than replacing it — run this first to kill obviously bad plans fast, then `grill-me` to resolve what survives.

User input: $ARGUMENTS

**Usage:**
```
/red-team                     # plan/design already in context
/red-team ./plans/feature.md  # read plan from file
/red-team 1234                # red-team a GitHub issue
```

Parse the input:
- If empty: assume the plan is in context. If not, ask for a one-paragraph summary — what it is, who it's for, the goal, what success looks like in 6 months.
- If a file path: read the file.
- If a GitHub issue number or URL: fetch with `gh issue view`.

---

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "The user seems confident, I'll go easy" | Confidence isn't evidence. Sycophancy is a failure mode this skill routes around — but so is manufactured doom. Report what's actually there. |
| "I'll skip the Rival Implementation, the assumptions check already found the real issue" | Each pass targets a different blind spot. Skipping one leaves a blind spot uncovered. Run all 4. |
| "I can summarize instead of writing the full pre-mortem" | Vague hypotheticals produce vague answers. Write the full chronological artifact — that's what makes the failure mode concrete. |
| "I found a real gap but I'll call it minor so it lands softer" | Severity is a judgment call about actual likelihood and blast radius, not about being liked. Call it what it is. |
| "Every plan has unverifiable assumptions, so this one must be CRITICAL too" | Absence of proof for a not-yet-built thing is normal, not a defect. Only tag CRITICAL what would actually sink the plan if wrong. |

## Red Flags — STOP

- About to soften a finding because the plan is the user's own work
- About to inflate a finding's severity because a clean pass feels like it didn't try hard enough
- About to skip straight to the synthesis without producing all 4 full artifacts
- About to render a verdict on the user's behalf instead of handing back the graded findings

---

## Process

### 1. Load the plan

Get it into context per the input parsing above.

### 2. Run all 4 passes, straight through, no pause

Produce each artifact in full — these are meant to be read, not summarized.

#### Pass 1: Key Assumptions Check

Roleplay: a red team analyst whose only job is to audit the plan's assumptions — not judge whether it's a good idea.

1. List every assumption the plan depends on, including ones not stated outright. At least 10.
2. Classify each: **LOAD-BEARING** (if wrong, the plan fails outright), **IMPORTANT** (weakened but survives), **MINOR** (barely matters).
3. For each LOAD-BEARING assumption: what specific evidence would prove it wrong, and does that evidence already exist? Missing evidence for something not yet built is normal — only flag it as a real risk if it's *knowable now* and nobody checked.
4. Name any assumption that's actually well-supported. A load-bearing assumption backed by real evidence is a finding too — it's a foundation, not a gap.

#### Pass 2: Pre-Mortem

Roleplay: it's 18 months from now, the plan failed catastrophically — not "underperformed," failed. Write the honest post-mortem, chronologically:

- Early period: the warning signs that were ignored
- Middle period: the decisions that made it worse
- Late period: the point of no return
- End: the collapse and what it cost

Be specific — name the exact mistakes. If the plan already has a safeguard that would catch this failure early (a check, a rollback path, a staged rollout), say so — a caught failure mode isn't a finding, it's evidence the plan is doing its job. End with: `"The root cause was ___."`

#### Pass 3: The Rival Implementation

Roleplay: a senior engineer tasked with proving this approach was the wrong call. 90 days, full access to the codebase, motivated to be right.

- Days 1–30: how they study the plan and find its weakest premise
- Days 31–60: what they'd build differently, and why it's better
- Days 61–90: where the original approach breaks under load, scale, or maintenance
- What the plan is uniquely blind to — and, separately, what it already handles well that a rival couldn't actually exploit

Name concrete technical tactics, not vague strategy. If 90 days of trying doesn't surface a real exploit, say that plainly instead of stretching for one. End with: `"The weakness that proves me right is ___."` (or, if none survive scrutiny: `"No exploit found — the approach holds under adversarial review."`)

#### Pass 4: The On-Call Postmortem

Roleplay: the engineer paged at 3am, 6 months from now, because of a decision made in this plan.

Write the incident writeup: what broke, what the on-call engineer had to piece together under pressure, why the plan didn't see this coming, what they wish had been done differently. Be specific about the failure mode — a bad abstraction, an unhandled edge case, a scaling assumption, an ops burden nobody owns.

End with: `"The single decision that caused this was ___."`

### 3. Synthesize

Grade each finding from the 4 passes by actual likelihood × blast radius — not by whether the pass "found something":

- **CRITICAL**: would sink the plan or cause a serious incident, and is plausible given what's actually known
- **MODERATE**: real risk, but survivable or cheap to mitigate
- **MINOR**: technically true but low likelihood or low impact
- **NON-ISSUE**: findings from the passes above that turned out to be already-handled or well-supported — these count too

```
## Red Team Report

**Critical:** <findings that would actually sink this, with source pass>
**Moderate:** <real but manageable risks, with source pass>
**Minor / already handled:** <the well-supported assumptions, caught failure
modes, and exploits that didn't hold up — what's actually solid>

**Pattern:** <one line: e.g. "0 critical, 2 moderate, both cheap to fix" or
"3 critical, all pointing at the same load-bearing assumption">
```

Do not compute or assert a DEAD/FIXABLE verdict — that call belongs to the user, who has context (timeline, risk tolerance, what's non-negotiable) this process doesn't. End by handing it back: state the pattern, then ask how they want to weigh it against the plan going forward.

---

## Rules

1. **All 4 passes, every time.** Each targets a different blind spot; skipping one leaves it uncovered.
2. **Full artifacts, not summaries.** The specificity is the point — vague inputs produce vague findings.
3. **Calibrate, don't manufacture.** Route around sycophancy without swinging to guaranteed pessimism — a clean pass with real non-issues in it is a valid, honest outcome, not a sign the passes weren't run hard enough.
4. **Every finding must be traceable.** Each item in the synthesis maps to a specific line from one of the 4 passes — no free-floating opinions, and no computed verdict standing in for the user's own judgment call.

## Composability

- **Often followed by:** `grill-me` to resolve what survives, then `decompose-plan`
- **Use before:** implementation gets approved — pairs with a "never implement without an approved plan" workflow
- **Standalone use:** any plan, PRD, or design doc, not just code-shaped ones
