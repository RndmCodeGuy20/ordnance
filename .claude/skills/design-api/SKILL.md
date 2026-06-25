---
name: design-api
description: >-
  Design a REST API contract section-by-section, producing a Markdown design
  document grounded in Google API Design Guide and Stripe conventions. Trigger
  on: "design an API for X", "what endpoints should I create", "review my API
  design", "is this REST API good", "how should I structure this", or when a
  user shares a list of routes and asks for feedback.
---

# API Contract Designer

Design a REST API contract section by section. Claude enforces conventions and catches anti-patterns — you approve each section before moving forward.

User input: $ARGUMENTS

**Usage:**
```
/design-api                         # describe what the API is for in context
/design-api orders service          # brief description as argument
/design-api                         # paste existing routes for review
```

---

## Convention defaults (baked in)

These apply unless you override at the relevant section:

| Convention | Default |
|---|---|
| Protocol | REST / JSON |
| Versioning | Path prefix `/v1/` |
| IDs | Prefixed strings (`usr_abc123`, `ord_xyz456`) — not integers |
| Field names | `snake_case` |
| Boolean fields | `is_` or `has_` prefix (`is_active`, `has_subscription`) |
| Enum values | `SCREAMING_SNAKE_CASE` (`"status": "PENDING_REVIEW"`) |
| Timestamps | ISO 8601 UTC (`"2024-01-15T10:30:00Z"`) |
| Auth | Bearer JWT (`Authorization: Bearer <token>`) |
| Pagination | Cursor-based — `{ data: T[], next_cursor?: string, has_more: boolean }` |
| Error shape | `{ error: { code: string, message: string, param?: string, details?: unknown } }` |
| Idempotency | `X-Idempotency-Key` header on POST and non-idempotent DELETE |

---

## Preamble — 2 context questions (ask both upfront)

Before designing, ask:
1. **Who are the consumers?** Internal services, external developers, mobile clients, third parties?
2. **Public or internal?** Public APIs require stricter stability contracts and explicit versioning.

If both are clear from context, skip and proceed.

---

## Section 1: Scope

Define what the API does and explicitly what it does NOT do. Confirm with user before proceeding.

Output:
```
**API:** <name>
**Purpose:** <one sentence>
**Out of scope:** <what this API will never handle>
**Consumers:** <from preamble>
```

---

## Section 2: Resource model

Apply resource-oriented design — nouns, not verbs.

**Rules:**
- Resources are plural nouns (`orders`, `users`, `payment-methods`)
- Build a hierarchy: `GET /orders/{id}/line-items` not `GET /getOrderLineItems?orderId=X`
- Non-CRUD actions become sub-resources: `POST /orders/{id}/cancel` not `POST /cancelOrder`

**HTTP verb → semantics:**

| Verb | Meaning | Idempotent |
|---|---|---|
| GET | Read | Yes |
| POST | Create | No — use Idempotency-Key |
| PUT | Replace entirely | Yes |
| PATCH | Partial update | No (usually) |
| DELETE | Remove | Yes |

Present the resource map (nouns + relationships) and confirm with user.

---

## Section 3: Endpoints

Build the endpoint table from the resource model:

| Method | Path | Description | Auth | Idempotency-Key |
|---|---|---|---|---|
| GET | /v1/orders | List orders | Required | — |
| POST | /v1/orders | Create order | Required | Required |
| GET | /v1/orders/{id} | Get order | Required | — |
| PATCH | /v1/orders/{id} | Update order | Required | — |
| POST | /v1/orders/{id}/cancel | Cancel order | Required | Required |

Flag any endpoint that looks like an action verb in the path — offer the sub-resource alternative.

Confirm with user before proceeding.

---

## Section 4: Auth

Default: Bearer JWT. Present default and prompt:

> "Auth default is Bearer JWT on all endpoints above. Any endpoints that should be public (no auth)? Any that need a different scheme?"

Override if specified. Note public endpoints explicitly in the table.

---

## Section 5: Request/Response shapes

For each endpoint, define:
- **Path parameters** — name, type, example
- **Query parameters** — name, type, required/optional, example
- **Request body** (POST/PUT/PATCH) — JSON shape with types
- **Response body** — JSON shape with types, wrapped in envelope

