---
name: architecture-review
description: >-
  Write the mandatory pre-development architecture doc for a feature that
  changes architecture or needs a migration, and prep it for reviewer
  approval. Trigger on: "architecture review", "this needs a migration",
  "write an architecture doc", "before I start building X", or when a
  planned change touches endpoints, permissions, core logic, or the schema.
---

# Architecture Review

Write a short architecture doc — API/Backend impact + Data Management impact — and prep it for sign-off. No development starts until one reviewer approves.

User input: $ARGUMENTS

**Usage:**
```
/architecture-review                    # describe the change in context
/architecture-review new billing sync   # brief description as argument
/architecture-review ./plans/feature.md # read a plan/issue and extract the doc
```

---

## Reviewers (any one approval unblocks dev)

| Reviewer | Slack |
|---|---|
| Kavin | [@Kavin](https://tooljet.slack.com/team/U039MLK86N6) |
| Johnson | [@Johnson](https://tooljet.slack.com/team/U05329VMZE3) |
| Akshay | [@Akshay](https://tooljet.slack.com/team/U02160P4SHK) |
| Midhun G S | [@Midhun G S](https://tooljet.slack.com/team/U02QJE196EL) |

---

## Gate

If the change doesn't touch API endpoints, permissions, core logic, or the DB schema, say so and stop — no doc, no reviewer needed.

Otherwise, confirm (skip what's already clear from context): what the change is, and whether it involves a migration.

---

## Doc contents

Three sections, confirm each with the user before moving to the next:

**Summary** — feature name, one-line description, migration involved (Y/N).

**API / Backend** — impacted/new endpoints (method, path, change type), permission updates, core logic changes in plain language. If none, write "No API/Backend impact" — don't fabricate endpoints.

**Data Management** — schema migrations (tables/columns, backward compatibility) and data migration strategy (backfill/dual-write, rollback plan). If none, write "No schema or data migration required" — never omit silently.

---

## Write the doc

Check `./docs/architecture/` for an existing file for this feature; update it if found, else create `./docs/architecture/<feature-name>.md`. Confirm path first.

```markdown
# Architecture Review: <Name>

> Status: Draft | Pending Review | Approved
> Migration involved: Yes | No

## Summary
## API / Backend
## Data Management

## Approval
- [ ] Approved by: <reviewer> — <date>
```

---

## Approval request

Draft (don't send — Slack isn't authorized in this session):

```
Hi team, requesting architecture review sign-off before starting dev on <feature>.
Doc: <path or link>
@Kavin @Johnson @Akshay @Midhun G S — approval from any one of you unblocks development.
```

Tell the user to send it themselves. Offer to post it directly once Slack is authorized.

---

## Rules

1. Keep the doc lightweight — Summary, API/Backend, Data Management, Approval only.
2. State "none" explicitly rather than omitting a section.
3. Don't fabricate endpoints or migrations.
4. Status stays `Pending Review` until the user confirms an approval.
5. One doc per feature — update in place if one exists.

## Composability

- **Precedes:** `decompose-plan` (wait for approval first)
- **Pairs with:** `design-api` for the full endpoint design once API impact is flagged
- **Followed by:** `grill-me` / `red-team` to stress-test while awaiting approval
