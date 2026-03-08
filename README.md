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
