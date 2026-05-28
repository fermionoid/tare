---
name: tare
description: Use this skill when the user explicitly asks to audit, prune, clean, or "tare" their accumulated agent scaffolds — phrases like "run tare", "audit my skills", "tare my scaffolds", "clean up my custom instructions", "my prompts feel bloated". Also appropriate to surface after a major model version has shipped, since past scaffolding tends to become overfit to past model behavior. Walks through the user's scaffolds (skill files, custom instructions, system prompts, slash commands, project rules), judges each one's value vs cost given current model capabilities, proposes KEEP / MODIFY / REMOVE for the user to confirm, then executes the change where the agent has file-system tools — otherwise outputs the exact change for the user to apply manually. Do NOT use for general coding tasks, single-shot prompts, debugging individual outputs, or anything other than explicit scaffold auditing.
---

# tare

You are running a scaffold audit. The user has accumulated scaffolds — skill files, custom instructions, system prompts, slash commands, project-level rules — across one or more agent platforms (Claude Code, ChatGPT, Codex, Cursor, etc.). Some of these were crutches for past model limitations and the current model doesn't need them anymore. Some overlap. Some are scoped wrong. tare's job is to find the bloat and remove or edit it.

## Step 1: Determine scope (don't ask if you can figure it out)

**If you have file-system tools, just go.** Default to the standard scaffold locations for whatever platform you're running on:

- Claude Code → `~/.claude/skills/*` and `~/.claude/commands/*` (skip `tare` itself)
- Codex CLI → the user's Codex skills/instructions directory
- Cursor / Windsurf → `.cursor/rules/*` or `.cursorrules` in the current project root
- Anything else with file access → check that platform's obvious config location

State what you're about to audit in one sentence ("I'll audit the N skills under `~/.claude/skills/`") and start. **Do not block on confirmation.** The user invoked you — they've already opted in.

**Only ask first if:**

- You're in a chat-only environment with no file tools (e.g., ChatGPT web) — the user has to paste what they want audited
- The user explicitly scoped it ("tare just my project rules", "audit only my custom GPT instructions") — follow that scope
- Multiple plausible default locations exist and you genuinely can't tell which they meant — but assume the broadest reasonable one before asking

## Step 2: For each scaffold, judge three things

- **Is it still solving a problem the current model has?** Assume the user is on the latest model their platform offers — that's the default case. If the capability has been internalized by that model, the scaffold is dead weight. *Exception:* if the user has mentioned they also use older or cheaper models (e.g., "I sometimes drop to a smaller model for cost"), bias toward keeping scaffolds the weaker model still needs.
- **Does it overlap with another installed scaffold?** Duplicates accumulate when users layer different sources of advice (multiple skill packs, copy-pasted prompts).
- **Is its trigger or wording correctly scoped?** Too broad → fires when not wanted. Too narrow → never fires. Stale wording → may misfire on current model behavior.

Read the full body of the scaffold for context, not just its description or title.

## Step 3: Recommend

One of three labels:

- **KEEP** — still net positive
- **MODIFY** — has value but needs editing. Include the concrete proposed change (specific lines, before/after, replacement text)
- **REMOVE** — no longer earns its cost given current model capabilities

## Step 4: Present each scaffold clearly

The user must be able to decide **yes / no / skip from what you show them alone** — don't make them open files or remember what they wrote months ago. Use this exact format per scaffold:

---

### Scaffold N of M: `<path>` (X lines)

**What it does:** one plain-language sentence.

**Recommendation: KEEP / MODIFY / REMOVE**

**For REMOVE** — quote the whole file (or the contiguous section being removed) in a code block, then 2-3 plain-language sentences on why it's no longer earning its keep. If the scaffold contradicts how the model currently behaves by default, quote that conflicting default behavior too, so the contradiction is concrete.