**Success envelope:**
```json
// Single resource
{ "data": { "id": "ord_abc123", "status": "PENDING", "created_at": "2024-01-15T10:30:00Z" } }

// Collection
{ "data": [...], "pagination": { "next_cursor": "cursor_xyz", "has_more": true } }
```

**HTTP status mapping:**
| Status | When |
|---|---|
| `200 OK` | Read or update |
| `201 Created` | Creation — include `Location: /v1/orders/ord_abc123` header |
| `204 No Content` | Delete |
| `400 Bad Request` | Invalid request shape or params |
| `401 Unauthorized` | Missing or invalid credentials |
| `403 Forbidden` | Valid credentials, insufficient permissions |
| `404 Not Found` | Resource doesn't exist |
| `409 Conflict` | State conflict (duplicate, already cancelled) |
| `422 Unprocessable Entity` | Valid syntax, failed business validation |
| `429 Too Many Requests` | Rate limited |
| `500 Internal Server Error` | Unexpected server error |

**Rule:** Never return `200 OK` with `"success": false`. HTTP status IS the status.

Confirm shapes per endpoint before proceeding.

---

## Section 6: Error catalogue

Map business errors to HTTP status + error code:

| Scenario | HTTP Status | `error.code` |
|---|---|---|
| Order not found | 404 | `ORDER_NOT_FOUND` |
| Order already cancelled | 409 | `ORDER_ALREADY_CANCELLED` |
| Insufficient stock | 422 | `INSUFFICIENT_STOCK` |
| Invalid currency | 400 | `INVALID_CURRENCY` |

`error.param` = the request field that caused the error (for 400/422).

Confirm with user. Add/remove entries as needed.

---

## Section 7: Pagination (skip if no list endpoints)

Default: cursor-based. Present default and prompt:

> "Pagination default is cursor-based. Override to offset-based only if this list is small (<1000 items), stable, and needs UI page numbers."

Filtering convention:
```
GET /v1/orders?status=PENDING&created_after=2024-01-01&sort=created_at:desc&limit=20&after=cursor_xyz
```

---

## Section 8: Idempotency (skip if no POST/DELETE)

For each non-idempotent endpoint, confirm `Idempotency-Key` header is expected. Server stores key + result for 24h. Repeated requests return cached result without re-executing.

Present which endpoints require it and confirm.

---

## Section 9: Anti-patterns review

Before writing the doc, check the proposed design against each:

| Anti-pattern | Check |
|---|---|
| Verb-based URLs (`/createUser`) | All paths use nouns + HTTP verbs? |
| Leaking internals | No DB column names, internal states, or integer sequence IDs exposed? |
| Inconsistent naming | All fields `snake_case`? All enums `SCREAMING_SNAKE_CASE`? |
| Missing idempotency | All POST/DELETE have Idempotency-Key or are intentionally excluded? |
| `200` with `"success": false` | No endpoint returns 200 on failure? |
| Chatty API | Any logical operation requiring 3+ calls? Consider embedding or batch endpoint. |
| Breaking change without version | Any change to existing contracts? Flag for versioning. |

Flag any hits and propose fixes. Confirm before writing.

---

## Section 10: Open questions

List any decisions deferred to implementation:
- Business rules not yet defined
- Auth edge cases
- Rate limit thresholds
- Any section where an override was noted but not resolved

---

## Write the design doc

Save to `./docs/api/<api-name>-design.md`. Create `./docs/api/` if it doesn't exist.

Structure:
```markdown
# API Design: <Name>

> Status: Draft | Reviewed | Approved
> Consumers: <from preamble>

## Scope
...

## Resource Model
...

## Endpoints
<table>

## Auth
...

## Request/Response Shapes
<per endpoint>

## Error Catalogue
<table>

## Pagination
...

## Idempotency
...

## Open Questions
...
```

---

## Rules

1. **Section-by-section**: get approval after each section. Don't write the full doc until all sections are confirmed.
2. **Conventions are defaults, not mandates**: prompt for overrides at auth, pagination, and versioning sections. Record deviations in the doc.
3. **Anti-patterns check is mandatory**: run Section 9 before writing the file.
4. **No implementation details**: paths, shapes, and error codes only — no handler names, no DB schema.
5. **One doc per API**: if re-running, update the existing file.

## Composability

- **Often preceded by:** `grill-me` on the requirements before designing
- **Often followed by:** `decompose-plan` to break implementation into phases
- **Pairs with:** `postman-to-openapi` to generate a spec from the finished doc
