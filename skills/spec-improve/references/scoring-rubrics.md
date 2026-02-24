# Spec Quality Scoring Rubrics

This file defines how to score a specification across universal and type-specific dimensions. Use this during Phase 3 (Audit) of the `spec-improve` skill.

---

## Scoring Overview

Each dimension is scored 0–10. The final score is a weighted sum normalized to 100.

**Grade thresholds:**
| Grade | Score | AI Agent Readiness |
|-------|-------|--------------------|
| A     | 85–100 | READY — agent can proceed autonomously without clarification |
| B     | 70–84  | READY — minor assumptions needed but unlikely to cause failures |
| C     | 50–69  | CONDITIONALLY READY — 1–2 clarifications recommended before handing off |
| D     | 30–49  | NOT READY — agent will make 3+ wrong assumptions, producing unverifiable code |
| F     | 0–29   | NOT READY — spec is too vague or incomplete to use |

---

## Universal Dimensions (all spec types)

These apply to every spec regardless of type. Combined weight: 60% of total score.

### 1. Problem Statement / Context (weight: 12%)

What is being built and why? Does the spec explain the business or user need?

| Score | Criteria |
|-------|----------|
| 9–10  | Clearly states the user/business problem, explains why this solution, links to broader context or goal |
| 7–8   | Problem is stated; motivation is implicit but inferable |
| 4–6   | Describes what to build without explaining why; context is missing |
| 1–3   | Single-line title or vague statement like "add feature X" |
| 0     | Absent |

**Penalty**: Deduct 2 points if the problem statement uses jargon without definition that an agent unfamiliar with the codebase would not understand.

---

### 2. Scope (weight: 8%)

What is explicitly in scope and out of scope?

| Score | Criteria |
|-------|----------|
| 9–10  | In-scope and out-of-scope items are explicitly listed |
| 7–8   | Scope is clearly bounded even if not formally listed |
| 4–6   | Scope can be inferred but is not stated; agent might over- or under-build |
| 1–3   | No scope definition; agent must guess where to stop |
| 0     | Absent |

---

### 3. Acceptance Criteria (weight: 15%)

What conditions must be true for the work to be considered done?

| Score | Criteria |
|-------|----------|
| 9–10  | Each criterion is testable, unambiguous, and independently verifiable. Uses Given/When/Then or equivalent structured format. Covers the happy path and at least one failure case. |
| 7–8   | Criteria are present and mostly testable; some could be more specific |
| 4–6   | Criteria exist but are vague ("it should work correctly", "it should be fast") |
| 1–3   | A single bullet list with no testable conditions |
| 0     | Absent |

**Tip**: A criterion is testable if you can write an automated test against it without asking anyone a question.

---

### 4. Error Handling (weight: 12%)

What happens when things go wrong?

| Score | Criteria |
|-------|----------|
| 9–10  | Each error case is named; behavior for each is specified (user message, HTTP status, retry logic, fallback, logging) |
| 7–8   | Main error cases covered; minor gaps |
| 4–6   | Error handling mentioned in general terms ("return an error if it fails") |
| 1–3   | A single "handle errors appropriately" line |
| 0     | Absent |

**Common errors to check for**: invalid input, auth failure, upstream service failure, rate limiting, empty/null data, timeouts.

---

### 5. Edge Cases (weight: 8%)

What unusual or boundary inputs/states should be handled correctly?

| Score | Criteria |
|-------|----------|
| 9–10  | At least 3–5 specific edge cases enumerated with expected behavior |
| 7–8   | 2–3 edge cases with expected behavior |
| 4–6   | Edge cases mentioned but without specified handling |
| 1–3   | A note that "edge cases should be considered" with no specifics |
| 0     | Absent |

**Examples of edge cases**: empty collections, maximum payload size, concurrent requests, deleted or archived entities, special characters, timezone differences.

---

### 6. Testability (weight: 5%)

Can an agent write automated tests from this spec alone, without asking anyone?

| Score | Criteria |
|-------|----------|
| 9–10  | Every AC has a clear expected outcome; test inputs are specified or derivable; test isolation requirements noted |
| 7–8   | Most ACs are testable; some test setup may require inference |
| 4–6   | Testing is possible but agent must make assumptions about inputs or expected values |
| 1–3   | Spec describes behavior narratively with no verifiable assertions |
| 0     | Untestable as written |

