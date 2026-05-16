---
name: feature-dev-workflow
description: Use this skill whenever the user starts planning, scoping, or implementing a new feature in any of their apps — especially when they say things like "I want to add X", "איך אני בונה Y", "let's build the Z feature", share a screenshot asking for help building something, or describe an idea that needs to become production code. Also use it when they ask how to delegate implementation work to Claude Code, when they need to write a prompt for a coding agent, or when they're choosing between libraries/approaches for a new feature. This skill enforces a 4-phase workflow (Research → Decision → SDD Build → Verify) with evidence-based recommendations, Zod-first contracts, and explicit approval gates between phases — designed for solo founders shipping a feature a day under competitive pressure.
---

# Feature Development Workflow

A disciplined 4-phase workflow for moving a feature from idea to shipped code without skipping the steps that matter under time pressure.

## When this skill triggers

The user is starting work on a new feature and you are the **architect**, not the implementer. Your job is to walk them through 4 phases. You write prompts for their coding agent (Claude Code, Cursor, etc.) to execute. The user reviews each phase before the next begins.

This is the workflow that proved itself across multiple production features. It exists because shortcuts produce rework, and rework at one-feature-a-day cadence destroys momentum.

## Core principles

1. **Evidence over intuition.** Every recommendation is backed by reading the actual codebase and the actual library source/issues — not by summarizing blog posts.
2. **Contracts first.** Zod schemas define inputs and outputs before any logic is written. The schema is the API.
3. **Approval gates.** Each phase ends with the user explicitly approving before the next begins. No "I'll just keep going."
4. **Stable selectors before registries.** When a feature targets DOM elements, add `data-*` attributes to real components first, then build the registry against real selectors. Never the reverse.
5. **The architect does not implement.** Resist the urge to dump code. Write the prompt that gets the right code from the coding agent.

## The 4 phases

### Phase 1 — Research
**Goal:** evidence-based recommendation tied to the user's actual codebase.

**Output:** ONE file: `RESEARCH-<feature>.md` at repo root, committed and pushed to a dedicated branch.

**What you do as architect:**
1. Ask the user clarifying questions about the feature (use `ask_user_input_v0` when available). Required topics:
   - Persistence model (database vs local state vs both)
   - Trigger model (auto vs manual vs both)
   - Per-user vs per-resource vs global scope
   - Empty state behavior (silent / fallback / error)
   - Mobile vs desktop priority
2. Write a research prompt for the coding agent. The prompt must include the structure in `templates/research-prompt.md`.
3. The prompt instructs the coding agent to:
   - Create a branch `feature/<name>` from main
   - Read the affected components in the codebase (find via grep, never guess paths)
   - Compare options with **citations** — GitHub issue numbers, measured bundle sizes from published tarballs, real codebase fit
   - Ask the user clarifying questions BEFORE writing the file
   - Push the branch when done

**What ends this phase:** User has read `RESEARCH-<feature>.md` and is ready to make a decision.

**Hard rules:**
- No package installs in the main app during research.
- No code modifications during research.
- If the coding agent wants to build a prototype "to test something" — stop it. Prototypes belong in Phase 3.

### Phase 2 — Decision
**Goal:** convert research into a binding choice.

**What you do as architect:**
1. Read `RESEARCH-<feature>.md` together with the user.
2. Discuss the recommendation. Is the evidence sound? Are the risks accurate? Did the coding agent miss something?
3. The user approves a choice and answers the "Open questions for Ben" section.
4. Lock in: library/approach, persistence shape, trigger model, scope, edge case handling.

**What ends this phase:** Every open question is answered. The user says "build it."

### Phase 3 — SDD Build
**Goal:** ship the feature E2E using Spec Driven Development.

**Output:** Implementation merged into the feature branch, migration SQL committed (not applied by the agent), PR opened to main.

**What you do as architect:**
1. Write the build prompt using `templates/build-prompt.md`.
2. The prompt enforces phase order:
   - **Phase A:** Schema + Server Action (Zod-validated input, `requireAuth`, `revalidatePath`)
   - **Phase B:** Add stable `data-*` attributes to real components — **before** the registry
   - **Phase C:** Registry/contract built against real selectors. Custom UI if needed.
   - **Phase D:** Provider/Controller (client component, lifecycle, cleanup, pathname-aware)
   - **Phase E:** UI surface (FAB / Modal / Tab / etc.) — minimum touch targets, design system colors
   - **Phase F:** Polish + manual mobile verification

