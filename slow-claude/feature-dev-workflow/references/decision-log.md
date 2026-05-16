# Decision Log Format

> Maintain one decision log per project. Append after every feature ships. Future-you and your collaborators will thank you.

## Why keep a decision log

After 3 months of shipping a feature a day, no one remembers:
- Why we picked library X over library Y
- Why this particular table has a JSONB column instead of a normalized relation
- Why this component is intentionally untyped
- Why we skipped the obvious approach

Without a log, every revisit means re-doing the research. With a log, you read 5 lines and move on.

## Format

One entry per feature. Append to `docs/decisions.md` at repo root.

```markdown
## {{YYYY-MM-DD}} — {{FEATURE_NAME}}

**Context:** {{ONE_LINE_WHAT_PROBLEM}}

**Decision:** {{ONE_LINE_WHAT_WE_CHOSE}}

**Alternatives considered:**
- {{ALTERNATIVE_1}} — rejected because {{REASON}}
- {{ALTERNATIVE_2}} — rejected because {{REASON}}

**Rationale (3 points):**
1. {{POINT_1}}
2. {{POINT_2}}
3. {{POINT_3}}

**Trade-offs accepted:**
- {{TRADE_OFF_1}}
- {{TRADE_OFF_2}}

**Reversibility:** {{EASY | MEDIUM | HARD}}. {{ONE_LINE_HOW_TO_REVERSE}}

**Research artifact:** [`RESEARCH-{{FEATURE_NAME}}.md`]({{URL_OR_PATH}})

**PR:** {{PR_URL}}
```

## Example entry

```markdown
## 2026-05-16 — onboarding-tours

**Context:** 50 beta contractors land with no idea what the buttons do. Need an interactive tour system across 4 screens.

**Decision:** React Joyride v3.1.0 with custom tooltip component. Persistence in `user_settings.tour_completions` JSONB.

**Alternatives considered:**
- Driver.js v1.4.0 — rejected because open RTL PR (#569), open mobile issues (#462, #524, #442), manual async-target handling
- DIY tour system — rejected because positioning + scroll-into-view + viewport collision is a bug magnet, not core to product value

**Rationale (3 points):**
1. React/Next 16 fit: Joyride v3 has `useJoyride` hook, async step hooks, React 19 support. Driver.js requires manual cleanup wrappers.
2. Async target safety: BoQ Draft loads from sessionStorage, Quote Sandbox renders client-side state. Joyride waits with `targetWaitTimeout`. Driver falls back to a center dummy element.
3. Hebrew RTL: Joyride lets us render a fully custom React tooltip with `dir="rtl"`. Driver requires CSS overrides + an unmerged PR.

**Trade-offs accepted:**
- Bundle: Joyride ~20kB gzipped vs Driver ~8kB. Mitigated by client-only dynamic import on tour screens.
- Persistence write requires a schema migration.

**Reversibility:** MEDIUM. Library can be swapped in 1–2 days. Persistence column stays (already in DB).

**Research artifact:** [`RESEARCH-onboarding-tours.md`](https://github.com/bendanziger/onsite-ai/blob/main/RESEARCH-onboarding-tours.md)

**PR:** https://github.com/bendanziger/onsite-ai/pull/123
```

## When to log

Log immediately after PR merge. Not before — the decision isn't real until it's shipped. Not later — you'll forget the nuance.

5 minutes to write. Saves hours later.

## When to revisit a decision

A decision log entry is not permanent. Revisit when:

- The library is abandoned (last commit > 1 year, security issues unpatched)
- A trade-off you accepted is now blocking other work
- A new alternative emerges that solves your original concerns

When revisiting, add a new entry rather than editing the old one:

```markdown
## 2026-08-20 — onboarding-tours (REVISITED)

**Context:** Bundle cost from Joyride became 15% of mobile bundle. Beta partners on 3G are slow to first paint.

**Decision:** Switch to lazy-loaded chunked tours per route.

**Supersedes:** 2026-05-16 onboarding-tours entry.

[... rest of format ...]
```

This preserves the audit trail.
