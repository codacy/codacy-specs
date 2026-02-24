# Spec Rewrite Templates

Use these templates when rewriting a spec in Phase 5 of the `spec-improve` skill. Select the template matching the detected spec type. Preserve all good original content; add structure around it.

Placeholders in `[BRACKETS]` must be filled from original content or clarification answers. Use `[TO BE DEFINED]` for sections the user skipped.

---

## Universal Header Template

Every spec — regardless of type — should open with this header block.

```markdown
## Overview

**What**: [One sentence: what is being built]
**Why**: [One sentence: the user/business problem this solves]
**Who**: [Primary user or system actor affected]

## Scope

**In scope:**
- [Item 1]
- [Item 2]

**Out of scope:**
- [Item 1]
- [Item 2]

## Background / Context

[Optional: links to related issues, previous decisions, design docs, Figma links, ADRs]
```

---

## API Endpoint Template

```markdown
## Overview

**What**: [One sentence describing the endpoint]
**Why**: [Business or user need this endpoint serves]
**Who**: [Caller: frontend client, internal service, third-party, etc.]

## Scope

**In scope:** [What this endpoint does]
**Out of scope:** [What it does not do, to prevent scope creep]

---

## Endpoint Definition

| Field | Value |
|-------|-------|
| Method | `[GET / POST / PUT / PATCH / DELETE]` |
| Path | `[/resource/{id}/sub-resource]` |
| Auth | `[Bearer token / API key / None]` |
| Required roles | `[admin / user / service-account / None]` |
| Rate limit | `[N requests per minute per user / None]` |

---

## Parameters

### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `[param]` | `string` | Yes | [Description] |

### Query Parameters

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `[param]` | `string` | No | `[default]` | [Description] |

---

## Request Body

**Content-Type**: `application/json`

```json
{
  "[field]": "[type] — [description] — [constraints: min/max/pattern/enum]",
  "[field]": "[type] — [description] — required"
}
```

**Example request:**
```json
{
  "[field]": "[example value]",
  "[field]": "[example value]"
}
```

**Validation rules:**
- `[field]` must be [constraint]
- `[field]` is required when [condition]

---

## Response

### Success

**Status**: `[200 / 201 / 204]`
**Content-Type**: `application/json`

```json
{
  "[field]": "[type] — [description]"
}
```

**Example response:**
```json
{
  "[field]": "[example value]"
}
```

### Errors

| Status | Code | When | Response body |
|--------|------|------|---------------|
| `400` | `VALIDATION_ERROR` | [Invalid input condition] | `{"error": "VALIDATION_ERROR", "message": "[description]", "fields": [...]}` |
| `401` | `UNAUTHORIZED` | Missing or invalid token | `{"error": "UNAUTHORIZED", "message": "Authentication required"}` |
| `403` | `FORBIDDEN` | Insufficient permissions | `{"error": "FORBIDDEN", "message": "[description]"}` |
| `404` | `NOT_FOUND` | [Resource] does not exist | `{"error": "NOT_FOUND", "message": "[description]"}` |
| `409` | `CONFLICT` | [Conflict condition] | `{"error": "CONFLICT", "message": "[description]"}` |
| `429` | `RATE_LIMITED` | Request rate exceeded | `{"error": "RATE_LIMITED", "message": "Retry after [N] seconds"}` |
| `500` | `INTERNAL_ERROR` | Unexpected server error | `{"error": "INTERNAL_ERROR", "message": "An unexpected error occurred"}` |

---

## Acceptance Criteria

- [ ] Given [precondition], when [action], then [expected outcome]
- [ ] Given [precondition], when [invalid input], then [error response with status and body]
- [ ] [Additional criteria]

## Edge Cases

- **[Edge case name]**: [Expected behavior]
- **[Edge case name]**: [Expected behavior]
- **Empty result set**: Returns `200` with `{"data": [], "total": 0}` (not 404)
- **Concurrent requests**: [Behavior — idempotency key / last-write-wins / reject]

## Performance

- Expected p95 latency: [TO BE DEFINED]
- Expected request volume: [TO BE DEFINED]
```

---

## UI Component Template

