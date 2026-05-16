# PR Description Template — Phase 4

> Use this format for every feature PR. Inconsistent PR descriptions make review slower; this template makes review fast.

---

```markdown
## {{FEATURE_NAME}}

{{ONE_LINE_SUMMARY_OF_WHAT_USER_GETS}}

### What changed

**Schema:**
- `{{TABLE}}.{{COLUMN}}`: new `{{TYPE}}` column. Default `{{DEFAULT}}`. Not null.

**Server actions:**
- `{{actionName}}`: {{ONE_LINE_DESCRIPTION}}

**New components:**
- `src/components/{{FEATURE}}/{{FILE}}.tsx`: {{PURPOSE}}

**Modified components:**
- `{{PATH}}`: added `data-{{FEATURE}}="..."` attributes (no logic changes)

**New libraries:**
- `{{PACKAGE_NAME}}@{{VERSION}}` ({{SIZE_GZIPPED}} kB gzipped)

### How to test locally

1. Apply the migration:
   ```sql
   {{INLINE_THE_MIGRATION_SQL}}
   ```
   Run via Supabase SQL Editor (project `xlnlggwoosckeubmkkrb`).

2. Verify the column exists:
   - Database → Tables → `{{TABLE}}` → confirm `{{COLUMN}}` exists with type `{{TYPE}}`.

3. Start dev server:
   ```bash
   npm run dev
   ```

4. Walk through:
   - {{STEP_1}}
   - {{STEP_2}}
   - {{STEP_3}}

5. Reset for re-testing (optional):
   ```sql
   {{RESET_SQL}}
   ```

### Mobile verification

Tested at {{MOBILE_WIDTH}}px viewport (iPhone 12 emulation):
- [ ] {{CHECK_1}}
- [ ] {{CHECK_2}}
- [ ] {{CHECK_3}}

### Known limitations / follow-ups

- {{LIMITATION_1}} — tracked for follow-up PR
- {{LIMITATION_2}}

### Decision log reference

See `RESEARCH-{{FEATURE_NAME}}.md` for the library/approach choice and rationale.
```

---

## Why every field matters

| Field | Why it's in the template |
|---|---|
| Schema section | Reviewers need to know if there's a migration before they pull and run |
| Inline migration SQL | Reviewer can copy-paste into DB console without hunting for the file |
| "How to test locally" steps | Without this, reviewers approve based on diff reading — that misses runtime bugs |
| Reset SQL | Lets the reviewer test "first-run" state more than once without creating new accounts |
| Mobile verification checklist | Forces the author to actually test mobile, not just claim they did |
| Known limitations | Prevents follow-up "you forgot X" comments — surface it yourself |
| Decision log reference | Future-you will thank present-you when wondering "why did we pick this library?" |
