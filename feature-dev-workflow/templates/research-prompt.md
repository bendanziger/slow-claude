# Research Prompt Template — Phase 1

> Adapt the placeholders in `{{ }}` before sending to the coding agent. Read each section — do not paste blindly.

---

```markdown
# {{FEATURE_NAME}} — Research & Recommendation ONLY

## Your role
You are researching, NOT implementing. End deliverable is ONE markdown file: `RESEARCH-{{FEATURE_NAME}}.md` at repo root. Do not write any application code. Do not install anything in the main app.

## Time budget
~10 minutes. Focused research, not exploration.

## Branch setup (do this first)
```bash
git checkout main
git pull
git checkout -b feature/{{FEATURE_NAME}}
```
All research and (later) implementation lives on this branch. Push the branch to remote after creating `RESEARCH-{{FEATURE_NAME}}.md`.

## The decision you're helping make
{{ONE_PARAGRAPH_PROBLEM_STATEMENT}}

The candidate options are:
- **{{OPTION_A}}** ({{OPTION_A_ONE_LINER}})
- **{{OPTION_B}}** ({{OPTION_B_ONE_LINER}})
{{OPTIONAL_OPTION_C}}

Pick one based on EVIDENCE from the codebase, not generic blog posts.

## EXPLORE phase

Read these files in full before writing anything:
- `CLAUDE.md` (or project equivalent)
- {{LIST_OF_AFFECTED_SCREEN_FILES_OR_COMPONENTS}}
- `{{SCHEMA_PATH}}` — pay attention to {{RELEVANT_TABLES_OR_TYPES}}
- {{LAYOUT_OR_ROOT_FILE}} — confirm i18n direction, font system, providers in scope

Use grep / glob / read. Do NOT install packages.

## Research the candidates

Read the official docs and GitHub repo for each candidate. Specifically gather evidence on:

1. **{{CRITICAL_REQUIREMENT_1}}** — search GitHub issues with relevant keywords. Report findings with issue numbers.
2. **Custom styling / component API** — how to provide UI that matches our design system ({{DESIGN_TOKENS}}). React-component-based or CSS-override-based?
3. **Async / timing handling** — what happens if the target/dependency is not yet ready? ({{SPECIFIC_TIMING_CONCERN_FROM_CODEBASE}})
4. **Mobile / touch behavior** — open issues on small viewports, touch event quirks
5. **Multi-instance state** — can it handle {{N}} independent instances with separate state, or is it built around a single instance?
6. **TypeScript types** — built-in or external `@types`?
7. **Bundle size** — claimed in README vs. measured from the published tarball. Run `npm pack <package>` and measure `dist/*` raw and gzipped.

## Ask the user clarifying questions

Before writing `RESEARCH-{{FEATURE_NAME}}.md`, if anything is unclear about the codebase or requirements, ASK. Wait for answers. Do NOT guess.

Examples of questions worth asking:
- "Should state persist per device or per user account?"
- "If a user has multiple {{resource}} — show this once ever, or once per {{resource}}?"
- "I see X in the code — did you mean Y?"

## Output: RESEARCH-{{FEATURE_NAME}}.md

```markdown
# {{FEATURE_NAME}} Research

## 1. Our codebase findings
- Affected screens with exact file paths
- For each screen: 3–6 key UI elements relevant to this feature (with selector or component name)
- Where i18n direction (`dir="rtl"` or equivalent) is set
- Font system in use
- Relevant schema fields for storing feature state
- Any existing related code

## 2. {{OPTION_A}} — evidence
- {{CRITICAL_REQUIREMENT_1}}: [finding with issue numbers]
- Custom API: [code example]
- Async/timing: [finding]
- Mobile/touch: [finding]
- Multi-instance: [finding]
- TypeScript: [finding]
- Bundle size (measured): [number]
- Top 3 risks for THIS codebase

## 3. {{OPTION_B}} — evidence
(same structure)

## 4. Recommendation
- Choice: [option name]
- 3 concrete reasons tied to OUR codebase (not generic)
- 1 paragraph: why the rejected option is wrong for THIS specific use case
- Top 3 risks of the chosen option + mitigation for each

## 5. Open questions for the user
- Anything that couldn't be determined from research alone
```

## Constraints
- Do NOT install packages in the main app
- Do NOT modify any application code
- Do NOT write `RESEARCH-*.md` before asking clarifying questions and getting answers
- If you find yourself wanting to build a prototype — STOP and ask the user first
- Length budget: 300–500 lines max
- Push the branch when done

## What "done" looks like
1. Branch `feature/{{FEATURE_NAME}}` exists on remote
2. `RESEARCH-{{FEATURE_NAME}}.md` exists at repo root, committed and pushed
3. Recommendation backed by specific evidence from our code + libraries' docs/GitHub
4. The user can read it in 5 minutes and approve
```

---

## Filling in the placeholders

| Placeholder | Example value |
|---|---|
| `{{FEATURE_NAME}}` | `onboarding-tours`, `auth-system`, `excel-export` |
| `{{ONE_PARAGRAPH_PROBLEM_STATEMENT}}` | "We need to add an interactive tour system that explains buttons across 4 screens. Auto on first visit + manual replay via FAB." |
| `{{OPTION_A}}` / `{{OPTION_B}}` | `React Joyride v3.1.0` / `Driver.js v1.4.0` |
| `{{LIST_OF_AFFECTED_SCREEN_FILES_OR_COMPONENTS}}` | List the actual file paths affected. Vague generalities here will produce vague research. |
| `{{SCHEMA_PATH}}` | `src/lib/db/schema.ts` |
| `{{RELEVANT_TABLES_OR_TYPES}}` | `user_settings`, `projects` |
| `{{LAYOUT_OR_ROOT_FILE}}` | `src/app/layout.tsx` |
| `{{CRITICAL_REQUIREMENT_1}}` | RTL/Hebrew support, accessibility, offline support — depends on feature |
| `{{DESIGN_TOKENS}}` | "Safety Yellow `#EAB308` on dark surfaces with Rubik font" |
| `{{SPECIFIC_TIMING_CONCERN_FROM_CODEBASE}}` | "BoQ Draft loads from sessionStorage after mount" |
| `{{N}}` | Number of independent instances. `4 tours`, `3 modals`, etc. |
