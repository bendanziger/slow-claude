# Build Prompt Template — Phase 3 (SDD)

> Use ONLY after Phase 1 (Research) and Phase 2 (Decision) are complete. The `RESEARCH-<feature>.md` file must exist and be approved.

---

```markdown
# {{FEATURE_NAME}} — Build Phase (SDD)

## Your role
You are now implementing. Branch `feature/{{FEATURE_NAME}}` already exists with `RESEARCH-{{FEATURE_NAME}}.md` approved. Build the full feature E2E in this same branch. Open a single PR back to `main` when done.

## Decision recap (already approved by the user)
- **Choice:** {{LIBRARY_OR_APPROACH}}
- **Persistence:** {{PERSISTENCE_SHAPE}}
- **Reset:** {{RESET_MECHANISM}}
- **Trigger:** {{TRIGGER_MODEL}}
- **Scope:** {{SCOPE}}
- **Edge case handling:** {{KEY_EDGE_CASE_DECISIONS}}

## Spec Driven Development — contracts first

### 1. Zod schemas

Create `src/lib/validations/{{FEATURE_NAME}}.ts`:

```typescript
import { z } from 'zod';

// {{INPUT_SCHEMA_DESCRIPTION}}
export const {{featureName}}InputSchema = z.object({
  {{FIELD_1}}: z.{{TYPE_1}}(),
  {{FIELD_2}}: z.{{TYPE_2}}(),
});

// {{OUTPUT_SCHEMA_DESCRIPTION}}
export const {{featureName}}StateSchema = z.object({
  {{STATE_FIELDS}}
});

export type {{FeatureName}}Input = z.infer<typeof {{featureName}}InputSchema>;
export type {{FeatureName}}State = z.infer<typeof {{featureName}}StateSchema>;
```

These are CONTRACTS. Every read/write of feature state passes through them.

### 2. Registry / config contract (if feature has configuration data)

Create `src/lib/{{FEATURE_NAME}}/registry.ts`. Use stable selectors that already exist when possible. For anything new, ADD stable `data-*` attributes to the real components.

## EXPLORE phase (mandatory before any code)

1. Read in full the components identified in `RESEARCH-{{FEATURE_NAME}}.md`:
   {{LIST_OF_COMPONENTS_FROM_RESEARCH}}

2. Use TodoWrite to list every file you will touch. Categorize: schema, server action, registry, provider, UI surface, target attributes per screen.

3. List edge cases up front and add them to the Todo:
   - {{EDGE_CASE_1}} (e.g., sessionStorage handoff timing)
   - {{EDGE_CASE_2}} (e.g., React Strict Mode double-mount)
   - {{EDGE_CASE_3}} (e.g., pathname change cleanup)
   - {{EDGE_CASE_4}} (e.g., user with demo mode bypassing DB)
   - {{EDGE_CASE_5}} (e.g., cache invalidation after writes)
   - Empty states (no data, no user) — silent, not error
   - Mobile viewport overlap with sticky elements

4. STOP and show the user the Todo list before writing code. Wait for approval.

## Implementation phases

### Phase A — Schema + Server Action
1. Migration via `drizzle-kit generate` (or project equivalent). Commit the SQL file. Do NOT run `push`. The user applies the migration manually.
2. Update `{{SCHEMA_PATH}}` with the new column/table typed via `$type<{{FeatureName}}State>()`.
3. Server action `{{actionName}}` in `src/app/actions.ts`:
   - Wraps in `requireAuth()` (or project equivalent)
   - Validates input with `{{featureName}}InputSchema.parse()`
   - {{SPECIFIC_BUSINESS_LOGIC}}
   - Calls `revalidatePath()` on affected routes
   - Demo mode: no-op early return

**Build gate:** `npm run build` + `npx tsc --noEmit` clean. Show output. Stop and wait.

### Phase B — Add stable `data-*` attributes to real components
**Important: do this BEFORE the registry.** The registry must be built against real selectors that exist in the actual components.

For each affected screen, edit the actual component files and add `data-{{FEATURE_PREFIX}}="..."` attributes. Do NOT change existing logic. Only add the attributes.

{{LIST_OF_COMPONENT_ATTRIBUTE_PAIRS}}

**Build gate:** clean build. Stop and wait.

### Phase C — Registry / contract built against real selectors
1. Create `src/lib/validations/{{FEATURE_NAME}}.ts` (the Zod file above).
2. Create `src/lib/{{FEATURE_NAME}}/registry.ts` with the full feature configuration. Use the selectors you just added in Phase B.
3. Create custom UI component(s) if needed:
   - Direction: {{DIRECTION}}
   - Font: {{FONT}}
   - Color tokens: {{TOKENS}}
   - Minimum touch target: {{TOUCH_TARGET_PX}}px
   - Accessibility: aria-labels in {{LANGUAGE}}

**Build gate:** clean build. Stop and wait.

### Phase D — Provider / Controller
1. Create `src/components/{{FEATURE_NAME}}/{{FeatureName}}Provider.tsx`:
   - Client component
   - Reads state from initial server-fetched user data (passed as prop)
   - Uses pathname/route to scope behavior
   - Handles lifecycle: mount, cleanup on pathname change, React Strict Mode safety
   - Listens for completion events → calls server action
   - Handles error states silently (log, don't throw, don't mark completed)
2. Mount the Provider once in the appropriate layout. It must wrap all affected screens.

**Build gate:** clean build. Stop and wait.

### Phase E — UI surface
1. Create `src/components/{{FEATURE_NAME}}/{{FeatureName}}UI.tsx`:
   - Only renders on relevant routes
   - Positioning: clears existing sticky elements ({{STICKY_OFFSET_NOTES}})
   - Style: secondary, does not compete with primary CTAs
   - Touch target ≥ {{TOUCH_TARGET_PX}}px
   - `aria-label` in {{LANGUAGE}}
2. Verify on 375px viewport (or project-specific mobile width).

**Build gate:** clean build. Stop and wait.

### Phase F — Polish + verification
1. Manual test on {{MOBILE_WIDTH}}px mobile viewport AND desktop:
   - Sign up a fresh user (or have the user reset state via SQL)
   - Walk through the feature E2E
   - Verify direction/language correctness
   - Skip/cancel → verify state record written
   - Reload → verify auto behavior is correct (e.g., does not re-show if completed)
   - Manual trigger (FAB/button) → feature re-runs
   - Repeat for each affected screen
2. Verify cleanup: navigate away mid-feature → no console errors, no leftover overlay
3. Verify React Strict Mode: should not double-fire events
4. Run final `npm run build` + `npx tsc --noEmit` → both clean

## Constraints (hard rules)

- {{LANGUAGE}} copy quality matters — this is user-facing. Test by reading aloud. Does it sound like a friend explaining, or like marketing?
- Touch targets ≥ {{TOUCH_TARGET_PX}}px on every interactive element.
- Never target {{FORBIDDEN_SELECTORS}}.
- Never modify {{FORBIDDEN_FILES}}.
- DO NOT use `drizzle-kit push` (or project equivalent auto-apply). Generate SQL, commit, hand to the user.
- DO NOT touch {{OUT_OF_SCOPE_DIRECTORIES}}.
- Demo mode must not crash — gate server writes behind a demo flag check.

## After each phase

Run `npm run build` + `npx tsc --noEmit`. Do not report success without showing clean build output.

## What "done" looks like

1. Branch `feature/{{FEATURE_NAME}}` has all functionality working E2E
2. Migration SQL file committed (not yet applied — user does that)
3. All affected screens have `data-*` attributes wired to the registry
4. {{LANGUAGE}} copy reviewed for clarity and tone
5. Mobile ({{MOBILE_WIDTH}}px) verified manually
6. PR opened to `main` with description following the format in `templates/PR-description.md`
7. Build + type-check clean
```

