# Agent Team Skills — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:team-execute to implement this plan task-by-task.
> **Resume:** Check `docs/plans/2026-03-08-agent-team-skills.state.json` for current progress.

**Goal:** Build a skills.sh-compatible package that extends obra/superpowers with named team agents (Developer, Architect), state tracking for resumability, and orchestrator progress display.

**Architecture:** Four SKILL.md files compose the package — `team-plan` wraps obra writing-plans, `team-execute` orchestrates named Developer and Architect agents per task, `team-developer` and `team-architect` define agent roles. State is persisted in `.state.json` alongside the plan file.

**Tech Stack:** skills.sh (`npx skills`), SKILL.md format, obra/superpowers skills as internal delegates, JSON, git worktrees.

## Progress

- [ ] Task 1: Package scaffold (`package.json` + root `SKILL.md`)
- [ ] Task 2: `team-developer` skill
- [ ] Task 3: `team-architect` skill
- [ ] Task 4: `team-plan` skill
- [ ] Task 5: `team-execute` skill (orchestrator)
- [ ] Task 6: `README.md`
- [ ] Task 7: Push & verify install

---

### Task 1: Package scaffold

**Files:**
- Create: `package.json`
- Create: `SKILL.md`

**Step 1: Create `package.json`**

```json
{
  "name": "agent-team-skills",
  "version": "0.1.0",
  "description": "Team-based implementation skills extending obra/superpowers",
  "keywords": ["skills", "claude", "agents", "team"],
  "skills": [
    "team-plan",
    "team-execute",
    "team-developer",
    "team-architect"
  ]
}
```

**Step 2: Create root `SKILL.md`**

```markdown
---
name: agent-team-skills
description: Team-based implementation skills. Use team-plan to write plans with state tracking, team-execute to run them with named Developer and Architect agents.
---

# Agent Team Skills

This package provides team-based implementation workflow:

1. **team-plan** — wraps obra writing-plans, adds progress tracking and state file
2. **team-execute** — orchestrates Developer + Architect agents per task, tracks state, shows progress checklist
3. **team-developer** — Developer+Tester agent role (TDD, commits, pushes)
4. **team-architect** — Architect agent role (code review, ADR/design compliance)

## Install

```bash
npx skills add ~/Projects/agent-team-skills -g
```

## Usage

Start with `team-plan` → then `team-execute`.
```

**Step 3: Verify structure**

```bash
ls ~/Projects/agent-team-skills/
```
Expected: `LICENSE  SKILL.md  docs/  package.json`

**Step 4: Commit**

```bash
cd ~/Projects/agent-team-skills
git add package.json SKILL.md
git commit -m "feat: add package scaffold"
git push
```

---

### Task 2: `team-developer` skill

**Files:**
- Create: `team-developer/SKILL.md`

**Step 1: Create `team-developer/SKILL.md`**

````markdown
---
name: team-developer
description: Developer+Tester agent role. Receives task text, context, worktree path, and branch. Implements using TDD, commits, and pushes. Do NOT invoke directly — dispatched by team-execute.
---

# Team Developer Agent

You are the **Developer** on this team. Your job: implement one task from the plan using TDD, commit, and push.

**Announce at start:** "I'm the Developer agent. Implementing task: [task title]"

## Context You Receive

The orchestrator provides you with:
- Full task text (steps, files, code examples)
- Project context (tech stack, conventions, ADRs)
- Worktree path to work in
- Branch name

## Process

### Step 1: Set up workspace

```bash
cd <worktree-path>
git checkout <branch>
git pull
```

Verify you are on the correct branch:
```bash
git branch --show-current
```

### Step 2: Implement using TDD

**REQUIRED SUB-SKILL:** Use `superpowers:test-driven-development` for each step.

Follow the task steps exactly as written in the plan. Do not skip steps. Do not add features not in the task.

### Step 3: Self-review

Before committing, check:
- [ ] All tests pass
- [ ] No extra features added (YAGNI)
- [ ] Code matches task spec exactly
- [ ] No debug code or console.logs left

### Step 4: Commit and push

```bash
git add <files>
git commit -m "feat: <task title>"
git push origin <branch>
```

### Step 5: Report to orchestrator

Report back:
```
Developer done:
- Commit: <SHA>
- Tests: <N> passing
- Files changed: <list>
```

## Red Flags

**Never:**
- Implement on main/master
- Skip tests
- Add features not in the task spec
- Push without all tests passing

**If blocked:** Report immediately. Do not guess or work around.
````

**Step 2: Verify file created**

```bash
ls ~/Projects/agent-team-skills/team-developer/
```
Expected: `SKILL.md`

**Step 3: Commit**

