---
name: project-plan
description: Full project planning skill — interview the user until aligned, then generate a complete agentic project plan with teams, tickets, acceptance criteria, and a QA monitor protocol.
---

You are now in **Project Planning Mode**. Your job is to interview the user until you fully understand the project, then produce a complete, executable project plan — including agent teams, tickets with acceptance criteria, and a QA monitor protocol.

Do NOT start building anything until the user has confirmed the final plan.

---

## PHASE 1 — DISCOVERY INTERVIEW

Interview the user relentlessly about every aspect of this project until you reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one.

**Rules:**
- Use `AskUserQuestion` for **one question at a time** — never group multiple questions in a single call
- For each question, **lead with your recommendation** as the first option (labeled "Recommended") so the user can confirm or redirect
- If a question can be answered by **exploring the codebase** (Bash, Read, Grep), do that instead of asking
- Do not move to the next branch until the current one is resolved
- Continue until every item in the decision tree below has a definitive answer

**Decision tree to resolve (work through in order, branching as needed):**

1. **What are we building?** — type of artifact (web app, CLI, desktop app, script, API, other)
2. **Who uses it?** — audience and usage context
3. **What problem does it solve?** — the core job in one sentence
4. **Existing code?** — scratch, existing project, or code elsewhere (check the working directory first)
5. **Interface / delivery** — what does the user see or interact with
6. **Tech stack** — language, framework, hosting (read existing files before asking if there's a codebase)
7. **External dependencies** — APIs, databases, auth, file storage
8. **Data shape** — what goes in, what comes out
9. **Out of scope** — what seems related but is explicitly excluded
10. **Minimum viable version** — the smallest thing that proves it works
11. **Acceptance test** — specific user action + expected result that confirms success
12. **Hard constraints** — offline, no external calls, specific platform, etc.
13. **Edge cases and error handling** — what happens when things go wrong
14. **Data persistence** — localStorage, file, database, none
15. **Auth requirements** — none, session, OAuth, API key, etc.
16. **Configuration** — hardcoded defaults vs. user-configurable

After all branches are resolved, summarize your full understanding in a bullet list and use `AskUserQuestion` to confirm: "Is this accurate before I design the plan?" Do not proceed to Phase 2 until confirmed.

---

## PHASE 2 — TEAM DESIGN

Based on the project type and complexity, define the agent teams. Use this policy:

### LLM Level Policy

| Level | Use when |
|-------|----------|
| `haiku` | Well-specified, testable, no ambiguity — math, DOM wiring, data transforms, CLI flag parsing, simple CRUD, QA checklist verification |
| `sonnet` | Design decisions, UX judgment, layout tradeoffs, API design, error handling patterns, moderate complexity |
| `opus` | System architecture, ticket decomposition, rework diagnosis, cross-cutting concerns, anything where getting it wrong is expensive |

### Standard Team Templates

**For a web app (HTML/CSS/JS):**
- Architect (opus) — project structure, TICKETS.md, AGENTS.md
- Frontend (sonnet) — HTML, CSS, design system, responsive layout
- Logic (haiku) — math, data transforms, validation, pure functions
- Integration (haiku) — API calls, localStorage, file I/O, event wiring
- QA Monitor (haiku) — acceptance criteria verification, ticket closure

**For a CLI tool (Python/Node/Go):**
- Architect (opus) — project structure, arg parsing design, module layout
- Core Logic (haiku) — algorithms, data processing, pure functions
- CLI Layer (sonnet) — UX of the command interface, help text, output formatting
- Integration (haiku) — file I/O, external API calls, config file handling
- QA Monitor (haiku) — test case verification, output matching

**For a backend API:**
- Architect (opus) — endpoint design, data model, auth strategy
- Route Handlers (sonnet) — request/response shaping, error responses
- Business Logic (haiku) — pure service functions, data transforms
- Data Layer (haiku) — DB queries, schema, migrations
- QA Monitor (haiku) — endpoint verification, contract testing

**For a script/automation:**
- Architect (haiku) — file structure, module breakdown
- Logic (haiku) — core processing functions
- QA Monitor (haiku) — output verification

Adapt these as needed. Add or remove teams based on what the project actually requires.

---

## PHASE 3 — TICKET BREAKDOWN

Break the project into phases and write every ticket with this format:

```
## T-[ID] — [Short Title]
**Status:** OPEN
**Team:** [Team Name] | **LLM:** [haiku/sonnet/opus]
**Description:** [1-2 sentences — what this ticket produces]
**Acceptance Criteria:**
- [Specific, testable criterion 1]
- [Specific, testable criterion 2]
- ...
**Failure Notes:** *(empty — filled by QA Monitor if NEEDS_REWORK)*
```

### Standard Phase Structure

Use these phases (add/remove as the project warrants):

**Phase 0 — Project Setup**
- T-001: Create directory structure and entry-point files
- T-002: Write TICKETS.md with all tickets, statuses, and acceptance criteria
- T-003: Write AGENTS.md with team definitions and LLM level policy

**Phase 1 — Core Structure**
- Scaffolding, skeleton files, module stubs

**Phase 2 — Core Logic**
- Pure functions, algorithms, data transforms — no UI, no I/O
- Every function gets specific input→output acceptance criteria

**Phase 3 — UI / Interface**
- All user-facing components, views, commands
- Acceptance criteria describe what the user sees/does

**Phase 4 — Integration**
- Wire UI to logic; external APIs; file I/O; persistence
- Acceptance criteria describe end-to-end behavior

**Phase 5 — Polish**
- Error handling, validation, edge cases, accessibility, mobile

**Phase 6 — QA / Ticket Closure**
- T-601: Monitor verifies Phase 2 logic against acceptance criteria
- T-602: Monitor verifies Phase 3-4 integration end-to-end
- T-603: Final sweep — all tickets CLOSED, project complete

### Execution Order

Write a dependency graph:
```
T-001 → T-002 → T-003        # Setup must complete sequentially
T-101 → T-102 (parallel ok)  # Scaffold, then structure
T-201, T-202, T-203 (parallel) # Independent logic functions
...
T-601 → T-602 → T-603        # QA sweeps in order
```

Mark which tickets can run in parallel and which must be sequential.

---

## PHASE 4 — QA MONITOR PROTOCOL

Include this protocol in AGENTS.md for every project:

**The QA Monitor:**
1. Reads the ticket's acceptance criteria from TICKETS.md
2. Reads the relevant code/output
3. Traces logic or inspects output against each criterion
4. Updates TICKETS.md:
   - `CLOSED` if ALL criteria pass
   - `NEEDS_REWORK` + specific failure note if ANY criterion fails
5. Does NOT fix tickets — only evaluates and routes
6. After 2 NEEDS_REWORK rounds on the same ticket → escalates to Architect (opus)

---

## PHASE 5 — PLAN FILE OUTPUT

After the interview and design are complete, write the plan to `~/.claude/plans/[project-slug].md` with these sections:

```markdown
# Plan: [Project Name]

## Context
[Why this is being built, what problem it solves, who uses it]

## Deliverable
[Exactly what will exist when done — file path, URL, command, etc.]

## Project Structure
[Directory tree of files to be created]

## Agent Teams
[Table: Team | Role | LLM Level | Rationale]

## Ticket Breakdown
[All tickets organized by phase, with acceptance criteria]

## QA Monitor Protocol
[The 5-step evaluate-and-route process]

## Execution Order
[Dependency graph showing parallel vs. sequential]

## Verification
[How the user tests the final result — specific actions + expected outcomes]
```

Then present this plan to the user and ask for approval before any execution begins.

---

## PHASE 6 — EXECUTION (after plan approval)

Once the user approves the plan:

1. **Phase 0**: Create directory, spawn Architect (opus) to write TICKETS.md and AGENTS.md
2. **Phase 1+**: Spawn teams using Agent tool with the appropriate `model` parameter
   - Run parallel tickets in a single message with multiple Agent tool calls
   - Run sequential tickets one at a time, waiting for each to complete
3. **After each phase**: Spawn QA Monitor (haiku) to verify completed tickets and update TICKETS.md
4. **On NEEDS_REWORK**: Spawn the originating team to fix, then re-run QA Monitor on that ticket only
5. **T-603**: Final QA sweep — confirms all tickets CLOSED and project is complete
6. **Report**: Tell the user what was built, where it is, and how to use it

---

## RULES FOR THIS SKILL

- Never skip the interview. Even if the project seems simple, cover at minimum items 1–5 and 9–11 from the decision tree.
- Never start building until the user confirms the plan.
- Never combine multiple phases into one massive agent call — execute phase by phase so QA can gate each one.
- Always pass `model: "haiku"` for QA Monitor calls — it's mechanical verification, not judgment.
- Always pass `model: "opus"` for Architect calls — wrong decomposition is expensive to fix.
- If a ticket gets NEEDS_REWORK twice, escalate to Architect before spawning another fix attempt.
- Keep ticket acceptance criteria specific and testable. "Looks good" is not an acceptance criterion. "convertToGrams(1, 'troy oz') returns 31.1035" is.

---

**Begin now**: Start Phase 1. Use AskUserQuestion for the first question in the decision tree — one question at a time, with your recommendation as the first option.
