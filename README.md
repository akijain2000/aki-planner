# aki-planner

Build overnight execution playbooks for autonomous AI agents (Opus, Claude Code, Codex).

## What it does

Runs a structured 6-phase flow to produce a tight, executable runbook:

1. **Intake** — What are we building? What's the demo?
2. **Audit** — Deep codebase read (schema, routes, components, auth, tests, reference repos)
3. **Founder Q&A** — 8 forcing questions asked one at a time (users, data reality, the demo, what works vs mocked, AI components, auth, deps, budget)
4. **Write** — Generates the playbook with phases, exit gates, budget management, watchdog timers
5. **Review** — 15-point checklist validation
6. **Iterate** — Founder review loop until ready

## Install

### Cursor

```bash
cp -r . ~/.cursor/skills/aki-planner/
```

### Claude Code

```bash
cp -r . ~/.claude/skills/aki-planner/
```

### Any SKILL.md-compatible agent

Just point it at `SKILL.md`.

## What the playbook looks like

The output is a single markdown file (~1000-1500 lines) with this structure:

```
1. WHO YOU ARE / SCOPE         — role, tonight's target, what's NOT tonight
2. USAGE BUDGET                — session limits, watchdog timer, checkpoint protocol
3. CONTEXT                     — product, tech stack, reference repos, architecture
4. CRITICAL PATH (phases)      — numbered phases with steps, exit gates, commit msgs
5. ENVIRONMENT                 — env vars, dependencies, DB setup
6. DISCIPLINE                  — git rules, DB rules, i18n, blocked-state handling
7. BACKLOG                     — P0/P1/P2/P3 items mapped to phases
8. STRATEGIC CONTEXT           — why this matters, competitive context
9. SESSION HISTORY             — checkpoint log (agent appends here)
10. MULTI-SESSION PLAN         — what happens tomorrow, budget allocation
```

## Key features

- **Budget-aware**: Plans around session limits and weekly quotas. Includes a background watchdog timer that detects session resets.
- **Phase-gated**: Every phase has an exit gate with a testable condition and commit message template.
- **Checkpoint protocol**: Agent commits and pushes after every phase. If rate-limited, it waits for reset and resumes.
- **Founder Q&A**: Not generic project planning. Structured questions that extract the context that makes or breaks autonomous execution.
- **Degree of freedom mapping**: DB migrations get exact commands. UI work gets principles and references.

## Origin

Built from the experience of creating Atelier's overnight execution playbook (April 2026) — an 11-phase runbook that took a Next.js property management dashboard from dummy data to working AI lead pipeline in one night.

## License

MIT