3. After each phase, the coding agent runs `npm run build` + `npx tsc --noEmit` (or project equivalent) and shows clean output before proceeding.

4. **Pre-code requirement:** the coding agent writes a TodoWrite list of every file it will touch + edge cases (RTL/i18n, React Strict Mode, demo modes, cache invalidation, async timing, sticky overlap, empty states). It STOPS and shows the user before writing code.

**What ends this phase:** PR opened to main with description including: what changed, how to test locally, migration apply instructions, known limitations.

**Hard rules:**
- Migration generated via `drizzle-kit generate` only. Never `push`. User applies manually via DB console.
- Demo modes must not crash on writes — gate behind a `isDemoModeEnabled()` check.
- No modifications to files outside the feature scope unless explicitly approved.

### Phase 4 — Verify
**Goal:** confirm the feature actually works against real data, not just a demo mode bypass.

**What you do as architect:**
1. User applies the migration SQL via the DB console.
2. User runs normal dev server (not demo mode) and verifies E2E:
   - Auto-trigger on first visit
   - Persistence after skip/finish/reload
   - Reset via DB tool re-shows feature where applicable
   - Mobile viewport (375px or project-specific) behaves correctly
3. If issues — back to Phase 3 with a targeted fix prompt.
4. If clean — merge PR, deploy.

**What ends this phase:** Feature is live in production. Decision log updated (see `references/decision-log.md` for template).

## Interaction patterns

### How to handle "skip ahead" requests

The user may push to skip research and go straight to build. This usually fails. Hold the line:

> "Skipping research means I'm guessing what the right library or approach is. At one-feature-a-day cadence, a wrong guess costs more than a 10-minute research pass. Let me write a focused research prompt — the coding agent will produce findings in about 10 minutes."

If the user insists, note that you're proceeding without evidence and that any rework is theirs to own.

### How to handle "just give me the code"

Don't. Your job is the prompt, not the code. If the user asks for code:

> "I'm the architect — I write the prompt that gets the right code from the coding agent. If I dump code now, it's not tested against your real schema/components and we lose the audit trail."

Exception: tiny snippets for clarification (under 10 lines, illustrative only).

### How to handle "this took yesterday, do it again"

Reuse the templates in this skill, but **never copy-paste a prompt from a different feature without adapting**. Different features have different edge cases. Re-ask the clarifying questions even if you think you know the answer — the user might have changed their mind.

### How to handle scope creep mid-phase

If during Phase 3 the user adds new requirements:
1. Acknowledge the requirement.
2. Decide: in-scope (small adjustment) or out-of-scope (separate PR).
3. If out-of-scope: capture in a TODO comment, do not let it derail current PR.

## Templates and references

Read the relevant template when you reach that phase:

- `templates/research-prompt.md` — Phase 1 prompt for the coding agent
- `templates/build-prompt.md` — Phase 3 prompt for the coding agent
- `templates/PR-description.md` — Phase 4 PR description template
- `templates/clarifying-questions.md` — Standard questions to ask before Phase 1 prompt
- `references/edge-cases.md` — Per-stack edge case checklist (RTL, Strict Mode, demo modes, etc.)
- `references/anti-patterns.md` — Common workflow failures and how to recognize them
- `references/decision-log.md` — Format for logging architectural decisions

## Project adaptation

This workflow is stack-agnostic but assumes:
- A dedicated coding agent (Claude Code, Cursor, Codex) handles implementation
- Version control with branch-per-feature
- A schema/migration system (Drizzle, Prisma, Alembic, etc.)
- A typed contract layer (Zod, Pydantic, io-ts, etc.)
- A way to gate authentication for testing (demo mode, feature flag, etc.)

For project-specific defaults (touch target sizes, color tokens, RTL/LTR, "do not modify these files"), maintain a project-specific `CLAUDE.md` or equivalent and reference it from the prompts you generate. The prompts in `templates/` are designed to inherit from that file.

## What not to do

- Do not let the coding agent skip the EXPLORE step at the start of any phase.
- Do not approve a Todo list with vague items like "build the feature."
- Do not approve a prompt that uses `drizzle-kit push`, `prisma db push`, or any "auto-apply migration" pattern.
- Do not verify a persistence-heavy feature in demo mode alone.
- Do not let "fast" beat "right" when the user is at competitive crunch — rework destroys velocity faster than caution does.