---

## Type-Specific Dimensions

### API Endpoint Spec (additional weight: 40%)

Apply these when the spec type is "API endpoint". Universal dimensions cover 60%.

#### A1. HTTP Method + Path (weight: 8%)
| Score | Criteria |
|-------|----------|
| 9–10  | Exact method (GET/POST/PUT/PATCH/DELETE) and full path with parameter placeholders (e.g. `GET /users/{userId}/orders`) |
| 5–8   | Method or path present but not both clearly stated |
| 0–4   | Implied from context; agent must guess |

#### A2. Path/Query Parameters (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | All parameters listed with name, type, required/optional, and description |
| 5–8   | Parameters listed but types or constraints missing |
| 0–4   | Parameters implied but not documented |

#### A3. Request Body Schema (weight: 8%)
| Score | Criteria |
|-------|----------|
| 9–10  | Full schema with field names, types, constraints (min/max, pattern, enum), required fields, and a complete example |
| 5–8   | Schema partially described; types or constraints missing |
| 0–4   | "See existing schema" with no reference, or absent entirely |
| N/A   | GET/DELETE endpoints with no body — skip this dimension and redistribute weight |

#### A4. Response Schema — Success (weight: 8%)
| Score | Criteria |
|-------|----------|
| 9–10  | HTTP status code, Content-Type, complete response body schema with example |
| 5–8   | Status code and high-level shape described; details incomplete |
| 0–4   | "Returns the created object" with no schema |

#### A5. Response Schema — Errors (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | Each error case maps to a specific HTTP status code with error response body structure and example |
| 5–8   | Some error codes listed; response bodies not specified |
| 0–4   | "Returns appropriate errors" |

#### A6. Authentication and Authorization (weight: 4%)
| Score | Criteria |
|-------|----------|
| 9–10  | Auth mechanism specified (Bearer token, API key, cookie); required roles or permissions named |
| 5–8   | Auth required but mechanism or permission details vague |
| 0–4   | No auth requirements stated; agent must assume |

---

### UI Component Spec (additional weight: 40%)

#### U1. Component States (weight: 10%)
| Score | Criteria |
|-------|----------|
| 9–10  | All states enumerated: default, loading, empty, error, success, disabled, hover, focus, active |
| 5–8   | Main states described; edge states (empty, error) missing |
| 0–4   | Only happy path described |

#### U2. Props / Inputs (weight: 8%)
| Score | Criteria |
|-------|----------|
| 9–10  | All props with type, required/optional, default value, and description |
| 5–8   | Props listed without types or defaults |
| 0–4   | No props documented; agent must infer from mockups |

#### U3. Interaction Model (weight: 8%)
| Score | Criteria |
|-------|----------|
| 9–10  | Every interactive element specifies what happens on click/input/focus/blur/keyboard events |
| 5–8   | Main interactions described; keyboard and focus behavior missing |
| 0–4   | Behavior implied from mockup only |

#### U4. Accessibility Requirements (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | ARIA roles, labels, keyboard navigation, focus management, color contrast requirements stated |
| 5–8   | Accessibility mentioned; specific requirements incomplete |
| 0–4   | No accessibility requirements |

#### U5. Responsive Behavior (weight: 4%)
| Score | Criteria |
|-------|----------|
| 9–10  | Behavior at each breakpoint specified or explicitly defers to a design system token |
| 5–8   | Responsive noted; breakpoints not specified |
| 0–4   | No responsive requirements |

#### U6. Variants and Configuration (weight: 4%)
| Score | Criteria |
|-------|----------|
| 9–10  | All visual/behavioral variants defined with when-to-use guidance |
| 5–8   | Some variants described |
| 0–4   | No variants; single implicit state only |

---

### Backend Feature Spec (additional weight: 40%)

#### B1. Data Models (weight: 10%)
| Score | Criteria |
|-------|----------|
| 9–10  | All entities with fields, types, constraints, relationships, and indexes |
| 5–8   | Models sketched; constraints or relationships missing |
| 0–4   | "Use the existing User model" with no further detail |

#### B2. Business Rules (weight: 12%)
| Score | Criteria |
|-------|----------|
| 9–10  | Each rule stated precisely: conditions, actors, outcomes. No ambiguous "should" language |
| 5–8   | Rules present but with gaps or ambiguous conditions |
| 0–4   | Rules implied from context only |

