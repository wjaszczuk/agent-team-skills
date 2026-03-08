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
| `team-brainstorming` | Design a feature (wraps obra `brainstorming`, hands off to `team-plan`) |
| `team-plan` | Write implementation plan with state tracking (wraps obra `writing-plans`) |
| `team-execute` | Orchestrate Developer + Architect agents per task |
| `team-developer` | Developer+Tester agent: TDD, commit, push |
| `team-architect` | Architect agent: code review, spec/ADR/design compliance |

## Install

```bash
npx skills add wjaszczuk/agent-team-skills -g
```

This installs each skill separately into `~/.agents/skills/`:

```
~/.agents/skills/team-brainstorming
~/.agents/skills/team-plan
~/.agents/skills/team-execute
~/.agents/skills/team-developer
~/.agents/skills/team-architect
```

Or from local clone:

```bash
npx skills add /path/to/agent-team-skills -g
```

### Update

```bash
npx skills update -g
```

### Uninstall

```bash
npx skills remove team-brainstorming team-plan team-execute team-developer team-architect -g -y
```

## Usage

### 1. Brainstorm and design

Start with `team-brainstorming`. It runs obra `brainstorming` (clarifying questions, design approval, design doc) and then automatically hands off to `team-plan` instead of obra `writing-plans`.

### 2. Write a plan (standalone)

If you already have a design, use `team-plan` directly. It delegates to obra `writing-plans` and adds:
- `## Progress` checkboxes
- Resume header pointing to `.state.json`
- Initial `.state.json` with all tasks at `pending`

### 3. Execute the plan

Use `team-execute` to start. It will:
1. Display a **resume prompt** — save this to resume after interruption
2. Per task: dispatch **Developer agent** (TDD implementation) → **Architect agent** (code review)
3. Push commits to remote after each approved task
4. Show progress checklist after each status change

### 4. Resume after interruption

If your session is interrupted, start a new session and paste the resume prompt displayed at the start. The orchestrator reads `.state.json` and skips completed tasks.

### 5. Cleanup

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
team-brainstorming
  └── delegates to: obra/brainstorming
  └── replaces final step: obra/writing-plans → team-plan

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
