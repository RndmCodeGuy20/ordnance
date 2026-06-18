---
name: postman-to-openapi
description: >
  Converts Postman collections into fully compliant OpenAPI 3.0 specs (YAML + JSON).
  Use when: user provides a .postman_collection.json path, shares a Postman collection URL,
  mentions "convert Postman to OpenAPI", "generate OpenAPI from Postman", "Postman to Swagger",
  or drops a Postman file with any hint they want API docs or a spec.
---

# Postman → OpenAPI Converter

Convert a Postman collection into a production-quality OpenAPI 3.0.0 specification. Be thorough and precise — missing a single endpoint, parameter, or auth setting is a failure. After converting, audit your own work, then hand the user a clear analysis of everything done.

---

## Phase 1 — Ingest the Postman Collection

**If the input is a URL:** Use WebFetch to download the raw JSON. Follow redirects and parse the response body as JSON.

**If the input is a local file path:** Use the Read tool. If the file is large (>2000 lines), read in chunks using `limit` and `offset`.

**Detect the collection format:**
- Look for `"schema": "...v2.1.0..."` → Collection v2.1
- Look for `"schema": "...v2.0.0..."` → Collection v2.0
- Note the version in the final report.

---

## Phase 2 — Full Extraction (Leave Nothing Behind)

Recursively walk the entire `item` tree. Postman collections can nest folders inside folders — recurse to any depth. For every request, extract:

| Field | How to extract |
|---|---|
| **Tag / folder** | Name of the parent folder |
| **Request name** | `item.name` |
| **HTTP method** | `request.method` |
| **Full URL** | `request.url.raw` |
| **Path segments** | `request.url.path[]` |
| **Query params** | `request.url.query[]` — capture `key`, `value`, `disabled` flag |
| **Headers** | `request.header[]` — skip standard headers unless unusual |
| **Body** | `request.body` — capture `mode` and content |
| **Auth** | `request.auth` or collection-level `auth` if request inherits |

**Variable conversion rules:**
- `{{variable}}` in path segments → OpenAPI path parameter `{variable}`
- `{{baseurl}}` or `{{host}}` → extract into the `servers` array, not a path param
- Hardcoded numeric/string IDs in paths → normalize to `{id}` path parameter; note in report

**Collapsing duplicate paths:**
When multiple Postman requests share the same HTTP method AND normalized path, collapse into a **single OpenAPI operation** with:
- A combined `summary` listing all operations
- A `description` explaining each variant
- Union of all request body examples

Keep a running tally: `total_postman_requests` vs `total_openapi_operations`. Explain every discrepancy in the final report.

---

## Phase 3 — Auth Mapping

Check both collection-level auth and per-request auth overrides. Map to OpenAPI `securitySchemes`:

| Postman auth type | OpenAPI securityScheme |
|---|---|
| `oauth2` | `oauth2` with flows |
| `oauth1` | `http` scheme with description noting OAuth 1.0a |
| `bearer` | `http` with `scheme: bearer` |
| `basic` | `http` with `scheme: basic` |
| `apikey` | `apiKey` with `in` (header/query) and `name` |
| `noauth` | No security applied |

If auth is uniform across all requests, declare it globally. If some requests override auth, apply `security` at the operation level for those.

---

## Phase 4 — Generate the OpenAPI 3.0.0 Spec

### Step 4a — Use the p2o CLI first (fast, reliable)

The `postman-to-openapi` npm CLI (p2o) handles large files well. Try it first:

```bash
# Install if needed
npm install -g postman-to-openapi

# Convert
p2o "/path/to/collection.postman_collection.json" -f "/path/to/output-openapi.yaml"
```

After p2o runs, read the output YAML. **p2o has known issues you MUST fix:**

| p2o Bug | What it produces | Correct OpenAPI |
|---------|-----------------|-----------------|
| **OAuth2 auth** | `type: http, scheme: oauth2` (INVALID) | `type: oauth2` with `flows`, `authorizationUrl`, `tokenUrl`, `scopes` |
| Missing securitySchemes | Omits auth entirely | Add from collection-level auth |
| Disabled query params | Silently drops them | Note as "not converted" in report |
| Non-standard MIME types | Passes through verbatim | Fix to standard types |
| Duplicate operationIds | Generates duplicates | Deduplicate with suffix |

**The OAuth2 bug is critical.** p2o writes `type: http, scheme: oauth2` which is invalid — `http` type only supports `basic` and `bearer`. OAuth2 is its own `type: oauth2` in OpenAPI 3.0 and requires `flows` with grant type, authorization URL, token URL, and scopes. Look up the actual OAuth2 endpoints for the API provider and construct the correct `securitySchemes` entry.

If p2o fails or is unavailable, fall through to Step 4b.

### Step 4b — Manual generation fallback

