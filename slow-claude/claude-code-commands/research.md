# Research Phase — for Claude Code

You are researching, NOT implementing. End deliverable is ONE markdown file at repo root: `RESEARCH-$ARGUMENTS.md`. Do not write any application code. Do not install anything in the main app.

## Time budget
~10 minutes. Focused research.

## Branch setup (do this first)
```bash
git checkout main
git pull
git checkout -b feature/$ARGUMENTS
```
All research and (later) implementation lives on this branch. Push the branch to remote after creating `RESEARCH-$ARGUMENTS.md`.

## The task
The user will describe in the next message what feature they want to add and what options they want compared. Wait for it before exploring.

## EXPLORE phase
1. Read `CLAUDE.md` (or project root context) in full
2. Find and read the affected components in the codebase (use grep/glob — do not guess paths)
3. Document where `dir` is set, font system, relevant schema fields, existing related code

## Research the candidates
For each candidate the user names, read official docs and GitHub repo. Find evidence on:
1. **Critical requirement** (RTL, accessibility, offline, etc. — the user will name it) — search GitHub issues with relevant keywords, link issue numbers
2. **Custom styling / component API** — code examples from official docs
3. **Async / timing handling** — what happens when target/dependency not yet ready
4. **Mobile / touch** — open issues
5. **Multi-instance state** — built-in or wrapper required
6. **TypeScript** — built-in types or `@types/*`
7. **Bundle size** — measure from `npm pack <pkg>` and gzip the dist files

## Ask the user clarifying questions
Before writing `RESEARCH-$ARGUMENTS.md`, ask at minimum:
- Persistence model (DB / localStorage / both)?
- Trigger model (auto / manual / both)?
- Scope (per user / per resource / global)?
- Empty state behavior (silent / fallback / error)?

Do NOT guess. Wait for answers.

## Output: RESEARCH-$ARGUMENTS.md
Structure:
1. Our codebase findings (files, targets, dir, fonts, schema)
2. Option A — evidence with citations
3. Option B — evidence with citations
4. Recommendation — 3 reasons tied to OUR codebase, 1 paragraph why rejected option is wrong here
5. Open questions for the user

## Constraints
- Do NOT install packages in main app
- Do NOT modify code
- Do NOT write file before asking clarifications
- Push branch when done
- 300–500 lines max
