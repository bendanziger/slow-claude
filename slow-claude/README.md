# slow-claude

> Slowing Claude down is the feature, not the bug.

A Claude.ai skill + Claude Code slash commands that enforce a disciplined feature development workflow. Stops Claude from jumping straight into code, demands evidence before recommendations, and uses Spec Driven Development with explicit approval gates between phases.

Built for solo founders shipping a feature a day under competitive pressure.

---

## The problem

You're moving fast. A new feature comes up. You open Claude and say "I want to add X."

What happens by default:
- Claude dumps code based on memory or vibes
- Recommends a library it half-remembers from a blog
- Skips the questions that would have changed the answer
- Lets the coding agent (Claude Code, Cursor, etc.) start implementing before you've decided the right approach

Three days later you realize the library is wrong, the persistence model is wrong, or the edge cases weren't considered. You rewrite. At one-feature-a-day cadence, this rework is what actually slows you down.

## The fix

**slow-claude** turns Claude into a structured architect. Not faster — more deliberate. The workflow is 4 phases, each with an approval gate:

1. **Research** — Claude Code creates a `feature/<name>` branch, reads your codebase, compares options with **citations** (GitHub issue numbers, measured bundle sizes), and writes `RESEARCH-<feature>.md`. ~10 minutes.
2. **Decision** — You read the research, approve a choice, answer open questions.
3. **SDD Build** — Zod schemas first. Then phases A→F with build-clean gates between them. Migrations generated but never auto-applied.
4. **Verify** — Manual test against real DB and real auth (demo mode bypasses persistence). Open PR.

The skill lives in Claude.ai. The slash commands live in Claude Code. Templates are shared between them — single source of truth.

---

## What's in this repo

```
slow-claude/
├── README.md                     ← you are here
├── feature-dev-workflow/         ← the Claude.ai skill (unpacked)
│   ├── SKILL.md
│   ├── templates/                ← prompt templates for each phase
│   └── references/               ← edge cases, anti-patterns, decision log format
├── claude-code-commands/         ← slash commands for Claude Code
│   ├── research.md
│   └── build.md
└── feature-dev-workflow.skill    ← packaged skill (one-click install)
```

---

## Install

### Step 1 — Install the skill in Claude.ai

1. Download `feature-dev-workflow.skill` from this repo (Releases page or root).
2. Open [claude.ai](https://claude.ai) → click your profile → **Settings**
3. **Capabilities** → **Skills** → **Upload skill**
4. Select the downloaded file.

That's it. The skill is active. There's nothing to configure.

### Step 2 — Add slash commands to your project (Claude Code)

In any project where you use Claude Code:

```bash
cd <your-project>
mkdir -p .claude/commands
curl -o .claude/commands/research.md https://raw.githubusercontent.com/<your-username>/slow-claude/main/claude-code-commands/research.md
curl -o .claude/commands/build.md https://raw.githubusercontent.com/<your-username>/slow-claude/main/claude-code-commands/build.md
git add .claude/commands/
git commit -m "chore: add slow-claude SDD slash commands"
```

Replace `<your-username>` with your GitHub handle after forking.

---

## Use

### Starting a new feature

1. Open Claude.ai. Say something like:
   > "I want to add an onboarding tour system across 4 screens."

   The skill triggers automatically. Claude will ask you 5 clarifying questions: persistence model, trigger model, scope, empty state behavior, mobile priority.

2. Claude hands you a research prompt. Paste it into Claude Code:
   ```
   /research onboarding-tours
   ```

3. Claude Code spends ~10 minutes reading your codebase, comparing options with real evidence, and writing `RESEARCH-onboarding-tours.md` on a dedicated branch.

4. Bring the research back to Claude.ai. Discuss, decide, lock in.

5. Claude hands you a build prompt. Paste into Claude Code:
   ```
   /build onboarding-tours
   ```

6. Claude Code builds the feature phase by phase, stopping after each one for your approval. Build output is shown before each phase ends.

7. You apply the migration manually via SQL console. Test on mobile. Merge.

### What you skip

You don't write the prompts. You don't list the edge cases. You don't argue with Claude about whether to use library X or Y based on vibes. The skill enforces all of that.

---

## What slow-claude does NOT do

- **Not a code generator.** It doesn't write your features. It coordinates between you and your coding agent.
- **Not project-specific.** The skill is stack-agnostic. Project-specific defaults (touch target sizes, color tokens, "don't modify these files") live in your project's `CLAUDE.md`, not here.
- **Not a replacement for thinking.** The 5 clarifying questions force you to make decisions you'd otherwise defer. The workflow surfaces tradeoffs, but you still pick.

---

## Why "slow"?

Slowing down at the start saves time at the end. Three days of fast typing into Claude with no audit trail produces three days of rework when you discover the library was wrong or the schema doesn't fit. A 10-minute research phase prevents that.

The name is a reminder: when you're tempted to skip a phase because "this one is simple" — that's exactly when the workflow earns its keep.

---

## Anti-patterns this prevents

| Without slow-claude | With slow-claude |
|---|---|
| "I think we should use library X" | RESEARCH.md cites GitHub issues + measured bundle |
| Claude dumps 200 lines of untested code | Claude writes a prompt; Claude Code writes the code against real schema |
| "Same as last feature, copy the prompt" | Re-asks clarifying questions; this feature has different edge cases |
| Coding agent skips EXPLORE, code references selectors that don't exist | TodoWrite list approved before any code |
| `drizzle-kit push` silently drops a column | Migrations generated only; human applies via console |
| "It worked in demo mode" | Verify against real DB + real auth before merge |

---

## Status

v1.0. Untested at scale. Built from a real workflow that shipped multiple production features. Expect iteration — file issues, send PRs.

## License

MIT — fork it, modify it, share it.

## Credits

Workflow developed by [@your-name] while building [Peles](https://peles-app.co.il), an Israeli construction bills-of-quantities SaaS. Refined across multiple production features under competitive deadline pressure.
