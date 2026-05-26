---
name: project-plan
description: Full project planning skill — interview the user until aligned, then generate a complete agentic project plan with teams, tickets, acceptance criteria, and a QA monitor protocol.
---

You are running the `/project-plan` command. Execute all seven phases below in order. Never skip phases, never build until the plan is approved.

---

## Phase 1 — Discovery Interview

Interview the user through up to 16 decision-tree questions. Ask **one question at a time**. Lead each question with a concrete recommendation when a sensible default exists. Wait for the user's answer before asking the next question.

**Interview depth scales with project type.** After question 1, assess complexity and adjust:
- **Simple project** (script, static page, estimated ≤5 tickets): cover questions 1–8, 14–15; skip 9–13 unless user raises them
- **Static web app (no backend)**: skip questions 9 (auth), 10 (data/persistence), 11 (performance) unless user mentions them
- **Complex project** (full-stack, multi-service, estimated >10 tickets): cover all 16 questions

For **simple projects** (confirmed ≤5 tickets after Q1), you may batch 2-3 closely related questions into one `AskUserQuestion` call to reduce round trips.

Decision tree (adapt wording naturally; keep the coverage):

1. **Project summary** — What are we building? (one sentence)
2. **Problem statement** — What pain or need does this solve, and for whom?
3. **Target users** — Who will use this? (developers, end-users, internal team, etc.)
4. **Core MVP features** — What are the 3–5 things it absolutely must do to be useful?
5. **Out of scope** — What are we explicitly NOT doing in this version?
6. **Tech stack** — Any language, framework, or runtime preferences? (lead with a recommendation based on the project type)
7. **Existing codebase** — Are we building from scratch or extending an existing repo? If extending, what's relevant context?
8. **External integrations** — APIs, databases, third-party services, message queues?
9. **Auth & security** — Do we need authentication, authorization, or any compliance constraints?
10. **Data & persistence** — What data needs to be stored, how, and for how long?
11. **Performance & scale** — Any latency SLAs, throughput targets, or scale expectations?
12. **Testing requirements** — Unit, integration, e2e? Coverage targets? Manual QA gates?
13. **Deployment target** — Where does this run? (local, Docker, cloud, serverless, embedded)
14. **Timeline & milestones** — Any hard deadlines or desired delivery phases?
15. **Definition of done** — How will we know when the project is complete and successful?
16. **Open risks or unknowns** — What is the user most uncertain or worried about?

After all required answers are collected, summarize what you heard back to the user in a short bulleted list and ask: "Does this capture everything, or anything to correct before I build the plan?"

---

## Phase 2 — Team Design

Based on the discovery answers, assign agent roles using this LLM-level policy:

| Role | Model | When to use |
|------|-------|-------------|
| Architect | opus | System-wide design decisions, cross-cutting concerns, ambiguous architecture choices |
| Engineer | sonnet | Feature implementation requiring design judgment |
| Mechanic | haiku | Mechanical, templated, or boilerplate work (scaffolding, renaming, formatting) |
| QA Monitor | sonnet | Reads acceptance criteria, inspects code/output, gates each phase |

List the team you are assembling and which tickets each role owns. If the project is small enough that one Engineer covers everything, say so.

---

## Phase 3 — Ticket Decomposition

Break the project into **phases**. Each phase must complete before the next begins (unless tasks within a phase are safely parallelizable).

**Parallel rule (apply mechanically):** Tickets run in parallel by default. Make sequential only when:
1. They write to the same file, OR
2. One ticket's output is another's required input.

Everything else is parallel — mark it explicitly with `Parallel-safe: yes`.

For each ticket write:

```
### [PHASE-N.M] Ticket Title

**Owner:** [role + model]
**Parallel-safe:** yes | no (reason if no)

**Description:**
One paragraph of what this ticket accomplishes and why.

**Acceptance Criteria:**
- [ ] Specific, observable, testable criterion
- [ ] Another criterion (no "works correctly" or "is implemented" — must be checkable)
- [ ] ...

**Out of scope for this ticket:**
- ...
```

Rules:
- Never write vague criteria like "feature is implemented" or "code is clean."
- Every criterion must be falsifiable: a QA agent can inspect output or code and give a binary PASS/FAIL.
- Keep tickets small enough that one agent can complete them in a single session.

**Size gate:** If the project will have ≤5 tickets total, skip a dedicated Phase 0 setup ticket. Do any directory creation and tracking file setup inline during Phase 6 execution — the agent-spawn overhead isn't worth it for a small project.

---

## Phase 4 — QA Protocol

Define the QA Monitor's job for this project:

- What artifacts does QA inspect per ticket? (test output, log lines, file diffs, API responses, UI screenshots, etc.)
- What constitutes CLOSED vs NEEDS_REWORK for each phase?
- Escalation rule: after **2 NEEDS_REWORK** rounds on the same ticket, escalate to Architect (opus) for a root-cause review before retrying.

Write a short QA runcard specific to this project's stack and acceptance criteria.

---

## Phase 5 — Plan Document & Approval

Write the complete plan as a single structured document:

```
# Project Plan: [Name]

## Summary
[2–3 sentences from discovery]

## Team
[From Phase 2]

## Phases & Tickets
[From Phase 3, all phases and tickets]

## QA Protocol
[From Phase 4]

## Risks & Mitigations
[From discovery Q16 + any identified during planning]
```

Then ask the user:

> "Here is the full plan. Please review it. Reply **APPROVED** to begin execution, or tell me what to change."

Do not proceed to Phase 6 until the user replies APPROVED (or equivalent confirmation).

---

## Phase 6 — Execution

Execute phase-by-phase:

1. Announce which phase you are starting.
2. **Size gate for setup:** If total ticket count ≤5, do any directory creation and tracking file setup inline — don't spawn a separate Architect agent for it.
3. For each ticket in the phase: implement it, then hand off to the QA Monitor role for gate review against the acceptance criteria.
4. If QA returns NEEDS_REWORK, fix and re-gate. The fix agent prompt must include:
   - The failing acceptance criterion verbatim from the ticket
   - The QA Monitor's specific failure note
   - The file path(s) to change
   A cold "fix ticket N.M" prompt is not enough — the agent needs the precise failure context to avoid re-discovering the same problem.
5. After 2 NEEDS_REWORK failures on the same ticket, pause and escalate to Architect (opus) analysis before retrying.
6. Once all tickets in a phase pass QA, announce the phase as CLOSED and move to the next.
7. After the final phase, deliver a completion summary: what was built, any deviations from the plan, and suggested follow-on work. Then immediately execute **Phase 7 — Post-Project Review**.

Always run tests and verify observable behavior — do not self-certify acceptance criteria without evidence.

---

## Phase 7 — Post-Project Review

Execute this phase automatically after Phase 6 completes. Do not ask the user for permission — just run it.

### Step 1 — Conversation Audit

Review the full project conversation and extract every instance of:

- **Rework cycles:** any ticket that received NEEDS_REWORK (note what was wrong and why)
- **Wrong assumptions:** anything assumed during planning or execution that turned out to be incorrect
- **Plan deviations:** anything built differently from the approved plan, and why
- **Fixes applied:** specific corrections made mid-project and what triggered them
- **Skipped steps:** any planned acceptance criteria or QA steps that were skipped or self-certified without evidence

### Step 2 — Lessons Distillation

For each finding from Step 1, decide if it is a lesson worth persisting:

- **Worth saving:** non-obvious, repeatable, caused real rework or wasted effort
- **Skip:** one-off situational details, things already obvious from context, things already captured in existing memory

### Step 3 — Duplicate Check

Before writing any memory entry, read `~/.claude/projects/-home-drake/memory/MEMORY.md`. If a lesson is already captured there, skip it. If an existing entry is outdated or incomplete, update it instead of creating a duplicate.

### Step 4 — Write Memory Entries

Write memory entries to `~/.claude/projects/-home-drake/memory/` using this exact frontmatter format:

```markdown
---
name: {{short-kebab-case-slug}}
description: {{one-line summary — specific enough to judge relevance at a glance}}
metadata:
  type: {{feedback | project}}
---

{{Lead with the rule or fact. Then:}}
**Why:** {{what went wrong and what it cost — be specific}}
**How to apply:** {{when this rule kicks in and what to do differently}}
```

Use type `feedback` for repeatable behavioral rules (how to approach a class of problem). Use type `project` for project-specific facts that decay over time.

Write **one file per lesson**. Name files descriptively: `feedback_<topic>.md` or `project_<topic>.md`.

### Step 5 — Update MEMORY.md Index

For each new or updated memory file, add or update a one-line entry in `~/.claude/projects/-home-drake/memory/MEMORY.md`:

```
- [Title](filename.md) — one-line hook (under 150 chars total)
```

### Step 6 — Deliver Review Report

Output a structured report to the user:

```
## Post-Project Review: [Project Name]

### Findings
| # | Type | Finding | Lesson saved? |
|---|------|---------|---------------|
| 1 | Rework / Wrong assumption / Deviation / Fix / Skipped step | What happened | Yes — feedback_xxx.md / No — already in memory / No — one-off |

### Memory Written
- [list of files written or updated, or "none"]

### Patterns to Watch
[1–3 sentences on any recurring themes across findings]
```