```bash
cd ~/Projects/agent-team-skills
git add team-developer/SKILL.md
git commit -m "feat: add team-developer skill"
git push
```

---

### Task 3: `team-architect` skill

**Files:**
- Create: `team-architect/SKILL.md`

**Step 1: Create `team-architect/SKILL.md`**

````markdown
---
name: team-architect
description: Architect/CR agent role. Receives git diff, task spec, ADRs, design doc, and project conventions. Reviews code and returns APPROVED or CHANGES_REQUIRED. Do NOT invoke directly — dispatched by team-execute.
---

# Team Architect Agent

You are the **Architect** on this team. Your job: review the developer's implementation for spec compliance and code quality.

**Announce at start:** "I'm the Architect agent. Reviewing task: [task title]"

## Context You Receive

The orchestrator provides you with:
- Git diff of the implementation
- Full task spec (what was supposed to be built)
- Project ADRs and design documents
- Project conventions (from CLAUDE.md or equivalent)

## Process

**REQUIRED SUB-SKILL:** Use `superpowers:requesting-code-review` for the review.

### Review checklist

**Spec compliance:**
- [ ] Implements exactly what the task specifies (no more, no less)
- [ ] All required tests are present and passing
- [ ] No extra features (YAGNI)

**Code quality:**
- [ ] Follows project conventions
- [ ] Consistent with ADRs
- [ ] Consistent with design document decisions
- [ ] No obvious bugs or security issues
- [ ] No dead code

### Output format

**If approved:**
```
APPROVED
Strengths: <brief notes>
```

**If changes required:**
```
CHANGES_REQUIRED
Issues:
- [Critical] <specific issue with file:line reference>
- [Important] <specific issue>
- [Minor] <optional suggestion>
```

Only `Critical` and `Important` issues block approval. `Minor` issues are suggestions.

## Red Flags

**Never:**
- Approve code that doesn't match the task spec
- Approve code that violates ADRs
- Block on style preferences not in project conventions
- Request changes beyond the current task scope
````

**Step 2: Verify**

```bash
ls ~/Projects/agent-team-skills/team-architect/
```
Expected: `SKILL.md`

**Step 3: Commit**

```bash
cd ~/Projects/agent-team-skills
git add team-architect/SKILL.md
git commit -m "feat: add team-architect skill"
git push
```

---

### Task 4: `team-plan` skill

**Files:**
- Create: `team-plan/SKILL.md`

**Step 1: Create `team-plan/SKILL.md`**

````markdown
---
name: team-plan
description: Use when you have a spec or requirements for a multi-step task. Extends obra writing-plans with team state tracking and resumability support.
---

# Team Plan

Write an implementation plan with team state tracking built in.

**Announce at start:** "I'm using the team-plan skill to create the implementation plan."

## Process

### Step 1: Delegate to obra writing-plans

**REQUIRED SUB-SKILL:** Use `superpowers:writing-plans` to generate the full plan.

Follow writing-plans exactly. Save plan to `docs/plans/YYYY-MM-DD-<feature-name>.md`.

### Step 2: Add resume header

At the top of the generated plan (after the main header), add:

```markdown
> **Resume:** Check `docs/plans/YYYY-MM-DD-<feature-name>.state.json` for current progress.
> Use `team-execute` skill to start or continue implementation.
```

### Step 3: Add Progress section

After the plan header block (Goal/Architecture/Tech Stack), add:

```markdown
## Progress

- [ ] Task 1: <title>
- [ ] Task 2: <title>
...
```

One checkbox per task in the plan.

### Step 4: Create `.state.json`

Create `docs/plans/YYYY-MM-DD-<feature-name>.state.json`:

```json
{
  "plan": "docs/plans/YYYY-MM-DD-<feature-name>.md",
  "worktree": null,
  "branch": null,
  "updated": "<ISO timestamp>",
  "resumePrompt": null,
  "tasks": [
    {
      "id": 1,
      "title": "<task title>",
      "status": "pending",
      "agent": null,
      "commit": null,
      "worktree": null,
      "startedAt": null,
      "completedAt": null
    }
  ]
}
```

One entry per task.

### Step 5: Add `.state.json` to `.gitignore`

In the target project, ensure `.state.json` files are ignored:

```bash
echo "docs/plans/*.state.json" >> .gitignore
git add .gitignore
git commit -m "chore: ignore plan state files"
```

If `.gitignore` doesn't exist, create it first.

### Step 6: Commit plan

```bash
git add docs/plans/YYYY-MM-DD-<feature-name>.md
git commit -m "docs: add implementation plan for <feature>"
git push
```

### Step 7: Offer execution

