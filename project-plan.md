---
name: project-plan
description: Full project planning skill — interview the user until aligned, then generate a complete agentic project plan with teams, tickets, acceptance criteria, and a QA monitor protocol.
---

You are running the `/project-plan` command. Execute all seven phases below in order. Never skip phases, never build until the plan is approved.

---

## Phase 1 — Discovery Interview

**Recall first (close the learning loop).** Before asking anything, pull prior lessons so you don't repeat past mistakes or re-ask settled questions:
- `mempalace_search` keyed on the project type + domain (e.g. "CLI tool python test detection", "web app fetch fallback CORS"), and again on `wing: guidance` for behavioral lessons.
- Surface what you recalled in one line ("Past runs flagged X — I'll account for that") so the user sees it informed the plan. If MemPalace is unavailable, note it and proceed — recall is best-effort, never a blocker.

Phase 1 then has two steps: **understand the project first, then ask only what you couldn't figure out.** Never open with a generic questionnaire.

### Step 1 — Comprehension pass (before asking anything)

Build a working understanding of the project from what's already in front of you, scaled to the project:

- **Read the request.** Parse the user's opening description and command args for the goal, named technologies, constraints, and scope signals.
- **If extending an existing codebase:** inspect it before asking anything. Identify the stack, build/test tooling, directory layout, and the conventions already in play (glob/grep/read the relevant areas; for a large or unfamiliar repo, spawn an `Explore` agent to map it). Tech stack, existing-codebase context, testing setup, and deployment target are usually *answerable from the repo itself* — answer them rather than asking.
- **If building from scratch:** restate the goal in your own words and note the conventional defaults for this kind of project (stack, layout, deploy target).

Produce a short internal model: what the project is, the likely stack, what you can already infer, and the genuine unknowns. **Size the project here** (simple ≤5 tickets vs. complex) — this drives interview depth.

### Step 2 — Targeted interview

Ask **only the questions comprehension left genuinely open**, phrased with the specifics you found — never generic boilerplate. For anything you inferred in Step 1, *state your assumption and ask the user to confirm or correct it* rather than asking from zero. Lead every question with a concrete, project-grounded recommendation.

> Generic (bad): "What tech stack do you want?"
> Grounded (good): "This repo is a FastAPI service on Python 3.12 with pytest — I'll keep that stack and put the new endpoints under `app/routers/`. Sound right?"

