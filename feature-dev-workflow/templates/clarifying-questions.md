# Clarifying Questions — Pre-Phase 1

> These are the questions to ask the user BEFORE writing the research prompt. Do not skip them, even if you think you know the answer. The user might have changed their mind from the last feature.

## The 5 core questions

Ask these as multiple-choice when possible (use `ask_user_input_v0` if available). The user can override any answer with free text.

### 1. Persistence
**Where does feature state live?**

Options to present:
- Database (e.g., `user_settings` JSONB) — per user, cross-device, requires migration
- `localStorage` — per device, no migration, lost on cache clear
- Both (DB for primary, localStorage for caching) — most resilient, most complex
- Stateless — no persistence needed

The "right" answer depends on:
- Is the user authenticated when interacting with this feature?
- Should state survive a device switch?
- Is a migration acceptable now?

### 2. Trigger model
**When does the feature appear / activate?**

Options:
- Auto on first visit — invasive but discoverable
- Manual only (button/menu) — opt-in but easy to miss
- Auto first time + manual replay — balances discovery and control (most common)
- Context-dependent (e.g., only after extraction completes)

### 3. Scope of "first time"
**What counts as "first time"?**

Options:
- Per user, ever (sees once, never again)
- Per user per resource (sees once per project, per quote, etc.)
- Per user per version (re-show after the feature is updated)
- Per device (rare; usually a sign the wrong persistence was chosen)

### 4. Empty state behavior
**What happens when the feature's dependencies aren't met (no data, no auth, no target)?**

Options:
- Silent skip — no log, no error, just doesn't appear
- Console debug log — for developer visibility, no user impact
- Error toast — user sees something went wrong
- Fallback content — show a different version

For most features, **silent skip + debug log** is correct. Errors should be reserved for actual failures, not "the conditions aren't right yet."

### 5. Mobile vs desktop priority
**Where will most users encounter this feature?**

Options:
- Mobile-first (contractor in the field, customer on phone) — design for 375px, test mobile first
- Desktop-first (admin panel, internal tool) — full-width design, test desktop first
- Both equally — design responsively, test both at every phase

## Project-specific questions (add as needed)

Some features need extra questions based on the project. Examples:

**For RTL/Hebrew projects:**
- Should the feature respect mixed LTR content (e.g., numbers, English brand names)?

**For multi-tenant projects:**
- Is this per-tenant or global?

**For audit-sensitive projects:**
- Does this need event logging for compliance?

**For projects with role-based access:**
- Which roles can see/use this feature?

## When the user is impatient

The user may push back: "just write the prompt, I'll figure it out."

This is a mistake. Hold the line politely:

> "Five questions, 30 seconds. Without them I'll have to guess, and a guess in research becomes rework in build. Faster to ask now."

If they still refuse: proceed with explicit assumptions stated in the research prompt. The coding agent will then ask the user during its own EXPLORE phase — same questions, just delayed.
