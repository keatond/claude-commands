# Session Handoff — 2026-05-25

## Accomplished

### project-plan skill improvements
- Reviewed `~/.claude/commands/project-plan.md` and identified 8 token/accuracy issues
- Merged our improvements onto the remote v2 (which had added Phase 7 Post-Project Review, a new 16-question format, and the plain-text plan format — all pushed from the other machine)
- Final merged skill at `~/claude-commands/project-plan.md` includes:
  - **Tiered interview depth**: after Q1, sets coverage based on project type (simple/static/complex) — skips irrelevant branches like auth and persistence for static web apps
  - **Question batching**: simple projects (≤5 tickets) may batch 2-3 related questions in one call
  - **Phase 0 size gate**: ≤5 tickets → inline setup, no Architect agent spawn
  - **Explicit parallel rule**: tickets sequential only if same-file writes or output-as-input dep
  - **Rework context spec**: fix agent prompt must include verbatim failing criterion + QA failure note + file path
  - **YAML frontmatter restored**: v2 had dropped it, skill triggering requires it
  - **Hardcoded path fixed**: v2 had `/home/drakenas/` in Phase 7 memory paths → changed to `~/.claude/` portable paths

### Copilot variant
- Created `~/claude-commands/project-plan-copilot.md` — drop-in for `.github/copilot-instructions.md` at work
- Same 7-phase structure and question coverage as the Claude version
- No YAML frontmatter, no `AskUserQuestion` tool references, Phase 7 simplified to conversation-based review

### Shared CSS file
- Created `~/.claude/plans/plan.css` (and `~/claude-commands/plan.css`) — shared stylesheet for HTML plan output files
- Future plan HTML files reference it via `<link rel="stylesheet" href="plan.css">` instead of inlining ~80 lines of CSS

### Cross-machine sync architecture (THIS MACHINE IS DONE)
- `~/claude-commands/` is the git repo — single source of truth
- `~/.claude/commands/project-plan.md` is now a **symlink** to `~/claude-commands/project-plan.md`
- `~/.claude/settings.json` has a `SessionStart` hook that runs `git -C ~/claude-commands pull --ff-only --quiet` at the start of every Claude Code session
- All changes pushed to `git@github.com:keatond/claude-commands.git` (commit `9e55a1c`)

---

## Current State

**This machine (PC 1) — fully set up:**
```
~/claude-commands/
├── project-plan.md          ← canonical source (Claude version)
├── project-plan-copilot.md  ← Copilot / .github/copilot-instructions.md version
├── plan.css                 ← shared stylesheet for HTML plan files
└── README.md

~/.claude/commands/project-plan.md  → symlink to ~/claude-commands/project-plan.md
~/.claude/plans/plan.css            ← also exists here for plan HTML output

~/.claude/settings.json SessionStart hook:
  git -C ~/claude-commands pull --ff-only --quiet
```

**Other machine (PC 2) — needs setup** (see Pending below)

---

## Pending

### Priority 1 — Set up PC 2 (the whole point of the handoff)

On the other machine, do these steps in order:

1. Clone the repo (if not already there):
   ```bash
   git clone git@github.com:keatond/claude-commands.git ~/claude-commands
   ```

2. Replace the local command file with a symlink:
   ```bash
   # Back up existing file first if it exists
   mv ~/.claude/commands/project-plan.md ~/.claude/commands/project-plan.md.bak 2>/dev/null || true
   ln -sf ~/claude-commands/project-plan.md ~/.claude/commands/project-plan.md
   ls -la ~/.claude/commands/project-plan.md  # should show symlink
   ```

3. Copy the plan.css to the plans directory:
   ```bash
   cp ~/claude-commands/plan.css ~/.claude/plans/plan.css
   ```