Use the decision tree below as a **coverage checklist, not a script.** For each item decide: *known* (inferred — confirm, don't ask) / *unknown and relevant* (ask) / *not applicable* (skip). Drop whole topics that don't apply.

**Batch what remains — no serial one-at-a-time round-trips.** `AskUserQuestion` takes up to 4 questions per call; group the still-open questions into a few themed calls (Framing / Scope / Build context / Infra / Delivery / Closeout). Open-ended items that don't fit fixed options may be asked as plain prose.

**Interview depth scales with project type:**
- **Simple project** (script, static page, ≤5 tickets): confirm goal and scope; skip infra (Q9–11) unless raised. Often 1–2 batched calls — or zero, if comprehension already answered everything and you only need a scope confirmation.
- **Static web app (no backend)**: skip Q9 (auth), Q10 (data/persistence), Q11 (performance) unless mentioned.
- **Complex project** (full-stack, multi-service, >10 tickets): cover all 16 topics — but still skip any the comprehension pass already nailed down.

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

Once the open questions are answered, summarize back in a short bulleted list **the inferences you made in Step 1 plus the user's answers** — clearly marking which items you inferred vs. were told — and ask: "Does this capture everything, or anything to correct before I build the plan?" This gives the user a chance to catch a wrong assumption before it hardens into the plan.

---

## Phase 2 — Team Design

Based on the discovery answers, assign agent roles using this LLM-level policy:

| Role | Model | When to use |
|------|-------|-------------|
| Architect | opus | System-wide design decisions, cross-cutting concerns, ambiguous architecture choices |
| Engineer | sonnet | Feature implementation requiring design judgment |
| Mechanic | haiku | Mechanical, templated, or boilerplate work (scaffolding, renaming, formatting) |
| QA Monitor | sonnet | Reads acceptance criteria, inspects code/output, gates each phase |

**These roles are spawned, not role-played.** Each role maps to a real `Agent` call:
- The role's **Model** column is the `Agent` `model` parameter — it accepts exactly `opus | sonnet | haiku`, so the table maps 1:1.
- Use `subagent_type: general-purpose` for Engineer/Mechanic/QA work unless a more specific agent type fits; use `feature-dev:code-reviewer` for the QA Monitor when reviewing code.
- **QA independence is mandatory:** the QA Monitor runs as a *separate* `Agent` call with fresh context and must not be the same agent that wrote the code it reviews. The main (orchestrator) thread never gates its own work — it spawns a QA agent to do it. This is the whole point of the gate; a thread reviewing its own output rubber-stamps.

List the team you are assembling and which tickets each role owns. If the project is small enough that one Engineer covers everything, say so — but QA is still spawned separately.

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

**Coverage check (do before moving on):** Every MVP feature from Q4 must map to at least one ticket, and every clause of the Definition of Done from Q15 must map to at least one acceptance criterion or QA gate. State the mapping briefly; if anything in Q4/Q15 has no ticket, you have a gap — add a ticket or flag it to the user.

**Size gate:** If the project will have ≤5 tickets total, skip a dedicated Phase 0 setup ticket. Do any directory creation and tracking file setup inline during Phase 6 execution — the agent-spawn overhead isn't worth it for a small project.

---

## Phase 4 — QA Protocol

Define the QA Monitor's job for this project:

- What artifacts does QA inspect per ticket? (test output, log lines, file diffs, API responses, UI screenshots, etc.)
- What constitutes CLOSED vs NEEDS_REWORK for each phase?
- Escalation rule: after **2 NEEDS_REWORK** rounds on the same ticket, escalate to Architect (opus) for a root-cause review before retrying.

**Evidence-first gating (do not trust narration).** A ticket is CLOSED only on observed evidence, never on the implementer's claim that it passed. For each acceptance criterion the QA Monitor must:
- Inspect the artifact **directly** — read the test-output file, the actual file/diff, the exit code, the API response — not the executing agent's summary of it.
- Run the relevant check itself where it can (the detected test command, a grep, a file existence test) and record the **exact command, its exit code, and a short output excerpt** in the gate note.
- Paste that observed evidence beside each criterion in the CLOSED/NEEDS_REWORK note. A gate note with no quoted evidence is not a pass — treat a criterion you cannot independently verify as NEEDS_REWORK.

Record the gate as one row per criterion:

```
| Criterion (verbatim) | Command / artifact inspected | Observed output (snippet) | PASS / FAIL |
```

A criterion with no command run and no artifact quoted is an automatic FAIL. The ticket is CLOSED only when every row is PASS with evidence attached.

Write a short QA runcard specific to this project's stack and acceptance criteria.

---

## Phase 5 — Plan Document & Approval

Write the complete plan as a single structured document **and save it to disk** as `PLAN.md` in the project directory (create the directory if building from scratch). The plan is a shared artifact: execution and QA agents read `PLAN.md` rather than re-deriving the plan from chat, and it survives context compaction across a long session.

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

Alongside `PLAN.md`, create a **status ledger** `PLAN-STATUS.md` — the durable source of truth for execution progress, one row per ticket:

```
# Status — [Name]

| Ticket | Owner | Status | Rework count | Evidence / notes |
|--------|-------|--------|--------------|------------------|
| 1.1 | Engineer/sonnet | OPEN | 0 | |
```

Status values: `OPEN → IN_PROGRESS → QA → CLOSED` (or `BLOCKED`). Update this file as tickets move; never track ticket state only in chat. (For a ≤5-ticket project the ledger may be a short section appended to `PLAN.md` instead of a separate file — your call.)

Then ask the user:

> "Here is the full plan (saved to `PLAN.md`). Please review it. Reply **APPROVED** to begin execution, or tell me what to change."

Do not proceed to Phase 6 until the user replies APPROVED (or equivalent confirmation).

---

## Phase 6 — Execution

Execute phase-by-phase:

1. Announce which phase you are starting.
2. **Size gate for setup:** If total ticket count ≤5, do any directory creation and tracking file setup inline — don't spawn a separate Architect agent for it.
3. **Spawn the phase's tickets.** Launch every `Parallel-safe: yes` ticket in the phase as concurrent `Agent` calls in a single turn (use `run_in_background: true` when there are more than a couple), each at the model its Owner specifies. Run sequential tickets in dependency order. Each agent prompt must point at `PLAN.md`, name the ticket it owns, and start with a quick recon (read the files the ticket will touch and any recalled lessons relevant to it) so it edits with context instead of cold. Update each ticket to `IN_PROGRESS` in `PLAN-STATUS.md` as it starts.
4. **Gate each ticket through a separate QA agent** (fresh context, not the implementing agent) against the acceptance criteria, requiring the Phase 4 evidence table. Move the ticket to `QA`, then `CLOSED` on PASS.
5. If QA returns NEEDS_REWORK, fix and re-gate. The fix agent prompt must include:
   - The failing acceptance criterion verbatim from the ticket
   - The QA Monitor's specific failure note, including the observed evidence (command, exit code, output excerpt) it captured
   - The file path(s) to change
   A cold "fix ticket N.M" prompt is not enough — the agent needs the precise failure context to avoid re-discovering the same problem. Increment the rework count in `PLAN-STATUS.md` each round.
6. **Bound the rework loop with a failure fingerprint.** Before each re-gate, reduce the QA failure to a short signature (failing criterion + the essential error — e.g. the assertion message or exit code, not incidental noise). If the **same signature repeats** (the fix changed nothing observable), stop immediately — do not keep retrying. After 2 NEEDS_REWORK failures on the same ticket, pause and escalate to Architect (opus) for root-cause analysis before retrying. If the ticket still fails after the Architect-guided retry (or a third attempt produces the same fingerprint), **stop and surface it to the user** — mark it `BLOCKED` in the ledger with the fingerprint and what was tried, and present options: replan the ticket, descope it, or accept-as-is with the gap documented. Do not loop indefinitely.
7. Once all tickets in a phase are `CLOSED`, announce the phase as CLOSED and move to the next.
8. After the final phase, deliver a completion summary: what was built, any deviations from the plan, and suggested follow-on work. Then immediately execute **Phase 7 — Post-Project Review**.

Always run tests and verify observable behavior — do not self-certify acceptance criteria without evidence.

---

## Phase 7 — Post-Project Review

Execute this phase automatically after Phase 6 completes. Do not ask the user for permission — just run it.

**Also run this phase on early termination.** If the project is abandoned, paused indefinitely, or the user calls it off mid-execution, run the audit over whatever happened so far — the rework cycles and wrong assumptions from a project that *didn't* finish are often the most valuable lessons. Note in the report that the project terminated early and at which phase.

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

Lessons are stored in **MemPalace**, not in `.md` files. The legacy
`~/.claude/projects/-home-drake/memory/*.md` store is frozen — do not read or write it.

Before writing any lesson, check the palace for an existing version:
- `mempalace_search` with the lesson's keywords (optionally `wing` = the relevant project
  or `guidance`), and/or `mempalace_check_duplicate`.