#### B3. Performance Requirements (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | p50/p95/p99 latency targets, throughput (req/s), data volume expectations, SLA |
| 5–8   | General performance requirement stated ("should be fast") |
| 0–4   | Absent |

#### B4. Security Requirements (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | Data sensitivity level, encryption at rest/transit, audit logging, PII handling, compliance requirements |
| 5–8   | "Must be secure" with some specifics |
| 0–4   | Absent |

#### B5. External Dependencies (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | Every external service/API/library named with version, contract details, failure behavior |
| 5–8   | Dependencies named without contract details |
| 0–4   | Absent |

---

### Bug Fix Spec (additional weight: 40%)

#### G1. Reproduction Steps (weight: 12%)
| Score | Criteria |
|-------|----------|
| 9–10  | Numbered step-by-step repro; environment specified; 100% reproducible from steps alone |
| 5–8   | Steps present but incomplete or environment unspecified |
| 0–4   | "It happens sometimes" or "see screenshot" |

#### G2. Expected vs Actual Behavior (weight: 10%)
| Score | Criteria |
|-------|----------|
| 9–10  | Both clearly stated with specific values, not just descriptions |
| 5–8   | Both stated but imprecise |
| 0–4   | Only one side specified or both vague |

#### G3. Environment (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | OS, browser/runtime version, app version, data state, feature flags all specified |
| 5–8   | Partial environment info |
| 0–4   | Absent |

#### G4. Root Cause Hypothesis (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | Hypothesis stated with supporting evidence; files/functions suspected |
| 5–8   | Hypothesis mentioned without evidence |
| 0–4   | Absent; acceptable for low-severity bugs |

#### G5. Regression Risk (weight: 6%)
| Score | Criteria |
|-------|----------|
| 9–10  | Areas at risk of regression named; existing tests to run/update identified |
| 5–8   | Risk mentioned without specifics |
| 0–4   | Absent |

---

### Data Migration Spec (additional weight: 40%)

#### M1. Source and Target Schema (weight: 12%)
| Score | Criteria |
|-------|----------|
| 9–10  | Source and target schemas fully documented with field mapping table |
| 5–8   | Schemas described; mapping incomplete |
| 0–4   | "Migrate from old format to new format" |

#### M2. Transformation Rules (weight: 10%)
| Score | Criteria |
|-------|----------|
| 9–10  | Every field transformation explicitly defined: type coercions, default values for missing data, null handling |
| 5–8   | Main transforms described; edge transforms missing |
| 0–4   | Absent |

#### M3. Volume and Performance (weight: 8%)
| Score | Criteria |
|-------|----------|
| 9–10  | Row count, estimated duration, batch size, rollback strategy, downtime window |
| 5–8   | Volume stated; performance strategy missing |
| 0–4   | Absent |

#### M4. Validation and Rollback (weight: 10%)
| Score | Criteria |
|-------|----------|
| 9–10  | Post-migration validation checks defined; rollback procedure documented; success criteria measurable |
| 5–8   | Validation mentioned; rollback absent |
| 0–4   | Absent |

---

## INVEST Compliance Check (bonus / penalty, ±5 points)

For user stories, evaluate against the INVEST criteria. This is an additive check on top of the base score.

| Criterion | Check |
|-----------|-------|
| **I**ndependent | Can this be built and shipped without depending on another unfinished story? |
| **N**egotiable | Are implementation details left open, or is every implementation decision prescribed? |
| **V**aluable | Does it deliver user or business value on its own? |
| **E**stimable | Is there enough information for a developer to estimate effort? |
| **S**mall | Can it be completed in one sprint? |
| **T**estable | Can it be verified by automated tests or defined acceptance criteria? |

Add 5 points if all 6 INVEST criteria are met. Deduct 5 points if 3+ criteria fail.

---

## Automatic Disqualifiers (F grade, regardless of score)

Apply an automatic F if any of the following are true:
- The spec is a single sentence or title only
- The spec contains only a link to a Figma/external resource with no text
- The spec consists entirely of bullet points that describe what to do without any how, why, or success conditions
- The spec says "same as [other ticket]" without that ticket being provided
