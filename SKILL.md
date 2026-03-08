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
