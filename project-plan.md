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

After the interview and design are complete, write the plan to `~/.claude/plans/[project-slug].html` as a self-contained HTML file. Use this exact structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Plan: [Project Name]</title>
<style>
  body { font-family: system-ui, sans-serif; max-width: 960px; margin: 0 auto; padding: 2rem; background: #0f0f0f; color: #e0e0e0; line-height: 1.65; }
  h1 { color: #fff; border-bottom: 2px solid #2a2a2a; padding-bottom: 0.5rem; margin-bottom: 0.25rem; }
  h1 small { font-size: 0.5em; color: #666; font-weight: normal; }
  h2 { color: #a0c4ff; margin-top: 2.5rem; border-left: 3px solid #a0c4ff; padding-left: 0.75rem; }
  h3 { color: #80d8a0; margin-top: 1.5rem; }
  code { background: #1e1e1e; padding: 0.1rem 0.4rem; border-radius: 3px; font-size: 0.88em; color: #f0a070; }
  pre { background: #1a1a1a; border: 1px solid #2a2a2a; padding: 1rem; border-radius: 6px; overflow-x: auto; font-size: 0.88em; }
  pre code { background: none; color: #ccc; padding: 0; }
  table { width: 100%; border-collapse: collapse; margin: 1rem 0; }
  th { background: #1a1a1a; text-align: left; padding: 0.6rem 0.75rem; color: #aaa; font-size: 0.85em; text-transform: uppercase; letter-spacing: 0.05em; }
  td { padding: 0.6rem 0.75rem; border-bottom: 1px solid #222; }
  .ticket { background: #141414; border: 1px solid #2a2a2a; border-radius: 8px; padding: 1.25rem; margin: 1rem 0; }
  .ticket-title { margin: 0 0 0.75rem; color: #ffd080; font-size: 1.05em; display: flex; align-items: center; gap: 0.75rem; }
  .badge { display: inline-block; padding: 0.15rem 0.55rem; border-radius: 999px; font-size: 0.75em; font-weight: 700; letter-spacing: 0.03em; }
  .badge-open { background: #1a2a3a; color: #6ab0f5; }
  .badge-haiku { background: #1a2a1a; color: #80d8a0; }
  .badge-sonnet { background: #2a2a1a; color: #ffd080; }
  .badge-opus { background: #2a1a2a; color: #d080ff; }
  .meta { font-size: 0.82em; color: #666; margin-bottom: 0.75rem; }
  .criteria { margin: 0; padding-left: 1.25rem; }
  .criteria li { margin: 0.3rem 0; color: #ccc; font-size: 0.93em; }
  .dep-graph { background: #141414; border: 1px solid #2a2a2a; border-radius: 6px; padding: 1rem; font-family: monospace; font-size: 0.88em; color: #aaa; white-space: pre; }
  ul, ol { padding-left: 1.5rem; }
  li { margin: 0.3rem 0; }
  p { margin: 0.6rem 0; }
  .section-meta { color: #555; font-size: 0.85em; margin-top: -0.5rem; margin-bottom: 1rem; }
</style>
</head>
<body>

<h1>Plan: [Project Name] <small>Generated [date]</small></h1>

<h2>Context</h2>
<p>[Why this is being built, what problem it solves, who uses it]</p>

<h2>Deliverable</h2>
<p>[Exactly what will exist when done — file path, URL, command, etc.]</p>

<h2>Project Structure</h2>
<pre><code>[Directory tree of files to be created]</code></pre>

<h2>Agent Teams</h2>
<table>
  <thead><tr><th>Team</th><th>Role</th><th>LLM</th><th>Rationale</th></tr></thead>
  <tbody>
    <tr><td>[Team]</td><td>[Role]</td><td><span class="badge badge-[level]">[level]</span></td><td>[Rationale]</td></tr>
  </tbody>
</table>

<h2>Ticket Breakdown</h2>

<!-- Repeat this block for each ticket -->
<div class="ticket">
  <div class="ticket-title">
    T-[ID] — [Short Title]
    <span class="badge badge-open">OPEN</span>
    <span class="badge badge-[llm]">[llm]</span>
  </div>
  <div class="meta">Team: [Team Name]</div>
  <p>[1-2 sentence description]</p>
  <h3 style="margin-top:0.5rem;font-size:0.9em;color:#888;">Acceptance Criteria</h3>
  <ul class="criteria">
    <li>[Criterion 1]</li>
    <li>[Criterion 2]</li>
  </ul>
</div>

<h2>QA Monitor Protocol</h2>
<ol>
  <li>Reads the ticket's acceptance criteria from TICKETS.md</li>
  <li>Reads the relevant code/output</li>
  <li>Traces logic or inspects output against each criterion</li>
  <li>Updates TICKETS.md: <code>CLOSED</code> if ALL criteria pass; <code>NEEDS_REWORK</code> + failure note if ANY fail</li>
  <li>Does NOT fix tickets — only evaluates and routes</li>
  <li>After 2 NEEDS_REWORK rounds → escalates to Architect (opus)</li>
</ol>

<h2>Execution Order</h2>
<div class="dep-graph">[dependency graph with comments]</div>

<h2>Verification</h2>
<p>[How the user tests the final result — specific actions + expected outcomes]</p>

</body>
</html>
```

Populate every `[placeholder]` with real content from the plan. Each ticket gets its own `<div class="ticket">` block. Apply the correct badge class: `badge-haiku`, `badge-sonnet`, or `badge-opus`.

After writing the file, tell the user:
> Plan written to `~/.claude/plans/[project-slug].html` — open it in your browser to review.

Then ask for approval before any execution begins. Do **not** print the full plan content in the terminal.

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