```markdown
## Overview

**What**: [Component name and one-line description]
**Why**: [User need or UX goal]
**Where used**: [Pages or contexts where this component appears]

## Scope

**In scope:** [What the component does]
**Out of scope:** [What it does not do]

---

## Props / Interface

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `[prop]` | `[type]` | Yes/No | `[default]` | [Description] |
| `onSubmit` | `(data: FormData) => void` | Yes | — | Called on successful form submission |

---

## States

| State | Description | Visual |
|-------|-------------|--------|
| Default | [Description of default appearance] | [link or description] |
| Loading | [Behavior while async action in progress] | Spinner replaces content / skeleton |
| Empty | [What renders when there is no data] | [Description] |
| Error | [What renders on failure] | [Error message placement and style] |
| Success | [Post-action state if applicable] | [Description] |
| Disabled | [When and how disabled state appears] | [Description] |

---

## Interactions

| Trigger | Element | Action |
|---------|---------|--------|
| Click | [Element] | [What happens] |
| Focus | [Element] | [What happens] |
| Blur | [Element] | [Validation / what happens] |
| Enter key | [Form / field] | [Submit / next field] |
| Escape key | [Modal / dropdown] | Close |
| [Custom] | [Element] | [Action] |

---

## Variants

| Variant | When to use | Key differences |
|---------|-------------|-----------------|
| `[variant-name]` | [Use case] | [Differences from default] |

---

## Accessibility

- **ARIA role**: `[role or "none"]`
- **aria-label**: `[Value or source]`
- **Keyboard navigation**: [Tab order and keyboard shortcuts]
- **Focus management**: [Where focus goes after action]
- **Screen reader announcement**: [What is announced on state changes]
- **Color contrast**: Minimum 4.5:1 for text, 3:1 for UI components (WCAG AA)
- **Motion**: Respects `prefers-reduced-motion`

---

## Responsive Behavior

| Breakpoint | Behavior |
|------------|----------|
| Mobile (< 768px) | [Layout change] |
| Tablet (768–1024px) | [Layout change] |
| Desktop (> 1024px) | [Default layout] |

---

## Acceptance Criteria

- [ ] Component renders in all states listed above
- [ ] All interactions produce the specified outcomes
- [ ] Component is keyboard-navigable per the interactions table
- [ ] ARIA attributes are present and correct (verified with axe or equivalent)
- [ ] All variants render correctly
- [ ] Responsive layout matches specification at each breakpoint

## Edge Cases

- **[Edge case]**: [Expected behavior]
- **Very long text**: [Truncation or wrapping behavior]
- **RTL layout**: [TO BE DEFINED]
```

---

## Backend Feature Template

```markdown
## Overview

**What**: [Feature name and one-line description]
**Why**: [Business problem or user need]
**Who**: [Teams or systems affected]

## Scope

**In scope:** [What this feature includes]
**Out of scope:** [What it explicitly excludes]

---

## Data Models

### [Entity Name]

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | `uuid` | PK, not null | Primary identifier |
| `[field]` | `[type]` | `[nullable/not null/unique/indexed]` | [Description] |

**Relationships:**
- [Entity] has many [Entity] via `[foreign key]`
- [Entity] belongs to [Entity] via `[field]`

**Indexes:**
- `[field]` — for [query pattern]

---

## Business Rules

1. **[Rule name]**: [Precise condition] → [Outcome]. Example: "When a user exceeds their plan limit, block the request and return [error]."
2. **[Rule name]**: [Condition] → [Outcome]
3. **[Rule name]**: [Condition] → [Outcome]

---

## External Dependencies

| Dependency | Version / Contract | Failure behavior |
|------------|-------------------|-----------------|
| [Service name] | [API version or endpoint] | [What to do if unavailable: retry, fallback, fail-fast] |

---

## Performance Requirements

- **Expected data volume**: [rows / requests per day]
- **p95 latency target**: [N ms]
- **Throughput**: [N requests/sec]
- **Batch size** (if applicable): [N records per job]

---

## Security Requirements

- **Data sensitivity**: [Public / Internal / Confidential / PII]
- **Encryption**: [At rest: yes/no] [In transit: yes/no]
- **Audit logging**: [What events to log and where]
- **PII handling**: [TO BE DEFINED]
- **Compliance**: [GDPR / SOC2 / HIPAA / None]

---

## Acceptance Criteria

- [ ] [Business rule 1] is enforced and returns [expected outcome]
- [ ] [Business rule 2] is enforced
- [ ] Performance target is met under [load conditions]
- [ ] Audit log entries are created for [events]
- [ ] [Dependency failure] results in [expected degraded behavior]

## Edge Cases

- **[Edge case]**: [Expected behavior]
- **Concurrent writes to same record**: [Locking strategy / expected behavior]
- **Empty dataset**: [Expected behavior]
```

