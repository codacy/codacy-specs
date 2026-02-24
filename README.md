# codacy-specs

A Claude Code plugin by [Codacy](https://codacy.com) that audits any specification for AI-agent readiness and rewrites it to be clear, testable, and unambiguous.

Vague specs are the leading cause of AI coding failures. If a ticket clearly states what to build, defines inputs/outputs, specifies error handling, and provides verifiable acceptance criteria, AI agents produce better code with fewer wrong assumptions.

---

## Features

- **Spec audit**: Score any spec on a 0–100 rubric across problem statement, acceptance criteria, error handling, edge cases, testability, and type-specific dimensions
- **Spec type detection**: Automatically identifies API endpoint, UI component, backend feature, bug fix, data migration, or generic specs
- **Guided clarification**: Asks one focused question at a time to fill critical gaps — never fabricates context
- **Spec rewrite**: Produces a fully structured, AI-agent-ready spec using proven templates
- **Multi-source**: Works with Jira issues, Linear issues, local files, or inline pasted content
- **Write-back**: Updates the original Jira or Linear issue directly after user confirmation
- **Project-aware**: Reads your `CLAUDE.md` to understand your team's conventions and terminology before auditing

---

## Installation

### 1. Install the plugin

Add this plugin to your Claude Code project:

```bash
# From your project root, or globally
claude plugin install https://github.com/codacy/codacy-specs
```

Or clone and install locally:

```bash
git clone https://github.com/codacy/codacy-specs
cd codacy-specs
claude plugin install .
```

### 2. (Optional) Connect Jira and/or Linear

This plugin does not bundle its own MCP configuration. If you want to audit and update Jira or Linear issues directly, install the dedicated plugins for those tools — they handle authentication and MCP setup for you:

- **Jira**: Install the [Atlassian Claude Code plugin](https://www.atlassian.com/claude-code)
- **Linear**: Install the [Linear Claude Code plugin](https://linear.app/claude-code)

Once either plugin is active, `spec-improve` will automatically use it to fetch and update issues.

> **No Jira/Linear?** The skill works without them. Paste any spec directly into the chat, or point it at a local file — it will audit and rewrite without needing any external connection.

### 3. (Recommended) Add project context to CLAUDE.md

The skill reads your project's `CLAUDE.md` before every audit. This is the single most impactful thing you can do to improve audit quality: the skill will not flag terms or patterns that are already defined there.

Two things worth adding to your `CLAUDE.md`:

**Terminology** — internal acronyms, service names, and domain concepts your team uses:

```markdown
## Terminology

- **SRM** — Security Risk Management service. Core internal microservice for vulnerability scanning.
- **org** — An organization account in our multi-tenant model. Maps to the `organizations` table.
- **PDQ** — Priority-Driven Queue. Internal async job scheduling system.
```

**API and engineering conventions** — defaults and standards your team follows that don't need to be re-stated in every ticket:

```markdown
## API Conventions

- All endpoints return `application/json` unless stated otherwise.
- Pagination uses `?page=` and `?pageSize=` query params. Default page size is 20, max is 100.
- Authentication is via Bearer token (JWT). All endpoints require auth unless explicitly marked public.
- Error responses follow the shape `{ "error": "ERROR_CODE", "message": "..." }`.
```

The skill uses this context during auditing: if a convention in `CLAUDE.md` covers a gap in the spec, the gap is scored as covered rather than missing.

> **Why `CLAUDE.md` and not a separate file?** Because knowledge in `CLAUDE.md` benefits all Claude interactions in your project, not just spec audits. One source of truth.

---

## Usage

The `spec-improve` skill activates automatically when you use natural language like:

```
audit this spec
improve this Jira ticket PROJ-123
review this Linear issue ENG-42
rewrite this story to have better acceptance criteria
check if this spec is ready for an AI agent
make this ticket clearer
improve the spec in docs/spec.md
```

It also activates as a **pre-work guardrail** when you (or an agent) are about to start implementing a ticket:

```
start working on PROJ-123
implement this ticket
pick up the next item
```

In this mode it runs a quick readiness check and offers to either review and improve the spec, or continue straight to implementation.

### Example: Audit a Jira issue

```
You: improve this Jira ticket PROJ-123
```

The skill will:
1. Fetch PROJ-123 from Jira
2. Read your `CLAUDE.md` to load project context
3. Classify the spec type (e.g. API endpoint)
4. Score it against the rubric and output a quality report
5. Ask if you want to rewrite it
6. Ask one question at a time to fill in missing sections
7. Show you a preview of the rewritten spec
8. Update PROJ-123 in Jira after your confirmation

### Example: Audit a local file

```
You: audit the spec in docs/api-spec.md
```

### Example: Audit inline content

```
You: check if this spec is good:

We need a search endpoint that returns users. Should filter by name and email.
```

---

## Skill Output Format

```
───────────────────────────────────────────────────────────────
  Spec Quality Report: PROJ-123
  Type: API endpoint
  Score: 42/100  (Grade: D)
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

  TYPE-SPECIFIC DIMENSIONS (API endpoint)

  ✅  HTTP method + path        10/10
  ❌  Request schema            0/10
      ↳ missing
  ✅  Pagination behavior       9/10
      ↳ covered by project conventions (CLAUDE.md)
  ❌  Error response codes      0/10
      ↳ missing
───────────────────────────────────────────────────────────────
  Agent Readiness: NOT READY
  An AI agent working from this spec will make 3+ wrong
  assumptions and produce unverifiable code.
───────────────────────────────────────────────────────────────
```

**Grade scale:**
| Grade | Score | Meaning |
|-------|-------|---------|
| A | 85–100 | READY — agent can proceed autonomously |
| B | 70–84  | READY — minor assumptions needed |
| C | 50–69  | CONDITIONALLY READY — clarifications recommended |
| D | 30–49  | NOT READY — 3+ wrong assumptions likely |
| F | 0–29   | NOT READY — too vague to use |

---

## Spec Types Supported

| Type | Examples |
|------|---------|
| API endpoint | REST endpoints, GraphQL mutations, webhook handlers |
| UI component | React/Vue components, pages, modals, forms |
| Backend feature | Services, workers, jobs, business logic |
| Bug fix | Crash reports, regression bugs, UI glitches |
| Data migration | ETL pipelines, schema migrations, data imports |
| Generic | Any spec that doesn't fit the above categories |

---

## How It Works Without MCP

If Atlassian or Linear MCP is not configured, the skill:
1. Asks you to paste the spec content directly
2. Runs the full audit on the pasted content
3. Outputs the rewritten spec for you to copy back manually

---

## Plugin Structure

```
codacy-specs/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── README.md
└── skills/
    └── spec-improve/
        ├── SKILL.md          # Skill definition
        └── references/
            ├── scoring-rubrics.md   # Scoring criteria for all spec types
            ├── spec-templates.md    # Rewrite templates by spec type
            └── spec-examples.md    # Before/after transformation examples
```

**Note**: all user-owned data (terminology, conventions) lives in your project's `CLAUDE.md` — not inside this plugin. This means it survives plugin updates, benefits all Claude interactions (not just spec audits), and can be committed to your repo for the whole team.

---

## Compatibility

- Claude Code CLI (any version supporting plugins)
- Atlassian MCP (optional) — for Jira integration
- Linear MCP (optional) — for Linear integration
- Local files — works out of the box, no MCP required

---

## Contributing

Issues and pull requests welcome at [github.com/codacy/codacy-specs](https://github.com/codacy/codacy-specs).

---

## License

MIT License. Free for personal and commercial use.

---

*Built by [Codacy](https://codacy.com) — code quality for every team.*
