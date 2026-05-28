---
name: tare
description: Use this skill when the user explicitly asks to audit, prune, clean, or "tare" their accumulated agent scaffolds — phrases like "run tare", "audit my skills", "tare my scaffolds", "clean up my custom instructions", "my prompts feel bloated". Also appropriate to surface after a major model version has shipped, since past scaffolding tends to become overfit to past model behavior. Walks through the user's scaffolds (skill files, custom instructions, system prompts, slash commands, project rules), judges each one's value vs cost given current model capabilities, proposes KEEP / MODIFY / REMOVE for the user to confirm, then executes the change where the agent has file-system tools — otherwise outputs the exact change for the user to apply manually. Do NOT use for general coding tasks, single-shot prompts, debugging individual outputs, or anything other than explicit scaffold auditing.
---

# tare

You are running a scaffold audit. The user has accumulated scaffolds — skill files, custom instructions, system prompts, slash commands, project-level rules — across one or more agent platforms (Claude Code, ChatGPT, Codex, Cursor, etc.). Some of these were crutches for past model limitations and the current model doesn't need them anymore. Some overlap. Some are scoped wrong. tare's job is to find the bloat and remove or edit it.

## Step 1: Determine scope (don't ask if you can figure it out)

**If you have file-system tools, just go.** Default to the standard scaffold locations for whatever platform you're running on:

- Claude Code → `~/.claude/skills/*` and `~/.claude/commands/*`
- Codex CLI → `~/.codex/skills/*` (excluding `.system/`), the user's `~/.codex/superpowers/` if installed, and `~/.codex/AGENTS.md`
- Cursor / Windsurf → `.cursor/rules/*` or `.cursorrules` in the current project root
- Anything else with file access → check that platform's obvious config location

**Always skip these — they're not the user's choices:**

- `tare` itself
- **Platform-bundled / system scaffolds.** Strong signals: path contains `/.system/` (Codex convention), directory name describes the platform's own runtime (e.g., `codex-primary-runtime`, `claude-primary-*`), or SKILL.md frontmatter explicitly marks it as bundled. These are infrastructure shipped by the platform, not things the user installed — auditing them wastes attention with no possible action (the user can't remove what the platform manages).

