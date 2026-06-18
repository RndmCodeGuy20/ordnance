---
name: release-notes
description: >
  Use whenever release notes need to be written for a product feature, fix, or improvement.
  Triggers include: "write release notes for", "draft a release note", "changelog entry",
  "heading and description for the release", or any request to communicate what a feature
  does to an end user in a product update context.
---

# Release Notes

Writes release notes for product features using the Jobs To Be Done framework. Release notes are not feature descriptions — they communicate the value the user gets and the job they can now do.

---

## Core Format

Every release note has exactly two parts:

### Heading
```
[Achieve / Do / Build / Find / ...] X by doing Y
```
- **X** = the value or outcome the user derives
- **Y** = the job they are trying to do

The verb at the start should match the nature of the value. Use "Achieve", "Debug", "Organise", "Control", "Track", "Manage", etc. Never use generic verbs like "Improve" or "Enable" that could apply to anything.

### Description
2–3 sentences that:
1. Briefly describe what the solution does (the mechanism)
2. Connect it to the outcome the user cares about
3. Include a specific detail that makes it concrete (what changed, where it appears, what it replaces)

---

## Tone

> The best release notes feel like a conversation with a knowledgeable friend who's excited to show you something cool, rather than a dry technical document.

- Write as if telling someone what they can now do — not what was built
- Be specific and concrete — name the surface, the interaction, the result
- Avoid passive voice ("has been added", "is now supported") — use active voice
- No filler adjectives — "seamlessly", "powerful", "robust", "easily" add nothing
- No hedging — don't say "helps you" when you can say what it actually does
- Short sentences over long ones

---

## Input Modes

### From a PRD
Extract:
- The core job from the JTBD section
- The primary value from success criteria
- The mechanism from implementation details
- Any specific interaction or surface name from user flows

### From a feature description
Identify:
- What the user was unable to do before
- What they can now do
- Where in the product this happens

### From scratch (just a topic)
Ask two questions before writing:
1. "What could the user not do before this, and what can they do now?"
2. "Where in the product does this show up?"

Do not write without knowing the before/after state.

---

## Writing Process

**Step 1: Identify the job**
What is the user trying to accomplish? Frame it as a verb phrase — "organise data sources by team", "debug event errors faster", "deploy apps without reconfiguring credentials".

**Step 2: Identify the value**
What outcome does completing that job produce? This is X in the heading. It should be a result, not a feature — "faster debugging", "no manual reconfiguration", "organised data layer".

**Step 3: Draft the heading**
Try 2–3 heading variants before settling. Test: if someone reads only the heading, do they understand what changed and why it matters?

**Step 4: Draft the description**
Lead with the mechanism (what changed and where), then connect to the outcome. Include at least one specific detail.

**Step 5: Self-check**

| Check | Question |
|---|---|
| Job clarity | Does the heading name an actual job the user does? |
| Value clarity | Is X a real outcome, not a feature description? |
| Specificity | Does the description name where this appears? |
| Active voice | Any passive constructions to replace? |
| Filler words | Does removing any adjective change the meaning? If not, cut it. |
| Heading self-sufficiency | Does the heading make sense without the description? |
| Tone | Does it read like a person explaining something? |

---

## Examples

### ✅ Strong

**Heading:** Debug easily with event errors integrated into the debugger
**Description:** Event error messages now appear directly in the Debugger, helping developers quickly identify whether the issue is with a page, component, or query. Each message includes the associated action, making debugging faster and efficient.

---

**Heading:** Deploy apps without reconfiguring data sources across environments
**Description:** Data sources can now be branched, committed, and deployed alongside your applications using the same Git workflow. When you deploy an app, its data source configurations move with it — no manual reconfiguration, no production breakage from mismatched dependencies.

---

### ❌ Weak

**Heading:** Improved data source management with folder support
*Problem:* X is a feature description, not a value. Y is vague. Reframe: what does the user achieve?

**Heading:** Data source folders are now available
*Problem:* Changelog entry, not a release note. Describes what was shipped, not what the user can now do.

**Description:** We have added the ability to organise your data sources into folders for better organisation and management.
*Problems:* Passive origin, redundant ("organise" and "organisation"), vague value, no specific surface named.

---

## Output Format

```
Heading: [heading]

Description: [description]
```

If presenting multiple variants:

```
Option A — [angle, e.g. "job-focused"]
Heading: ...
Description: ...

Option B — [angle, e.g. "pain-focused"]
Heading: ...
Description: ...
```

Then give a recommendation: which is stronger and why.
