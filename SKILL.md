---
name: aki-planner
description: >-
  Build overnight execution playbooks for autonomous AI agents. Audits the
  codebase, runs structured founder Q&A (8 forcing questions), generates
  budget-aware phase-gated runbooks with session-reset watchdogs and checkpoint
  protocols. Use when asked to "plan overnight work", "create a playbook",
  "orchestrate opus", "autonomous agent plan", "make a runbook", or "plan for
  claude to run overnight". Proactively suggest when the user has a multi-phase
  project and wants an AI agent to execute autonomously while they sleep.
  Do NOT use for single-session tasks, pure brainstorming (use office-hours),
  or code review (use review skill).
---

# Aki Planner

Build execution playbooks that let autonomous AI agents ship real product work
overnight. Not a generic plan. A tight, executable runbook with phases, exit
gates, budget management, and fallback protocols.

<HARD-GATE>
Do NOT write any playbook content until Phase 3 (FOUNDER Q&A) is complete and
the user has approved the scope. Phases 1-3 are diagnostic only.
</HARD-GATE>

## Question format

For every question, follow this structure:

1. **Context**: State what we're building and where we are (1 sentence)
2. **Plain English**: Explain so a non-engineer could follow
3. **Your recommendation**: `RECOMMENDATION: [letter] because [reason]`
4. **Options**: Lettered choices: `A) ... B) ... C) ...`

Ask one question per message. Wait for the answer before proceeding.

## Reframing rules

When answers are vague, push back:
- "Build the whole thing" → "If the agent finishes ONE thing tonight, what's the demo you show tomorrow?"
- "Make it work" → "What specific screen does the user see? Walk me through the 60-second demo."
- "Fix everything" → "What's broken vs what's mocked vs what's missing? Each is different work."

Never accept scope that covers more than 3-4 P0 items without flagging it.

---

## Phase 1: INTAKE

Ask one open question:

> "What do you want the agent to build overnight? Describe the product, where
> it is today, and what 'done' looks like when you wake up."

Listen for:
- Working codebase or starting from scratch?
- P0 (must-have for demo) vs P1 (nice-to-have)?
- Reference repos, designs, or prior art?
- Deployment target?

If vague, push back before continuing.

## Phase 2: AUDIT

Before writing any plan, audit everything. **Non-negotiable.**

### 2.1 Read all project docs

Read every `.md` in the project root. Prioritize:
- README, PRD, MASTER_PLAN, BUILD_PLAN, CHANGELOG
- DESIGN, ARCHITECTURE, CONTRIBUTING
- CLAUDE.md or AGENTS.md (agent instructions)
- Any `docs/` directory

### 2.2 Audit the codebase

Use explore subagents or direct reads:

| What | Why |
|------|-----|
| DB schema (all tables, relationships) | What exists vs what's mocked |
| API routes | What's wired vs stub |
| Components | What's built vs placeholder |
| Dependencies (package.json) | What's installed vs missing |
| Auth | What's configured vs hardcoded |
| Tests | What exists, coverage level |

### 2.3 Read reference repos

If user mentions other repos or prior work, read their key files. Extract:
- Architecture decisions and prompts
- Data models and scoring rubrics
- What can be ported vs rewritten

### 2.4 Present findings

> "Here's what I found:
> - **Working:** [list]
> - **Mocked/stub:** [list]
> - **Missing:** [list]
> - **Key decisions from prior work:** [list]
> - **Blockers:** [list]"

## Phase 3: FOUNDER Q&A

8 forcing questions, asked one at a time. Skip any already answered. **Adapt
questions to the specific project** — these are templates, not a rigid script.

### Q1: Users and data reality

> "Who is the primary user? Is the app running on real data or dummy/seed data?"

If dummy: data import becomes P0. Always.

### Q2: The 60-second demo

> "If you had 60 seconds to demo this tomorrow, what would you show?
> Walk me through screen by screen."

This defines the critical path. Everything else is secondary.

### Q3: What works vs what's mocked