4. Add the SessionStart hook to `~/.claude/settings.json`:
   - Open `~/.claude/settings.json`
   - Add this entry to the `hooks.SessionStart` array (create the array if it doesn't exist):
   ```json
   {
     "matcher": "",
     "hooks": [
       {
         "type": "command",
         "command": "git -C ~/claude-commands pull --ff-only --quiet 2>/dev/null || true",
         "statusMessage": "Syncing claude-commands..."
       }
     ]
   }
   ```

5. Verify the hook is valid:
   ```bash
   jq '.hooks.SessionStart' ~/.claude/settings.json
   ```

### Priority 2 — Copilot skill at work

- Copy the contents of `~/claude-commands/project-plan-copilot.md` into `.github/copilot-instructions.md` in the relevant work repo
- No other changes needed — it's already Copilot-compatible (no frontmatter, no Claude tool references)

### Priority 3 — Future workflow reminders

- **Always edit `~/claude-commands/project-plan.md` directly** — it's the source of truth on both machines
- **Never edit `~/.claude/commands/project-plan.md`** — it's a symlink, but the habit matters
- After editing on either machine: `git commit && git push` from `~/claude-commands/`
- The other machine will auto-pull at next session start

---

## Context for Next Session

### Key files
| File | Purpose |
|------|---------|
| `~/claude-commands/project-plan.md` | Claude Code command — 7-phase planning skill |
| `~/claude-commands/project-plan-copilot.md` | Copilot instructions variant |
| `~/claude-commands/plan.css` | Shared stylesheet for HTML plan output |
| `~/.claude/commands/project-plan.md` | Symlink to the above (do not edit directly) |
| `~/.claude/settings.json` | Has the SessionStart auto-pull hook |

### Gotchas
- The remote repo (`keatond/claude-commands`) had a v2 update pushed from the other machine that hadn't been pulled to PC 1 — caused a merge conflict mid-session. The sync hook prevents this going forward.
- v2 had dropped the YAML frontmatter (`name:` / `description:`) from `project-plan.md`. This was re-added — without it, the skill description doesn't appear in Claude's available-skills list.
- v2 also had hardcoded `/home/drakenas/` paths in Phase 7 (memory writes). These were changed to `~/.claude/` which works on both machines regardless of username.
- `plan.css` lives in two places: `~/claude-commands/plan.css` (in version control) and `~/.claude/plans/plan.css` (where plan HTML files look for it). Keep them in sync if you ever update the styles.
- GitHub uses SSH (`git@github.com:keatond/claude-commands.git`) — SSH key must be set up on PC 2 for push/pull to work.

### How the sync works
- `SessionStart` hook fires once per Claude Code session start
- It runs `git -C ~/claude-commands pull --ff-only --quiet`
- `--ff-only` means it only pulls if it's a clean fast-forward — it will NOT clobber local uncommitted edits
- If you have uncommitted local changes that conflict with remote, the pull silently skips (exits 1, swallowed by `|| true`) — you'll need to manually resolve
- Best practice: always commit before switching machines

---

## Resume Commands

### Verify PC 1 is still set up correctly
```bash
# Check symlink
ls -la ~/.claude/commands/project-plan.md

# Check hook is in settings
jq '.hooks.SessionStart[] | select(.hooks[].command | test("claude-commands"))' ~/.claude/settings.json

# Check repo is up to date
cd ~/claude-commands && git log --oneline -3 && git status
```

### Set up PC 2 (run these in order)
```bash
# 1. Clone repo (skip if already cloned)
git clone git@github.com:keatond/claude-commands.git ~/claude-commands

# 2. Pull latest
cd ~/claude-commands && git pull

# 3. Create symlink
mv ~/.claude/commands/project-plan.md ~/.claude/commands/project-plan.md.bak 2>/dev/null || true
ln -sf ~/claude-commands/project-plan.md ~/.claude/commands/project-plan.md

# 4. Copy CSS
cp ~/claude-commands/plan.css ~/.claude/plans/plan.css

# 5. Verify symlink
ls -la ~/.claude/commands/project-plan.md

# 6. Add SessionStart hook — open settings and add manually, then verify:
jq '.hooks.SessionStart' ~/.claude/settings.json
```

### Test the skill works
```bash
# Should show the skill in available commands
ls ~/.claude/commands/

# Should show current content (from the repo)
head -10 ~/.claude/commands/project-plan.md
```