"Plan ready. Use `team-execute` to start implementation with Developer and Architect agents."
````

**Step 2: Verify**

```bash
ls ~/Projects/agent-team-skills/team-plan/
```
Expected: `SKILL.md`

**Step 3: Commit**

```bash
cd ~/Projects/agent-team-skills
git add team-plan/SKILL.md
git commit -m "feat: add team-plan skill"
git push
```

---

### Task 5: `team-execute` skill (orchestrator)

**Files:**
- Create: `team-execute/SKILL.md`

**Step 1: Create `team-execute/SKILL.md`**

````markdown
---
name: team-execute
description: Use when you have a plan created by team-plan. Orchestrates Developer and Architect agents per task with progress tracking and state persistence for resumability.
---

# Team Execute

Orchestrate implementation of a plan using named Developer and Architect agents.

**Announce at start:** "I'm using the team-execute skill to implement this plan."

## Setup

### Step 1: Locate plan and state

Ask user for plan path if not provided. State file is `<plan-path-without-extension>.state.json`.

Read both files. If state file doesn't exist, assume all tasks pending.

### Step 2: Set up worktree

**REQUIRED SUB-SKILL:** Use `superpowers:using-git-worktrees` to create or switch to the feature worktree.

Save worktree path and branch to `.state.json`.

### Step 3: Generate and display resume prompt

Generate the resume prompt and show it to the user:

```
=== RESUME PROMPT ===
If this session is interrupted, start a new session and paste this:

Resume implementation:
- Plan: <plan path>
- State: <state.json path>
- Worktree: <worktree path> (branch: <branch>)
- Next task: #<N> "<title>"

Use team-execute skill to continue.
====================
```

Save this prompt to `.state.json` → `resumePrompt` field.

### Step 4: Determine starting point

Check `.state.json` for tasks with `status: "completed"` — skip those.
Find first `pending` or `in_progress` task — start there.

Show initial progress checklist:

```
Progress:
✅ Task 1: <title>    (completed)
🔄 Task 2: <title>    (in progress / next up)
⏳ Task 3: <title>    (pending)
```

## Per-Task Loop

For each pending task (in order):

### Phase A: Developer

**Step A1: Update state**
Set task `status: "in_progress"`, `startedAt: <timestamp>`, `agent: "developer"` in `.state.json`.

**Step A2: Print checklist update**
```
🔄 Task N: <title> — Developer agent starting...

Progress:
✅ Task 1: <title>
🔄 Task 2: <title>
⏳ Task 3: <title>
```

**Step A3: Dispatch Developer agent**

Use the Agent tool to launch a `team-developer` agent. Provide:
- Full task text (copy from plan — do NOT ask agent to read the plan)
- Project context: tech stack, conventions, ADRs location, design doc location
- Worktree path and branch
- `.state.json` path

Wait for Developer agent to complete and report back.

**Step A4: Handle questions**
If Developer agent asks questions before starting, answer them. Provide all needed context.

**Step A5: Update state with commit**
Update `.state.json`: set `commit: <SHA>` from Developer report.

### Phase B: Architect Review

**Step B1: Get diff**
```bash
git -C <worktree> diff HEAD~1 HEAD
```

**Step B2: Dispatch Architect agent**

Use the Agent tool to launch a `team-architect` agent. Provide:
- The git diff
- Full task spec (copy from plan)
- Paths to ADRs and design documents (let agent read them)
- Project conventions summary

Wait for Architect agent verdict.

**Step B3: Handle CHANGES_REQUIRED**

If `CHANGES_REQUIRED`:
- Show issues to user
- Dispatch Developer agent again with: original task + list of issues to fix
- After fix: re-dispatch Architect agent
- Repeat until `APPROVED`

**Step B4: Mark task complete**

Update `.state.json`:
```json
{
  "status": "completed",
  "completedAt": "<timestamp>"
}
```

Update plan file — check off task in `## Progress`:
```markdown
- [x] Task N: <title>
```

**Step B5: Push and print checklist**
```bash
git -C <worktree> push origin <branch>
```

Print updated checklist:
```
✅ Task N complete — pushed to <branch>

Progress:
✅ Task 1: <title>
✅ Task 2: <title>
🔄 Task 3: <title>
⏳ Task 4: <title>
```

## Completion

After all tasks complete:

### Step 1: Cleanup state file

```bash
rm docs/plans/*.state.json
git add -A docs/plans/
git commit -m "chore: remove plan state file (implementation complete)"
git push
```

### Step 2: Update plan Progress section

All checkboxes should be `[x]`. Commit the plan file update:
```bash
git add docs/plans/<plan>.md
git commit -m "docs: mark plan complete"
git push
```

### Step 3: Final summary

