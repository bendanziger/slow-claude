# Edge Cases Checklist

> Read this when writing the Phase 3 (Build) prompt. Add the relevant items to the coding agent's TodoWrite list as explicit checks.

## Universal edge cases (apply to nearly every feature)

### React-specific
- **React Strict Mode double-mount.** Dev mode runs effects twice. Any `useEffect` that fires a server action, starts a tour, or registers a listener must be idempotent. Use `useRef` guards or `AbortController`.
- **Hydration mismatch.** Server-rendered HTML and client-rendered HTML must match exactly. Watch for `Date.now()`, `Math.random()`, browser-only APIs called during render.
- **Stale closures.** Long-lived callbacks in `useEffect` capture values at registration time. Use `useRef` for values that change, or include them in the dependency array.

### Next.js App Router
- **Server vs client components.** Server Components can't use hooks, state, or browser APIs. Client Components can't be async or read server-only env vars.
- **`revalidatePath` after mutations.** Without it, the cached Server Component data is stale and the UI shows old state.
- **`force-dynamic` for pages that depend on cookies/auth.** Otherwise Next caches the static version and serves the wrong data.
- **Middleware vs route handler auth.** If your project has both, be consistent about where auth is enforced.

### State synchronization
- **Cache invalidation across providers.** If feature state lives in `user_settings` and also in a React Context, both must be updated atomically after a write.
- **Optimistic updates.** UI updates before server confirms. Failure path must roll back. Without rollback, the UI shows a state that doesn't exist in the DB.

### Async timing
- **Target element not yet in DOM.** Tours, focus management, scroll-into-view, animations — all break if the target hasn't mounted. Use `MutationObserver`, polling with timeout, or `useEffect` chained to the data load.
- **`sessionStorage`/`localStorage` handoff.** Reading on first render returns null. Use `useEffect` for reads, not render-time reads.
- **Race conditions on rapid input.** Search-as-you-type, button mashing. Use `AbortController` or request-id matching to ignore stale responses.

## Internationalization / RTL

For projects in Hebrew, Arabic, Farsi, Urdu:

- **`dir="rtl"` on `<html>` not always enough.** Some libraries render outside the document tree (portals, overlays) and don't inherit direction. Test every overlay.
- **Mixed content in RTL.** Numbers, English brand names, code snippets render LTR within RTL paragraphs. Use `<bdi>` or `unicode-bidi: isolate`.
- **Logical CSS properties.** `margin-inline-start` instead of `margin-left`. `inset-inline-end` instead of `right`. Otherwise layouts break on direction flip.
- **Icons that point.** Arrow icons must mirror in RTL. `transform: scaleX(-1)` or use direction-aware icon variants.
- **Built-in library labels.** Many libraries hardcode "Next", "Back". Use the library's `locale` prop or wrap with custom components.
- **Date/time pickers.** Often LTR-only by default. Test with Hebrew month names.

## Demo / preview / staging modes

If the project has a `DEMO_MODE` or similar bypass:

- **Auth bypass must not break read paths.** If the feature reads `user_settings`, demo mode must return a sensible default, not throw.
- **Auth bypass must not break write paths.** Server actions in demo mode should `return early` as no-op, not crash on missing auth context.
- **Demo mode != production validation.** A feature that works in demo mode but writes to DB has not been verified. Always test on a real-DB dev server before declaring done.

## Database / migration

- **`drizzle-kit push` is dangerous.** It diffs your schema against the DB and applies destructive changes (drops, renames) without warning. Use `drizzle-kit generate` and apply SQL manually.
- **NOT NULL columns need defaults.** Adding a NOT NULL column to a table with existing rows fails unless you provide `DEFAULT '<value>'` or the column is nullable.
- **JSONB defaults need explicit cast.** `DEFAULT '{}'` may fail. Use `DEFAULT '{}'::jsonb`.
- **RLS policies don't auto-apply.** Adding a new column doesn't update existing policies. New columns may need new policies if they contain sensitive data.

## Mobile / touch

- **48px minimum touch target.** iOS HIG says 44px, Material says 48px. For gloved hands (construction, food service, healthcare), 48px is the minimum.
- **Sticky / fixed elements stack.** Multiple sticky bars at the bottom = footer covers content. Use spacer divs or `padding-bottom: env(safe-area-inset-bottom)`.
- **iOS Safari quirks:**
  - 100vh includes the browser chrome. Use `100dvh` or JS-measured height.
  - `position: fixed` with keyboard open jumps. Use `visualViewport` API.
  - `touch-action: manipulation` to disable double-tap zoom on interactive elements.
- **Pull-to-refresh** can fire on accident if your top scrollable element starts at y=0. Use `overscroll-behavior: contain`.

## Forms / validation

- **Client validation is UX, server validation is security.** Both must run. Same schema (Zod) on both sides.
- **`onSubmit` only fires on Enter or click.** Mobile users may use "Done" on the keyboard — make sure that also submits.
- **Disabled submit with invalid form.** Better UX is to show errors on blur or after first submit attempt. Disabled buttons confuse users.

## Empty states

- **Zero data is not an error.** First-time user, fresh project, deleted everything — all valid states. Plan UI for them.
- **Loading vs empty.** "No items" displayed before the fetch returns looks like a bug. Show skeleton or spinner until you know the data is empty.
- **Empty + filter applied.** Different from no data ever — message should be "no matches" not "you have nothing."

## Cleanup / unmount

- **Event listeners.** Every `addEventListener` needs a `removeEventListener` in cleanup.
- **Timers.** Every `setTimeout` / `setInterval` needs `clearTimeout` / `clearInterval` in cleanup.
- **Subscriptions / observables.** Unsubscribe in cleanup.
- **Pathname change.** If your component depends on pathname, cleanup must run when pathname changes (not just unmount). React Router / Next router give hooks for this.

## Performance

- **Bundle bloat from one feature.** Check bundle size impact with `npm run build` before and after. A 50kB tour library is fine; a 500kB chart library on a landing page is not.
- **Re-renders on every keystroke.** Forms with non-memoized derived state cause cascading re-renders. Use `useMemo` for expensive computations.
- **Layout thrashing.** Reading layout (`offsetWidth`) and then writing (`style.width`) in a loop causes forced reflows. Batch reads, then writes.

---

## How to use this checklist

When writing the Phase 3 build prompt, scan this list and add to the EXPLORE phase:

```
3. List edge cases up front and add them to the Todo:
   - Universal: React Strict Mode, async target timing, cleanup on pathname change
   - Project-specific RTL: portals, mixed content, logical CSS properties
   - Demo mode: writes no-op, reads return defaults
   - Migration: drizzle generate not push, NOT NULL needs default
   - Mobile: 48px touch targets, sticky footer overlap at 375px
   - Empty state: silent skip with debug log
```

Don't dump the whole file — pick what's relevant to this feature and inline it.
