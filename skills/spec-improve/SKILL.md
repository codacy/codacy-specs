---
name: spec-improve
description: "This skill should be used when the user wants to 'audit a spec', 'improve a spec', 'review a ticket', 'improve this Jira issue', 'rewrite this story', 'check if this spec is good', 'make this ticket clearer', 'improve this Linear issue', or mentions that a spec needs better acceptance criteria, error handling, or clarity for AI agents. It should ALSO be used as a guardrail before starting implementation work on any ticket, story, or spec — for example when the user says 'start working on PROJ-123', 'implement this ticket', 'pick up the next item', or when an agent is about to begin coding from a specification. In this guardrail context, use Quick Check Mode."
tools: Read, Edit, Write, Glob, Grep, Bash
---

# spec-improve

Audit any specification for AI-agent readiness and optionally rewrite it to be clear, testable, and unambiguous. Works with Jira issues, Linear issues, and local spec files.

An **AI-agent-ready spec** leaves no room for guessing: it states what to build, defines inputs and outputs, specifies error handling, enumerates edge cases, and provides acceptance criteria an agent can verify autonomously.

---

## Mode Selection

Determine which mode to use before doing anything else.

**Full Audit Mode** — use when the user explicitly asks to audit, improve, review, or rewrite a spec. Runs all 6 phases (fetch → type detect → audit → clarify → preview → write back).

