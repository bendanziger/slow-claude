# Build Phase (SDD) — for Claude Code

Branch `feature/$ARGUMENTS` already exists with `RESEARCH-$ARGUMENTS.md` approved by the user. Build the feature E2E in this branch.

## SDD — contracts first
1. **Zod schemas** in `src/lib/validations/$ARGUMENTS.ts` (input/output, parse on every server boundary)
2. **Registry/types** in `src/lib/$ARGUMENTS/registry.ts`
3. **Migration** via `drizzle-kit generate` — NEVER `push`. Commit SQL, user applies via Supabase SQL Editor.
4. **Server actions** in `src/app/actions.ts` wrapped in `requireAuth()`, input validated with `.parse()`, `revalidatePath()` after writes
5. **Client components** separated from server logic. Demo mode no-op for writes.

## EXPLORE phase (mandatory before any code)
1. Read affected components in full (cite file paths back to user)
2. TodoWrite: every file you'll touch, categorized (schema / action / registry / provider / UI / target attributes)
3. List edge cases: RTL, React Strict Mode double-mount, DEMO_MODE bypass, cache invalidation, async target timing, sticky footer overlap, empty states
4. **STOP. Show user the Todo list. Wait for approval.**

## Implementation phases (in this order)
- **Phase A:** Schema + Server Action. Build gate.
- **Phase B:** Add stable `data-*` attributes to real components BEFORE registry.
- **Phase C:** Registry/contract built against real selectors. Custom RTL UI if needed. Build gate.
- **Phase D:** Provider/Controller component. Lifecycle, cleanup, pathname-aware. Build gate.
- **Phase E:** UI surface (FAB / Modal / Tab / etc). 48px touch targets. Build gate.
- **Phase F:** Polish + manual verification at 375px mobile + desktop.

**After each phase:** `npm run build` + `npx tsc --noEmit`. Show clean output. Do not proceed if errors.

## Hard rules
- Touch targets ≥ 48px (contractors wear gloves)
- Hebrew RTL, Rubik font, Safety Yellow `#EAB308` only for primary CTAs
- DEMO_MODE writes are no-ops, not crashes — gate behind `isDemoModeEnabled()`
- Never modify `QuoteView.tsx`
- Never touch `src/app/(auth)/onboarding/*` or `WelcomeScreens.tsx` / `TradesPicker.tsx` / `FirstRunDetailsOverlay.tsx` unless explicitly assigned

## Done definition
- Branch pushed to remote
- Migration SQL committed (user applies, not you)
- 375px mobile manually verified
- PR opened to `main` with description including:
  - What changed (component-level summary)
  - How user tests it locally
  - SQL migration apply instructions inline
  - Known limitations / follow-ups