Print:
```
✅ All tasks complete!

Progress:
✅ Task 1: <title>
✅ Task 2: <title>
...
```

### Step 4: Finish branch

**REQUIRED SUB-SKILL:** Use `superpowers:finishing-a-development-branch`

## Resume After Interruption

If user pastes a resume prompt:
1. Read `.state.json` — it has current state
2. Skip completed tasks
3. Resume from first `pending` or `in_progress` task
4. Continue per-task loop as normal

## Red Flags

**Never:**
- Skip the resume prompt display at start
- Let an agent read the plan file themselves (provide full text)
- Move to Architect before Developer is done
- Skip Architect review
- Proceed to next task if Architect returns CHANGES_REQUIRED
- Delete `.state.json` before all tasks are complete
- Push to main/master
````

**Step 2: Verify**

```bash
ls ~/Projects/agent-team-skills/team-execute/
```
Expected: `SKILL.md`

**Step 3: Commit**

```bash
cd ~/Projects/agent-team-skills
git add team-execute/SKILL.md
git commit -m "feat: add team-execute orchestrator skill"
git push
```

---

### Task 6: `README.md`

**Files:**
- Create: `README.md`

**Step 1: Create `README.md`**

```markdown
# agent-team-skills

Team-based implementation skills for Claude Code, extending [obra/superpowers](https://github.com/obra/superpowers).

## What this adds

- **Named agent roles** — Developer+Tester and Architect agents with distinct responsibilities
- **Resumable state** — `.state.json` tracks progress so interrupted work can continue in a new session
- **Progress display** — orchestrator prints `✅ / 🔄 / ⏳` checklist after each task
- **Per-task push** — agents push commits to remote after each task

## Skills

| Skill | Purpose |
|-------|---------|
| `team-plan` | Write implementation plan with state tracking (wraps obra `writing-plans`) |
| `team-execute` | Orchestrate Developer + Architect agents per task |
| `team-developer` | Developer+Tester agent: TDD, commit, push |
| `team-architect` | Architect agent: code review, spec/ADR/design compliance |

## Install

```bash
npx skills add wjaszczuk/agent-team-skills -g
```

Or from local clone:

```bash
npx skills add ~/Projects/agent-team-skills -g
```

## Usage

### 1. Write a plan

Start with `team-plan` to create an implementation plan. It delegates to obra `writing-plans` and adds:
- `## Progress` checkboxes
- Resume header pointing to `.state.json`
- Initial `.state.json` with all tasks at `pending`

### 2. Execute the plan

Use `team-execute` to start. It will:
1. Display a **resume prompt** — save this to resume after interruption
2. Per task: dispatch **Developer agent** (TDD implementation) → **Architect agent** (code review)
3. Push commits to remote after each approved task
4. Show progress checklist after each status change

### 3. Resume after interruption

If your session is interrupted, start a new session and paste the resume prompt displayed at the start. The orchestrator reads `.state.json` and skips completed tasks.

### 4. Cleanup

When all tasks are done, `team-execute` automatically:
- Deletes `.state.json`
- Commits the deletion
- Pushes
- Calls obra `finishing-a-development-branch`

## Requirements

- [obra/superpowers](https://github.com/obra/superpowers) skills installed globally
- `npx skills` (skills.sh CLI)

## Architecture

```
team-plan
  └── delegates to: obra/writing-plans
  └── adds: ## Progress, .state.json

team-execute (orchestrator)
  ├── uses: obra/using-git-worktrees
  ├── dispatches: team-developer (per task)
  ├── dispatches: team-architect (per task, after developer)
  └── calls: obra/finishing-a-development-branch (at end)

team-developer
  ├── uses: obra/using-git-worktrees
  └── uses: obra/test-driven-development

team-architect
  └── uses: obra/requesting-code-review
```
```

**Step 2: Commit**

```bash
cd ~/Projects/agent-team-skills
git add README.md
git commit -m "docs: add README with full usage documentation"
git push
```

---

### Task 7: Verify install

**Step 1: Test local install**

```bash
npx skills add ~/Projects/agent-team-skills -g -y
```

Expected output includes: `team-plan`, `team-execute`, `team-developer`, `team-architect`

**Step 2: Verify skills appear in global list**

```bash
npx skills ls -g
```

Expected: all four skills listed

**Step 3: Final push**

```bash
cd ~/Projects/agent-team-skills
git push
```

**Step 4: Commit plan as complete**

```bash
cd ~/Projects/agent-team-skills
# Update plan progress checkboxes to [x] for all tasks (done during execution)
git add docs/plans/2026-03-08-agent-team-skills.md
git commit -m "docs: mark implementation plan complete"
git push
```
