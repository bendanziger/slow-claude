# Anti-Patterns — Workflow Failures and How to Recognize Them

> These are the patterns that have failed in real feature work. Read this before each phase to catch yourself before you do them again.

## Architecture-time anti-patterns

### "I think we should use X"
**Symptom:** Architect recommends a library based on memory or a recent blog post.

**Why it fails:** No evidence. The library might be abandoned, have a critical bug for your use case, or be wrong for your stack.

**Fix:** Delegate to the coding agent. Have it measure tarball size, search GitHub issues, read the official docs. Recommendation must cite evidence.

### "Let me show you the code"
**Symptom:** Architect dumps a 200-line snippet to "explain" the feature.

**Why it fails:** That code wasn't tested against the real schema, real components, or real types. It will need rewriting. And now there's an "official" code blob the user might paste in without verification.

**Fix:** Stay at the prompt level. If you need to illustrate a concept, 5–10 lines max, clearly labeled as illustrative.

### "Same as last time"
**Symptom:** Architect reuses a prompt verbatim from a previous feature.

**Why it fails:** Each feature has different edge cases, different affected components, different decisions. Verbatim reuse spreads the previous feature's assumptions into the new one.

**Fix:** Use the templates as scaffolding. Re-ask clarifying questions. Re-list affected components. Re-list edge cases for THIS feature.

### Skipping the clarifying questions
**Symptom:** Architect writes the research prompt without asking the user about persistence/trigger/scope.

**Why it fails:** The research prompt encodes assumptions. If those assumptions are wrong, the recommendation is wrong, even if the evidence is solid.

**Fix:** 5 questions. 30 seconds. Always.

## Research-time anti-patterns (coding agent)

### "Both libraries support RTL, so it's a tie"
**Symptom:** Vague claims without evidence.

**Why it fails:** "Supports" can mean anything from first-class to "there's an open PR that might land someday."

**Fix:** Require specific evidence per claim. GitHub issue numbers. Code samples from docs. Measured behavior in the codebase.

### Building a prototype
**Symptom:** Coding agent says "let me build a quick POC to test this."

**Why it fails:** POCs slip into production code. Cleanup never happens. And the time was budgeted for research, not implementation.

**Fix:** The coding agent must STOP and ask the architect before building anything. POC belongs in Phase 3, not Phase 1.

### Reading only the README
**Symptom:** Research cites only the library's marketing page.

**Why it fails:** READMEs claim "lightweight, fast, accessible." GitHub issues tell the truth.

**Fix:** Require issue searches with specific keywords. Read at least the top 5 open issues. Skim closed-recently for "won't fix" patterns.

## Build-time anti-patterns

### Skipping the EXPLORE phase
**Symptom:** Coding agent jumps straight to writing code.

**Why it fails:** Without reading the actual components, the code references selectors/props that don't exist. Build fails. Or worse, build passes but runtime fails.

**Fix:** The build prompt must explicitly require EXPLORE → TodoWrite → user approval before any code.

### Building the registry before the selectors exist
**Symptom:** Registry references `data-tour="review-first-item"` but ReviewScreen.tsx doesn't have that attribute yet.

**Why it fails:** Registry will silently target the wrong elements (or nothing). Tour appears broken in QA.

**Fix:** Phase B (add data attributes) MUST come before Phase C (build registry). Enforce this order in the build prompt.

### "I'll handle the edge cases later"
**Symptom:** Coding agent's Todo list doesn't include RTL, Strict Mode, demo mode, cleanup.

**Why it fails:** "Later" never comes. Edge cases get discovered in production by users.

**Fix:** Edge cases go in the Todo BEFORE code. If the agent's Todo doesn't list them, reject the Todo and ask for a revision.

### Running `drizzle-kit push`
**Symptom:** Coding agent applies migration directly to the DB.

**Why it fails:** `push` diffs schema vs DB and can drop/rename without warning. Has caused data loss in real projects.

**Fix:** Hard rule in every build prompt: `generate` only. Human applies via DB console. Use the actual word "NEVER" in the prompt.

### Reporting success without showing the build output
**Symptom:** "I've implemented the feature. Build is clean."

**Why it fails:** The build is often not clean. Type errors, ESLint errors, runtime errors. The agent claims success because it didn't actually run the build.

**Fix:** Build prompt requires `npm run build` + `npx tsc --noEmit` output pasted as evidence. Without the output, the phase is not done.

## Verify-time anti-patterns

### "It worked in demo mode"
**Symptom:** Feature uses demo mode for verification because it's faster than logging in.

**Why it fails:** Demo mode bypasses auth and often bypasses the DB. The actual write path was never exercised. The first real user hits the bug.

**Fix:** Verification MUST happen against real DB with real auth. Use the existing test account or create one.

### "Tested on desktop, looks fine"
**Symptom:** Mobile-first product, mobile not tested.

**Why it fails:** Sticky footer overlap, touch target size, viewport keyboard behavior — none visible on desktop.

**Fix:** Phase F verification requires mobile width (375px or project-specific). Add screenshots to PR.

### "I'll write the PR description later"
**Symptom:** PR has 1-line description: "Add tour system."

**Why it fails:** Reviewer (or future you) has to read the diff to figure out what changed and how to test. Slow review, missed issues.

**Fix:** Use `templates/PR-description.md`. It's a checklist. If the author can't fill in "how to test locally," they didn't test it.

## Workflow-level anti-patterns

### "Let's skip research, this one is simple"
**Symptom:** Architect agrees to skip Phase 1 to save time.

**Why it fails:** "Simple" features are where assumptions hide. Skipped research = no audit trail = no way to know why a decision was made 3 months later.

**Fix:** Even simple features get a 100-line RESEARCH.md. The format scales down. Research isn't about complexity; it's about evidence.

### Mixing phases
**Symptom:** Architect writes the build prompt while still iterating on the research prompt.

**Why it fails:** The build prompt embeds research conclusions that haven't been validated. If research changes the recommendation, the build prompt is wrong but already sent.

**Fix:** Strict phase boundaries. Phase 1 ends with `RESEARCH.md` pushed. Phase 2 ends with approval. Only then write Phase 3.

### Letting one phase silently bleed into another
**Symptom:** Coding agent finishes Phase A, then "since I'm here," moves into Phase B without confirming with the user.

**Why it fails:** No approval gate. The user wakes up to find half the feature built against wrong assumptions.

**Fix:** Every build prompt ends each phase with "STOP and wait for approval." This is non-negotiable.

---

## Recovery patterns

If you've already committed an anti-pattern, here's how to recover:

| Mistake | Recovery |
|---|---|
| Recommended a library based on vibes | Run a real research pass. Compare to alternatives. Document why you'd keep or switch. |
| Dumped code before research | Treat that code as throwaway. Run the full workflow against the feature, ignore the code. |
| Skipped clarifying questions | Ask them now. Compare answers to the assumptions in the prompt. If they mismatch — rewrite the prompt. |
| Coding agent built without TodoWrite | Stop. Have it produce the Todo retroactively. Compare to what was built. Fix gaps. |
| Used `drizzle-kit push` | Check the DB state. If destructive changes happened — restore from backup, replay migrations. |
| Verified in demo mode only | Re-verify against real DB before merging. Find the bugs now, not after deploy. |

Most anti-patterns are recoverable if caught before merge. After merge, recovery is harder. Catch them at the phase gate.