> "What works end-to-end today? What looks like it works but is mocked?
> What's completely missing?"

Cross-reference with audit findings. Challenge discrepancies.

### Q4: AI/automation components

> "Does this product have AI features? What model, provider, prompt
> architecture? Is it wired up or just designed?"

Get specifics: model provider + key, prompt structure, where prompts live.

### Q5: Auth and multi-tenancy

> "How does auth work? Is it wired into API routes or are queries hardcoded
> to a default user/org?"

Hardcoded auth is extremely common in MVPs and breaks everything downstream.

### Q6: External dependencies

> "What external services does this need? Database, storage, APIs, webhooks?
> Are credentials in .env? Which ones are missing?"

### Q7: Budget and session limits

> "What's your AI usage budget for tonight? Show me your usage dashboard.
> When does your session limit reset?"

**Critical for planning.** Without this the agent burns through limits mid-work.

### Q8: Scope negotiation

After all answers:

> "Based on everything, here's what's achievable tonight:
>
> **Must ship (P0):** [3-4 items max]
> **Should ship (P1):** [4-6 items]
> **If time allows (P2):** [list]
>
> Does this match your priorities?"

Wait for approval before writing the playbook.

---

## Phase 4: WRITE

Generate the playbook as a single markdown file. Follow this exact structure:

### Playbook skeleton

```
# [Product] — Overnight Execution Playbook

## Who you are / scope
[Role, tonight's target, what's NOT tonight, priority order]

## Usage budget & execution strategy
[Budget snapshot, watchdog timer, checkpoint protocol, rate-limit handling]

## Context
[Product summary, tech stack, reference repos, architecture details]

## Phase 0 — Setup & recon
[Health check, doc reads, auth wiring, seed data, screenshots]

## Phase 1-N — [P0/P1]: [Title]
[Steps with exact file paths, commands, API shapes, exit gates]

## Environment
[Env vars, dependencies to install, DB setup commands]

## Discipline
[Git rules, DB rules, i18n, blocked-state handling]

## Backlog
[P0/P1/P2/P3 items mapped to phases, effort estimates]

## Strategic context
[Why this matters, competitive context, user outcomes]

## Session history
[Checkpoint log — agent appends here after each phase]

## Multi-session plan
[What happens tomorrow, day after, weekly budget allocation]
```

### Writing principles

**Executable, not aspirational.** Every instruction must be actionable.

| Bad | Good |
|-----|------|
| "improve the UI" | "read `src/app/dashboard/page.tsx`, add loading skeleton using existing `Skeleton` from `components/ui`" |
| "run the seed" | `npx tsx db/seed.ts` |
| "the auth helper" | `src/lib/api-auth.ts` |
| "add an API endpoint" | Full `POST /api/foo` with request/response TypeScript types |

**Reference existing code.** "Use the same pattern as `src/app/api/buildings/route.ts`."

**Exit gates per phase.** Testable condition + commit message template:
```
**Exit gate:** [condition]. Commit: `type(phase-N): description`
```

### Budget section (always include)