If a matching lesson already exists and is still accurate, skip it. If it exists but is now
outdated, `mempalace_update_drawer` the existing drawer (and `mempalace_kg_invalidate` +
`mempalace_kg_add` for any fact that changed) instead of creating a duplicate.

### Step 4 — Write Lessons to MemPalace

For each lesson worth persisting, write to the palace via MCP tools:

1. **Verbatim lesson → `mempalace_add_drawer`.** Lead with the rule/fact, then a `Why:`
   line (what went wrong and what it cost) and a `How to apply:` line (when it kicks in,
   what to do differently). Choose the wing/room:
   - Repeatable **behavioral** lesson → wing `guidance`, room `general`.
   - **Project-specific** fact → that project's wing (e.g. `trip-mapper`, `gold-calculator`),
     room `architecture` for design decisions or `general` otherwise.
   - **User** fact → wing `user`, room `general`.
2. **Discrete facts → `mempalace_kg_add`** as subject→predicate→object triples (add
   `valid_from` for dated facts). KG values allow only letters, digits, space, `.`, `'`,
   `-` — keep file paths, URLs, and punctuation in the drawer, not the triple.
3. **Session summary → `mempalace_diary_write`** (agent_name `Claude`): one AAAK-compressed
   entry covering what was built, key decisions, and the lessons saved.

### Step 5 — Verify Persistence

Confirm each lesson is retrievable before reporting done:
- `mempalace_search` returns the new drawer, and `mempalace_kg_query` returns any new facts.

There is no `.md` index to update — MemPalace's own wake-up/`mempalace_status` is the index.

### Step 6 — Deliver Review Report

Output a structured report to the user:

```
## Post-Project Review: [Project Name]

### Findings
| # | Type | Finding | Lesson saved? |
|---|------|---------|---------------|
| 1 | Rework / Wrong assumption / Deviation / Fix / Skipped step | What happened | Yes — drawer in `guidance` / Yes — KG triple / No — already in palace / No — one-off |

### Saved to MemPalace
- [list of drawers added/updated (wing/room), KG triples added, and diary entry — or "none"]

### Patterns to Watch
[1–3 sentences on any recurring themes across findings]
```