**For MODIFY** — quote the *exact text being removed* in one code block. Quote the *exact text being added* in another (if any). Then 2-3 sentences on why. End with: *"Result: file goes from X lines to Y lines."*

**For KEEP** — one sentence on what unique value it adds that the current model wouldn't do by default. No quoting needed.

**Your call:** type **yes** to apply, **no** to skip, or tell me a different edit.

---

Wait for the user's answer before moving to the next scaffold. Accept short answers ("yes", "ok", "no", "skip", "modify this way: …").

## Plain language rule

Write your reasoning as if to a technically literate person who *doesn't read engineering blogs*. Concrete substitutions:

| Don't write | Write instead |
|---|---|
| "scaffolding-vs-model conflict" | "this rule contradicts what Claude already does by default" |
| "verbatim duplicate" | "this is literally word-for-word the same as Claude's built-in behavior" |
| "first-principles" | "reasoning from fundamentals" — or describe what it actually means |
| "subagent" / "delegated subtask" | "spinning off a helper task" — describe the concrete action |
| "TDD" | "test-driven development" — or just describe: "write the test first, then iterate code until it passes" |
| "harness" (in user-facing text) | "the instructions you've stacked on top of the model" |
| "overfit" (in user-facing text) | "tuned to past behavior that no longer applies" |

If your sentence sounds like a Hacker News comment, rewrite it.

## Example of a good MODIFY presentation

> ### Scaffold 3 of 5: `~/.claude/skills/my-test-helper/SKILL.md` (45 lines)
>
> **What it does:** auto-triggers when you mention writing tests, and reminds Claude to follow test-driven development.
>
> **Recommendation: MODIFY**
>
> Remove this block (lines 12–28):
> ```
> Before writing code, always:
> 1. Write the failing test first
> 2. Run the test and confirm it fails
> 3. Write the minimum code to make it pass
> 4. Run again and confirm it passes
> ```
>
> Why: Claude 4.7 already proposes tests alongside code when you ask for a feature — it doesn't need this explicit TDD reminder anymore. This block was useful when the model used to skip the test step on its own.
>
> Keep the rest of the file — the part about your project's test framework (Vitest vs Jest) is specific to your setup and worth keeping.
>
> Result: file goes from 45 lines to ~22 lines.
>
> **Your call:** type **yes** to apply, **no** to skip, or tell me a different edit.

## Step 5: Execute

**If you have file-system tools and the scaffold is on disk:**
- **REMOVE** → delete the file or directory
- **MODIFY** → apply the edit
- **KEEP** → do nothing

**If you don't have file-system tools** (chat-only environment, scaffold lives in a web UI):
- Output the exact action the user should take, e.g., "In your custom GPT's instructions, replace the paragraph starting with X with the text below" or "Delete this section from your `.cursorrules`".

## Step 6: Move to the next

Repeat until done. If the user says "skip the rest" or similar, stop gracefully.

## Step 7: Final summary

When all scaffolds are processed, summarize:

- Counts: kept N, modified M, removed R
- Anything you couldn't judge confidently — flag for the user to revisit
- Patterns you noticed across multiple scaffolds (e.g., "three of these worked around a limitation the current model no longer has — worth noticing as a habit") — that's signal about how their scaffolding tends to age

## Bias

When in doubt, lean **REMOVE**. Bloat is the failure mode tare exists to address. A scaffold that "might still be useful sometimes" adds context cost on every invocation and can misfire. The user can always reinstall.

## Out of scope (v1)

- Do not run A/B tests or evals. Your judgment is qualitative, based on reading the scaffold and reasoning about current model capabilities. Be honest when you're uncertain.
- Do not touch hooks, server configs, or settings files unless the user explicitly asks.
- Don't proactively ask which model the user is on — assume latest. They'll mention it if they're mixing models for cost.

## Why the name

A *tare* on a scale subtracts the weight of the container so you can measure what's actually inside. Same here: subtract the accumulated scaffolding so what's left is what's actually doing work.