**For borderline scaffolds** (user's skills directory but the user might not have installed it themselves — e.g., something auto-installed by a sibling tool like a Mac menu-bar app): don't silently include or exclude. Mention it once in your initial scope statement: *"I'll also include `chronicle` — that one might have been auto-installed by another tool; tell me if you'd rather skip it."*

State what you're about to audit in one sentence ("I'll audit N user-installed scaffolds…") and start. **Do not block on confirmation.** The user invoked you — they've already opted in.

**Only ask first if:**

- You're in a chat-only environment with no file tools (e.g., ChatGPT web) — the user has to paste what they want audited
- The user explicitly scoped it ("tare just my project rules", "audit only my custom GPT instructions") — follow that scope
- Multiple plausible default locations exist and you genuinely can't tell which they meant — but assume the broadest reasonable one before asking

## Operating modes

Before Step 2, decide which mode the user wants and stick to it:

- **Default mode** (the implicit one) — walk through scaffolds one at a time. For KEEP, announce and proceed without prompting. For MODIFY / REMOVE, pause for confirmation, then execute.
- **Dry-run / preview mode** — triggered when the user says *"preview"*, *"dry run"*, *"don't apply"*, *"just show me"*, *"audit only"*, or similar. In this mode you don't execute anything. Instead produce a single compact summary report — one line per scaffold:
  > `1. <path> (X lines) — **KEEP / MODIFY / REMOVE** — one-clause reason. Lines: X → Y.`
  
  After the full list, ask: *"Apply all? Apply some (tell me which numbers)? Apply none? Or want to dive into a specific scaffold for more detail?"* Based on the answer, either execute the chosen subset (using Step 4 templates for each) or stop.

## Step 2: Classify first, then judge

Scaffolds come in three fundamentally different kinds, and they deserve different biases:

- **Capability crutch** — written to patch something the model used to fail at, or to enforce a workflow that helped a weaker model perform. These age out as models improve. Examples: "remember to use the to-do list tool", "always write a failing test before code", "verify your assumptions explicitly before answering".
- **Personal style / values / workflow** — reflects how the user wants the model to think, speak, weight things, or describes their specific setup that the model can't infer. Examples: "always start with the actual problem", "default to first-principles thinking", "be honest about uncertainty", "this project uses Vitest", "save observations to my notes file when X happens".
- **Convention slot** — a file whose value lives in *the filename being a recognized hook*, not in its content. The agent platform looks for this exact name and reads whatever's inside (which may be nothing). Examples: `AGENTS.md` (Codex / Cursor / Aider), `CLAUDE.md` (Claude Code), `GEMINI.md` (Gemini CLI), `.cursorrules` / `.cursor/rules/*` (Cursor), `.aider.conf.yml` (Aider), and similar platform conventions.

**The distinction matters because "the model CAN do X" is not the same as "the model WILL do X consistently / announce it / emphasize it."** A style instruction shifts visible model behavior toward what the user wants even when the underlying capability is already there. Erasing it erases the user's voice.

**A convention slot is even more delicate:** an empty `AGENTS.md` looks like dead weight, but it's actually a placeholder the platform hooks into. Removing it loses the slot — the user can't easily get back the "if I drop instructions here, they're picked up automatically" behavior without recreating the file. **Empty does not mean dead.**

**If you can't tell which kind a scaffold is, ask the user one line:** *"Is this an aging workaround, your personal style, or a platform convention slot? Either is fine — I just want to bias the right way."* Don't guess.

Then judge:

- **For capability crutches** — Has the current model internalized this? If yes → REMOVE candidate. If no → KEEP. Assume the user is on the latest model their platform offers; if they've mentioned mixing in older/cheaper models, bias toward keeping crutches the weaker model still needs.
- **For personal style / values / workflow** — Default to KEEP. Only suggest REMOVE if it actively contradicts another scaffold or current default behavior. MODIFY is still on the table if the wording is stale or too broad.
- **For both kinds** — Does it overlap with another installed scaffold? Flag for MODIFY. Is the trigger / wording scoped correctly? Flag for MODIFY.

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

**For KEEP** — one sentence on what unique value it adds that the current model wouldn't do by default. No quoting needed. End the block with: *"Moving on to the next scaffold."* and proceed without prompting. KEEP has no action to confirm — don't ask the user to make a decision about a non-action. (They can always interrupt if they disagree.)

**For MODIFY** — quote the *exact text being removed* in one code block. Quote the *exact text being added* in another (if any). Then 2-3 plain-language sentences on why. End with: *"Result: file goes from X lines to Y lines."* Then prompt:
> **Your call:** type **modify** to apply this edit, **keep** to leave the file unchanged, **remove** to delete the whole scaffold instead, or describe a different edit.

**For REMOVE** — quote the whole file (or the contiguous section being removed) in a code block. Then 2-3 plain-language sentences on why it's no longer earning its keep. If the scaffold contradicts how the model currently behaves by default, quote that conflicting default behavior too. Then prompt:
> **Your call:** type **remove** to delete, **keep** to leave it alone, **modify** if you'd rather edit, or describe a different action.

---

For MODIFY and REMOVE, wait for the user's answer before moving on. Accept short answers ("keep", "modify", "remove", "skip") or full descriptions ("modify like this: …"). For KEEP, you've already moved on — no input needed.

### Any other time you need user input — use the same bold-verb format

If you pause for the user to make a decision *outside* the per-scaffold template — for example, asking whether to include a borderline scaffold, flagging a transition point, or summarizing options before continuing — use the same structured prompt:

> **Your call:** type **modify** / **keep** / **remove** / or describe a different approach.

**Don't bury decision points in prose.** Phrases like *"I'd recommend modifying or removing this one"* let the action verbs blend into the paragraph and get missed on first read. The user must be able to spot the decision point at a glance — bolded verbs, separate line, no surrounding sentence-shaped prose.

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
> **Your call:** type **modify** to apply this edit, **keep** to leave the file unchanged, **remove** to delete the whole scaffold instead, or describe a different edit.

## Example of a good KEEP presentation (style case)

> ### Scaffold 4 of 5: `~/.claude/CLAUDE.md` lines 23–28
>
> **What it does:** instructs Claude to start by questioning assumptions and naming the actual problem before proposing solutions.
>
> **Recommendation: KEEP**
>
> This is a personal style preference. Claude 4.7 *can* reason from fundamentals on its own — but it doesn't always lead with *"what's the actual problem here?"* The explicit instruction makes that pattern reliable and visible. Not a capability gap to patch, it's a way of working you've chosen to make explicit. Removing this would technically save context, but it would also erase a behavior you specifically asked for.
>
> *Moving on to the next scaffold.*

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

**For capability crutches — lean REMOVE.** Bloat is a real failure mode and aged workarounds add context cost while sometimes misfiring. The user can always reinstall.

**For personal style / values / workflow — lean KEEP.** Even when the model could behave that way unprompted, the user has chosen to make the behavior explicit, and erasing that erases their voice. Only REMOVE these if they actively contradict another scaffold or current default behavior.

**For convention slots — always KEEP, regardless of content.** Empty or near-empty convention files (`AGENTS.md`, `CLAUDE.md`, etc.) are placeholders the platform hooks into — the slot itself is the value. Recommending REMOVE on these is one of the most damaging mistakes tare can make, because the user loses functionality silently. MODIFY is fine if the user wants to add content. REMOVE never applies.

**When uncertain which kind a scaffold is — ask before recommending.** Don't guess and risk deleting someone's voice or breaking a convention hook. Tare exists to catch bloat, not to flatten personal style or remove protected slots.

## Out of scope (v1)

- Do not run A/B tests or evals. Your judgment is qualitative, based on reading the scaffold and reasoning about current model capabilities. Be honest when you're uncertain.
- Do not touch hooks, server configs, or settings files unless the user explicitly asks.
- Don't proactively ask which model the user is on — assume latest. They'll mention it if they're mixing models for cost.

## Why the name

A *tare* on a scale subtracts the weight of the container so you can measure what's actually inside. Same here: subtract the accumulated scaffolding so what's left is what's actually doing work.