Build the spec manually. Produce a complete OpenAPI 3.0.0 document:
- `info`: title (from collection name), description, `version: "1.0.0"`
- `servers`: extract `{{baseurl}}` or actual host
- `paths`: one entry per unique (normalized-path + method); per operation include `operationId` (camelCase), `summary`, `description`, `tags`, `parameters`, `requestBody`, and `responses` (at minimum 200, 400, 401, 403, 404, 500)
- `components.securitySchemes`: from Phase 3

### Step 4c — Generate the JSON version

```bash
node -e "const y=require('js-yaml'),fs=require('fs');fs.writeFileSync('out.json',JSON.stringify(y.load(fs.readFileSync('out.yaml','utf8')),null,2))"
```

### Output file naming
- Same directory as the input file (or cwd for URLs)
- `<base-name>-openapi.yaml` and `<base-name>-openapi.json`

---

## Phase 5 — Bug Detection

Flag any of these issues found in the Postman collection:

- **GET requests with bodies** — usually a Postman artifact
- **Non-standard MIME types** — e.g., `application/text` instead of `text/plain`
- **Hardcoded credentials/tokens** — actual API keys in auth fields (vs `{{variable}}` references)
- **Hardcoded timestamps or nonces** — static values in OAuth fields
- **Duplicate path+method combos** — same endpoint defined twice without collapsing
- **Inconsistent variable usage** — most use `{{baseurl}}` but one has a hardcoded host
- **Disabled query params** — present but disabled; note as "not converted"
- **Empty request names** — unnamed requests
- **Missing required variables** — paths reference `{{var}}` with no default

---

## Phase 6 — Diff Mode (only when an existing OpenAPI spec is also provided)

If the user provides an existing OpenAPI spec alongside the Postman collection:

1. Read the existing spec
2. Compare against the freshly generated spec: new endpoints, removed endpoints, modified endpoints, auth changes
3. Bump `info.version` (patch for minor changes, minor for new endpoints, major for removed/breaking)
4. Write a diff report to `<base-name>-openapi-diff.md`

---

## Phase 7 — Quality Verification

Self-audit pass before finalizing:

**Coverage checks:**
- [ ] Every Postman request's (method + normalized path) has a matching operation
- [ ] Every enabled query parameter appears in the spec
- [ ] Every POST/PUT/PATCH with a body has a `requestBody`
- [ ] Every `{param}` in a path has a corresponding parameter declaration with `in: path`

**Correctness checks:**
- [ ] No duplicate `operationId` values
- [ ] No path has the same parameter name twice (rename one if so)
- [ ] Mandatory parameters are marked `required: true`
- [ ] Content-type for query-style POST bodies is `text/plain` not `application/text`

Fix every issue found. If a fix changes something structural, update both YAML and JSON.

---

## Phase 7.5 — Swagger CLI Validation

```bash
# Install if needed
npm install -g @apidevtools/swagger-cli

# Validate
swagger-cli validate "/path/to/output-openapi.yaml"
```

- **Exit 0 + "is valid"** → PASS.
- **Exit non-zero** → Fix each error, re-run until it passes.

Common fixes:

| Error | Fix |
|---|---|
| `Duplicate operationId` | Rename conflicting operationId to be unique |
| `Missing required property: responses` | Add `responses: { '200': { description: OK } }` |
| `must have required property 'schema'` | Add `schema: { type: string }` to parameter |
| `path template variable not found` | Ensure every `{param}` in path has a matching parameter entry |

---

## Phase 8 — Final Analysis Report

```
## Postman → OpenAPI Conversion Report

### Input
- Source: <file path or URL>
- Postman collection format: <v2.0 / v2.1>
- Collection name: <name>

### Conversion Stats
- Total Postman requests found: <N>
- Total OpenAPI operations generated: <N>
- Difference explained: <reason>

### Tags / Folders Converted
<bulleted list of all tags and operation counts>

### Authentication
- Auth type detected: <type>
- Applied as: <global / per-operation>
- securityScheme name: <name>

### Bugs Found in Postman Collection
<numbered list, or "None found">

### Quality Check Results
- Coverage: <PASS / FAIL>
- No duplicate operationIds: <PASS / FAIL>
- No duplicate path params: <PASS / FAIL>
- Required flags correct: <PASS / FAIL>
- Content-type correctness: <PASS / FAIL>

### Swagger CLI Validation
- Result: <PASS / FAIL>
- Rounds needed to fix: <N>

### Output Files
- YAML: <absolute path>
- JSON: <absolute path>
- Diff report: <absolute path, if applicable>

### Notes
<any other observations, caveats, or recommendations>
```

---

## Tips for Large Collections

- **Always try p2o first** — faster and more complete. Focus your effort on the audit: auth mapping, disabled params, MIME types, duplicate operationIds.
- Use Read tool's `limit`/`offset` to page through large Postman files.
- When generating YAML manually, write it in a single Write tool call — partial writes corrupt the file.
- If the collection has 50+ endpoints, prioritize correctness over prettiness.