---

## Bug Fix Template

```markdown
## Overview

**Summary**: [One sentence: what is broken]
**Severity**: [Critical / High / Medium / Low]
**Reported by**: [User segment or specific reporter]
**Affects**: [Version, environment, feature flag]

---

## Reproduction Steps

1. [Step 1 — be specific: exact URL, exact input, exact state]
2. [Step 2]
3. [Step 3]
4. Observe: [What actually happens]

**Reproduction rate**: [Always / Intermittent (~N% of the time) / Only under [condition]]

---

## Expected vs Actual Behavior

**Expected**: [Precisely what should happen — specific values, messages, or states]

**Actual**: [Precisely what happens instead — error text, wrong value, blank screen, etc.]

---

## Environment

| Field | Value |
|-------|-------|
| OS | [macOS 14.x / Windows 11 / Ubuntu 22.04] |
| Browser / Runtime | [Chrome 121 / Node 20.x / iOS 17] |
| App version | [v1.2.3 / commit SHA] |
| Feature flags | [flag-name: enabled/disabled] |
| User state | [Logged in as admin / Free plan / Fresh account] |
| Data state | [Empty account / Account with N records] |

---

## Root Cause Hypothesis

[Optional but valuable: where in the code the issue likely originates]

Suspected: `[file path or module name]` — [reason for suspicion]

---

## Fix Acceptance Criteria

- [ ] Reproduction steps no longer reproduce the bug
- [ ] Expected behavior is now observed
- [ ] [Related edge case] is also fixed / verified unaffected
- [ ] Existing tests pass
- [ ] New regression test covers this scenario

## Regression Risk

**Areas at risk**: [List of features or code paths that might be affected by the fix]

**Tests to verify**: [Specific test files or test names to run after the fix]

---

## Gherkin Acceptance Criteria (optional)

```gherkin
Given [precondition]
When [the action from reproduction step N]
Then [expected behavior]
And [additional assertion]

Scenario: [Edge case name]
Given [precondition]
When [condition]
Then [expected behavior]
```
```

---

## Data Migration Template

```markdown
## Overview

**What**: [Describe what data is being migrated and from/to where]
**Why**: [Business reason for migration]
**Estimated records**: [N rows / N GB]
**Deadline**: [Date or sprint]

## Scope

**In scope**: [Data sets included in this migration]
**Out of scope**: [Data sets explicitly excluded]

---

## Source and Target Schema

### Source Schema: `[table/collection/service]`

| Field | Type | Notes |
|-------|------|-------|
| `[field]` | `[type]` | [Description] |

### Target Schema: `[table/collection/service]`

| Field | Type | Notes |
|-------|------|-------|
| `[field]` | `[type]` | [Description] |

### Field Mapping

| Source field | Target field | Transformation |
|-------------|-------------|----------------|
| `[source]` | `[target]` | Direct copy |
| `[source]` | `[target]` | [Transform description, e.g. "Convert UNIX timestamp to ISO 8601"] |
| N/A | `[target]` | Default value: `[value]` |
| `[source]` | N/A | Drop — not migrated |

---

## Transformation Rules

1. **Null handling**: [How null values in source are handled in target]
2. **Type coercions**: [e.g. "string '123' → integer 123"]
3. **Default values**: [Fields that don't exist in source get default `[value]`]
4. **Deduplication**: [How duplicates are detected and resolved]
5. **Encoding**: [Character set, timezone normalization]

---

## Execution Plan

- **Batch size**: [N records per batch]
- **Estimated duration**: [N hours]
- **Downtime window**: [Required downtime / zero-downtime approach]
- **Rollback trigger**: [Condition that triggers rollback, e.g. ">1% error rate"]
- **Rollback procedure**: [Steps to undo the migration]

---

## Validation

**Post-migration checks:**
- [ ] Record count in target matches source (tolerance: [±N%])
- [ ] Spot-check [N] records for data integrity
- [ ] [Application-level check]: [Description]
- [ ] No new errors in application logs for [N minutes] after migration

**Success criteria**: [Specific, measurable conditions that define success]
```