---

## Filling in the placeholders

The placeholders inherit from your Phase 1 research and Phase 2 decisions. Common values:

| Placeholder | Where it comes from |
|---|---|
| `{{FEATURE_NAME}}` | Same as Phase 1 |
| `{{LIBRARY_OR_APPROACH}}` | Phase 2 decision |
| `{{PERSISTENCE_SHAPE}}` | Phase 2 decision — JSONB shape, table name, etc. |
| `{{TRIGGER_MODEL}}` | Phase 2 decision |
| `{{LIST_OF_COMPONENTS_FROM_RESEARCH}}` | Copy from RESEARCH-*.md section 1 |
| `{{EDGE_CASE_1..N}}` | From RESEARCH-*.md risks + your project's known patterns (see `references/edge-cases.md`) |
| `{{LIST_OF_COMPONENT_ATTRIBUTE_PAIRS}}` | Spell out exactly: "ReviewScreen.tsx: `data-tour="review-first-item"` on first item card" |
| `{{DIRECTION}}` | `rtl` for Hebrew/Arabic, `ltr` otherwise |
| `{{FONT}}` | Project font (Rubik, Inter, etc.) |
| `{{TOKENS}}` | Design system color names |
| `{{TOUCH_TARGET_PX}}` | Project minimum (48px is standard for gloved hands; 44px is iOS default) |
| `{{LANGUAGE}}` | Hebrew, English, Arabic, etc. |
| `{{STICKY_OFFSET_NOTES}}` | "FAB sits at `bottom-24` on screens with sticky footer, `bottom-4` on Home" |
| `{{MOBILE_WIDTH}}` | 375 (iPhone), 360 (Android median), or project-specific |
| `{{FORBIDDEN_SELECTORS}}` | Hidden print elements, internal test IDs, etc. |
| `{{FORBIDDEN_FILES}}` | Files that must never be modified (e.g., print renderer) |
| `{{OUT_OF_SCOPE_DIRECTORIES}}` | Other developers' scope, archived code, etc. |