Required elements:
1. Current usage snapshot (session %, weekly %)
2. Session reset time and timezone
3. Watchdog timer script (adapt to user's timezone):

```bash
DIFF=$(python3 -c "
import subprocess, os, time
result = subprocess.run(['date', '-j', '-v+0d', '-f', '%H:%M', 'RESET_HOUR:00', '+%s'],
  capture_output=True, text=True, env={**os.environ, 'TZ': 'USER_TZ'})
target = int(result.stdout.strip())
now = int(time.time())
d = target - now
print(d if d > 0 else d + 86400)
")
rm -f /tmp/SESSION_MARKER
sleep "$DIFF"
touch /tmp/SESSION_MARKER
```

4. Rate-limit protocol: commit → push → wait for marker → resume
5. Priority table: what to cut if budget runs low

### Degree of freedom per step type

| Step type | Freedom | Format |
|-----------|---------|--------|
| DB migrations, auth wiring | Low | Exact commands, exact file paths |
| API endpoints | Low-Medium | Request/response types, handler pseudocode |
| UI components | Medium | Which component to extend, what to add, design reference |
| Design polish, copy | High | Principles + examples, not 50-step scripts |

### Validation loops

Each phase should include a verify step:
- `npx tsc --noEmit` before committing
- `npx drizzle-kit push --dry-run` after schema changes
- Screenshot verification after UI phases (GStack `/browse` or similar)

## Phase 5: REVIEW

Validate the playbook against this checklist:

```
Playbook Quality:
- [ ] Every phase has an exit gate with testable condition
- [ ] Every phase has a commit message template
- [ ] All file paths are exact (not "the config file")
- [ ] All new API endpoints have request/response types
- [ ] Budget section has watchdog timer with correct timezone
- [ ] Rate-limit protocol is documented
- [ ] Env vars listed (required vs optional)
- [ ] Missing dependencies have exact install commands
- [ ] Reference repo files have exact access commands
- [ ] No phase requires user input (autonomous execution)
- [ ] P0 items come before P1, P1 before P2
- [ ] Strategic context explains WHY for agent decision-making
- [ ] Multi-session plan accounts for weekly budget
- [ ] No empty sections after headings
- [ ] Under 1500 lines (split to reference files if over)
```

## Phase 6: FOUNDER REVIEW

Present the playbook:

> "Here's the overnight playbook. Three things to check:
>
> 1. **Is the P0 scope right?** These items the agent MUST finish.
> 2. **Any env vars or credentials missing?**
> 3. **Is the budget plan realistic?**"

Iterate until the user says it's ready. Then save the file and commit.

---

## Gotchas

| Situation | What to do |
|-----------|-----------|
| User says "do everything" | Push back. 3-4 P0 items max. Rest is stretch. |
| No .env or missing API keys | Flag as blocker. Agent wastes budget on failed API calls. |
| Reference repo is private | Use `gh api` commands. Verify token access first. |
| Session reset time unknown | Ask for usage dashboard screenshot. Calculate from timezone. |
| Schema drift possible | Include `drizzle-kit push --dry-run` (or equivalent) in Phase 0. |
| User wants agent to deploy to prod | Include canary/verification steps. Never push blind. |
| Playbook over 1500 lines | Split: main playbook + `references/` files. |

## Anti-rationalization

| Thought the agent might have | Reality |
|-------------------------------|---------|
| "I'll skip the audit, figure it out as I go" | The audit prevents 80% of wrong turns. Never skip Phase 2. |
| "Budget is fine, I'll do all phases" | Budget-aware means stopping cleanly, not crashing mid-phase. |
| "This stub is too simple to read first" | Read it. It's probably 80% of what you need. |
| "I'll generate the schema from scratch" | Check existing schema first. It probably exists. |
| "I'll commit at the end" | Commit per phase. Progress lost to a crash is unrecoverable. |
| "I can skip the watchdog timer" | Session limits are real. The timer is the safety net. |

---

## Learnings (from real overnight builds)

These patterns emerged from building Atelier's overnight playbook (2026-04-15):

1. **Data import is always P0.** Dummy data = nothing else matters.
2. **AI wiring is the hardest phase.** Budget extra time for prompt porting.
3. **Existing components are gold.** Audit for 80%-done mocks. Wire them, don't rebuild.
4. **Auth is always half-done in MVPs.** Check if API routes use auth or hardcode defaults.
5. **The session-reset watchdog works.** Background sleep → marker file → agent survives limits.
6. **Reference repos contain canon.** Latest prompts/rubrics are usually in a different repo.
7. **Seed data needs real names/addresses.** Demo credibility depends on recognizable data.
8. **Pre-scripted demo scenarios save the demo.** 3-5 canned inputs exercising different tiers.
9. **Commit per phase, push per phase.** Progress must never be lost.
10. **Locale-first is easy to forget.** Add an i18n section if the product isn't English-default.

These learnings are injected into Phase 4 (WRITE) automatically. Update this list
as new overnight builds reveal new patterns.