**Quick Check Mode** — use when the trigger is about to start work: "implement this", "start on PROJ-123", "pick up the next ticket", or an agent transitioning from planning to coding. Runs a lightweight check and presents a decision point. See the [Quick Check Mode](#quick-check-mode) section below.

---

## Quick Check Mode

Use this mode when the skill is triggered as a guardrail before starting implementation — not as an explicit audit request.

### Quick Check Flow

1. Fetch the spec using the same source detection logic as Full Audit Mode.
2. Classify the spec type.
3. Score only the **critical universal dimensions**: problem statement, acceptance criteria, error handling, testability. Skip type-specific dimensions to keep it fast.
4. Output the compact quick check report (see format below).
5. Present exactly two options and wait for the user's choice.

### Quick Check Report Format

```
⚡ Pre-work spec check: <ID or filename>
Grade: <A/B/C/D/F> (<N>/100) — <READY / CONDITIONALLY READY / NOT READY>

<One sentence explaining the readiness verdict.>

Top gaps:
  • [Gap 1 — e.g. "missing acceptance criteria"]
  • [Gap 2 — e.g. "no error handling defined"]
  • [Gap 3 — e.g. "request schema absent"]   ← omit if fewer than 3 gaps

What would you like to do?
  [1] Review and improve — I'll ask clarifying questions and rewrite the spec before we proceed
  [2] Continue anyway — skip the review and start implementation
```

If the grade is **A or B**, skip the options entirely and output:

```
⚡ Pre-work spec check: <ID or filename>
Grade: <A/B> (<N>/100) — READY

This spec is clear enough to implement. Proceeding.
```

### After the Choice

- **Option 1 (Review and improve)**: Switch to Full Audit Mode starting at Phase 3 (Audit) — the spec is already fetched. Run the full quality report, clarification loop, preview, and write-back.
- **Option 2 (Continue anyway)**: Acknowledge and stand down. Do not mention the spec quality again unless the user asks. Example: "Understood. Continuing with implementation."

---

## Source Detection

Identify where the spec lives before doing anything else.

**Jira issue**: The user mentions a key matching `[A-Z]+-[0-9]+` (e.g. `PROJ-123`, `ENG-42`, or a full Jira URL). Fetch with the Atlassian MCP tool `atlassian:getJiraIssue`.

**Linear issue**: The user mentions a Linear issue ID (e.g. `ENG-123`, `ABC-456`, or a full Linear URL). Fetch with `linear:<read_tool_name>` — verify the exact tool name from your Linear MCP server configuration.

**GitHub issue**: The user mentions a GitHub issue URL (`github.com/owner/repo/issues/123`) or a `#123` reference while a GitHub repo is in context. Extract `owner`, `repo`, and `issue_number`. Fetch with the GitHub MCP tool `github:get_issue`.

**Local file**: The user provides a file path or says "this spec" while a file path is in context. Read with the `Read` tool.

**Inline spec**: The user pastes spec text directly into the message. Use the pasted content as-is.

If source is ambiguous, ask: "Is this a Jira issue, Linear issue, GitHub issue, or a local file? Please share the issue key, URL, or file path."

---

## Phase 1: Fetch the Spec

### Jira
```
atlassian:getJiraIssue(issueKey: "PROJ-123")
```
Extract: `summary` (title), `description` (body), `issuetype` (Story/Bug/Task/Epic).

If the tool is unavailable or returns an error, inform the user: "Atlassian MCP is not connected. Please paste the spec content directly or check your MCP configuration."

### Linear
Use `linear:<read_tool_name>` to fetch the issue by ID or URL. Verify the exact tool name from your Linear MCP server configuration.
Extract: title, description, issue type/label.

If unavailable, fall back gracefully: "Linear MCP is not connected. Please paste the spec content directly."

### GitHub
```
github:get_issue(owner: "org", repo: "repo-name", issue_number: 123)
```
Extract: `title`, `body` (description), `labels` (for type detection).

If the user provided a full URL, parse `owner`, `repo`, and `issue_number` from it. If only `#123` was given, infer `owner` and `repo` from the current git remote (`git remote get-url origin`) or ask the user to confirm.

If the tool is unavailable: "GitHub MCP is not connected. Please paste the spec content directly or check your MCP configuration."

### Local file
```
Read(file_path: "<path>")
```
If the file does not exist, ask the user to confirm the path.

---

## Phase 2: Detect Spec Type

Classify the spec into one of these types based on title, labels, and content keywords:

| Type | Signals |
|------|---------|
| **API endpoint** | "endpoint", "REST", "HTTP", "GET/POST/PUT/DELETE", "request", "response", "schema" |
| **UI component** | "component", "page", "screen", "modal", "form", "button", "UI", "UX", "design" |
| **Backend feature** | "service", "worker", "job", "queue", "database", "migration", "business logic" |
| **Bug fix** | "bug", "fix", "error", "crash", "broken", "regression", issue type = Bug |
| **Data migration** | "migrate", "migration", "import", "export", "ETL", "transform" |
| **Generic / other** | Anything that doesn't match the above |

State the detected type clearly: "I've classified this as an **API endpoint** spec."

Consult `references/scoring-rubrics.md` for the full rubric for each type.

---

## Phase 3: Audit

### Before Scoring: Load Context

Run these two steps before applying any rubric scores. They prevent false positives and reduce noise.

**1. Project context** — Search the working directory for Claude instruction files using Glob:
- `CLAUDE.md`, `**/CLAUDE.md`
- `AGENTS.md`, `**/AGENTS.md`
- `.claude/**/*.md`
- `docs/**/*guidelines*`, `docs/**/*conventions*`, `docs/**/*standards*`

Read any files found and extract two things:

- **Terminology**: any term, acronym, or service name defined in these files must not be flagged as undefined during the audit. Treat it as fully understood.
- **Conventions**: API standards, pagination rules, error format standards, auth patterns, naming conventions. If a project convention covers a gap that would otherwise lose points, score that dimension as covered and cite the source:

```
✅  Pagination behavior    8/10
    ↳ covered by project conventions (CLAUDE.md)
```

If no instruction files are found, proceed without project context.

**2. REST defaults (API specs only)** — Do not deduct points for omitting universally-implied REST defaults:
- HTTP 200 + `application/json` for GET, PUT, PATCH success responses
- HTTP 201 + `application/json` for POST responses that create a resource
- HTTP 204 (no body) for DELETE responses

Only flag HTTP status codes and Content-Type when: the correct code is ambiguous (e.g. 200 vs 201 for a POST), a non-JSON response type is possible, or it is an error response. Error response codes and bodies should always be specified regardless of whether they are "standard".

---

Score the spec against the quality rubric. Apply the **universal dimensions** to every spec. Then apply the **type-specific dimensions** for the detected type.

Consult `references/scoring-rubrics.md` for scoring criteria and weights.

### Quality Report Format

Output the report in this exact format:

```
───────────────────────────────────────────────────────────────
  Spec Quality Report: <ID or filename>
  Type: <API endpoint / UI component / Bug fix / ...>
  Score: <N>/100  (Grade: <A/B/C/D/F>)
───────────────────────────────────────────────────────────────
  UNIVERSAL DIMENSIONS

  ✅  Problem statement         8/10
  ✅  Scope                     7/10
  ⚠️  Acceptance criteria       3/10
      ↳ present but not independently testable
  ❌  Error handling            0/10
      ↳ missing
  ❌  Edge cases                0/10
      ↳ missing
  ✅  Testability               6/10

  TYPE-SPECIFIC DIMENSIONS (<type>)

  ✅  HTTP method + path        10/10
  ⚠️  Request schema            3/10
      ↳ params listed but not typed or constrained
  ⚠️  Response schema           6/10
      ↳ example given, enum values undefined
  ❌  Auth requirements         0/10
      ↳ missing
  ⚠️  Error response codes      2/10
      ↳ status codes listed, response bodies absent
───────────────────────────────────────────────────────────────
  Agent Readiness: NOT READY
  An AI agent working from this spec will make 4+ wrong 
  assumptions and produce unverifiable code.
───────────────────────────────────────────────────────────────
```

**Formatting rules — follow exactly:**
- No vertical borders. Structure comes from horizontal rules (`────`) and consistent 2-space indentation on every content line.
- Status icons: always 2 spaces after every icon — `✅  `, `❌  `, `⚠️  ` — same for all, no exceptions.
- Annotations: every dimension scoring below 9/10 gets an annotation on the next line, indented with 6 spaces + `↳ <short explanation>` — never inline after the score.
- Omit the annotation line for dimensions scoring 9–10.

**Grade thresholds**: A = 85–100, B = 70–84, C = 50–69, D = 30–49, F = 0–29.

**Agent Readiness**:
- A/B: READY — agent can work autonomously
- C: CONDITIONALLY READY — minor clarifications needed
- D: NOT READY — significant gaps will cause wrong assumptions
- F: NOT READY — spec is too vague to use

After the report, ask: "Would you like me to rewrite this spec to address these gaps? (yes/no)"

If the user says no, stop here. Offer to answer questions about the gaps instead.

---

## Phase 4: Clarification

Before rewriting, collect information for each **missing critical section** (score 0/10). Ask one question at a time. Do not proceed to the next question until the user answers.

**Critical sections** (always ask if missing):
- Acceptance criteria
- Error handling

**Important sections** (ask if score < 4):
- Edge cases
- Type-specific required fields (e.g. request/response schema for API specs)

**Question format:**
```
I noticed this spec has no [section name].

[One focused question about what's needed.]

(Type your answer, or say "skip" to mark it as [TO BE DEFINED].)
```

If the user says "skip", "don't know", or "move on": record `[TO BE DEFINED]` for that section and continue.

Do not ask about sections that already have good coverage (score ≥ 7).

---

## Phase 5: Preview and Confirm

Rewrite the full spec incorporating:
1. All original content (preserve what was good)
2. Answers collected during clarification
3. Structural improvements using templates from `references/spec-templates.md`
4. `[TO BE DEFINED]` placeholders for skipped sections

Present the rewritten spec as a clean, formatted document — not a diff. Specs are prose, not code.

```
Here's the rewritten spec. Review it carefully before I apply it.

---
[Full rewritten spec here]
---

Confirm to apply this to <Jira PROJ-123 / Linear ENG-42 / GitHub #123 / spec.md>, or say "edit" to request changes.
```

If the user requests changes, apply them and show the updated preview again. Repeat until confirmed.

**Never write back without explicit confirmation.**

---

## Phase 6: Write Back

Apply the confirmed spec to its source.

### Jira
```
atlassian:updateJiraIssue(issueKey: "PROJ-123", description: "<rewritten spec>")
```
Pass `description` as a plain markdown string — not an ADF object. The MCP tool converts markdown to ADF internally.

Confirm success: "PROJ-123 has been updated in Jira."

### Linear
Use `linear:<update_tool_name>` with the issue ID and new description. Verify the exact tool name from your Linear MCP server configuration.
Confirm success: "The Linear issue has been updated."

### GitHub
```
github:update_issue(owner: "org", repo: "repo-name", issue_number: 123, body: "<rewritten spec>")
```
Pass `body` as a plain markdown string. GitHub renders markdown natively — no format conversion needed.

Confirm success: "Issue #123 has been updated in GitHub."

### Local file
```
Edit(file_path: "<path>", old_string: "<entire original content>", new_string: "<rewritten spec>")
```
Confirm: "`spec.md` has been updated."

### Inline / no source
Output the final spec for the user to copy. Offer to save it to a file:
"Would you like me to save this to a file? If so, provide a filename."

---

## Additional Resources

Consult these reference files throughout the workflow:

- `references/scoring-rubrics.md` — Full scoring criteria for all spec types and dimensions. Required reading before Phase 3.
- `references/spec-templates.md` — Section-by-section templates for rewriting. Use in Phase 5.
- `references/spec-examples.md` — Before/after transformation examples. Use for calibration and when unsure how to rewrite a section.

---

## Optional Modes

### Gherkin Mode

If the user asks for "Gherkin acceptance criteria" or "Given/When/Then format", write acceptance criteria using the Gherkin syntax:

```gherkin
Given [precondition that must be true before the action]
When [the specific action or event]
Then [the observable expected outcome]
And [additional assertion, if needed]
```

Apply this format to all acceptance criteria in the rewritten spec, including error cases:

```gherkin
Scenario: Invalid input
Given a user is authenticated
When they submit the form with an empty required field
Then the API returns HTTP 422
And the response body contains {"error": "VALIDATION_ERROR", "fields": ["fieldName"]}
```

Gherkin mode does not change the scoring rubric — it only changes the format of the acceptance criteria section in the output.

### Audit-Only Mode

If the user says "just audit" or "don't rewrite", run Phases 1–3 only. Output the quality report and stop. Do not ask about rewriting.

---

## Tips for Good Rewrites

**On acceptance criteria**: Every criterion should be independently verifiable by a test. The pattern "Given [state], when [action], then [outcome]" is always a good structure even without full Gherkin mode.

**On error handling**: Push for specificity. "Returns an error" is not enough. Ask: what HTTP status? What error code in the body? Is it retryable? Does it log? Does it alert?

**On scope**: If the original spec does not define what is out of scope, scan the content for anything ambiguous and add an explicit out-of-scope bullet. Undefined scope is one of the most common causes of scope creep in AI-generated code.

**On edge cases**: A reliable heuristic is to ask: what happens when the primary entity is missing, empty, at maximum size, or deleted? These four questions surface the most common uncovered edge cases.

**On language**: Replace all instances of "should", "must", "may", "could", and "might" with unambiguous declarative language: "returns", "accepts", "rejects", "stores". Hedged language becomes ambiguous instructions for an agent.

**On fabrication**: When in doubt, use `[TO BE DEFINED]`. A spec with honest placeholders is safer than a spec with plausible-sounding invented details. An AI agent will try to implement `[TO BE DEFINED]` by asking; it will silently implement a fabricated spec incorrectly.

---

## Behavioral Rules

- **Never fabricate missing context.** If a critical section is absent, ask — don't invent.
- **Ask one question at a time.** Do not present a list of questions.
- **Never write back without confirmation.** Always show a preview first.
- **Preserve good content.** Rewriting means improving structure and filling gaps, not replacing everything.
- **Respect "skip".** When the user skips a question, insert `[TO BE DEFINED]` and move on.
- **Stay source-aware.** Know where the spec came from and write back to the same source.
- **Fail gracefully on MCP errors.** If Atlassian or Linear MCP tools are unavailable, fall back to asking the user to paste content.
- **Be honest about grades.** Do not soften scores. A D-grade spec should be called a D-grade spec.
- **Hedged language is a red flag.** When reviewing a spec, every "should", "may", "might", or "could" is a candidate for a clarification question.
- **Length is not quality.** A 100-word spec can be A-grade. A 1,000-word spec can be D-grade. Score the content, not the volume.
- **Suggest CLAUDE.md for false positives.** If you flagged a term or convention during the audit and the user says it is already known or defined, tell them: "Add it to your `CLAUDE.md` under a Terminology or Conventions section and it won't be flagged in future audits." Do not auto-edit `CLAUDE.md` — it is a curated document the team owns. The benefit of putting it there is that all Claude interactions will respect it, not just spec audits.
