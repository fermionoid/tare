---
name: tare
description: Use this skill when the user explicitly asks to audit, prune, clean, or "tare" their accumulated agent scaffolds — phrases like "run tare", "audit my skills", "tare my scaffolds", "clean up my custom instructions", "my prompts feel bloated". Also appropriate to surface after a major model version has shipped, since past scaffolding tends to become overfit to past model behavior. Walks through the user's scaffolds (skill files, custom instructions, system prompts, slash commands, project rules), judges each one's value vs cost given current model capabilities, proposes KEEP / MODIFY / REMOVE for the user to confirm, then executes the change where the agent has file-system tools — otherwise outputs the exact change for the user to apply manually. Do NOT use for general coding tasks, single-shot prompts, debugging individual outputs, or anything other than explicit scaffold auditing.
---

# tare

You are running a scaffold audit. The user has accumulated scaffolds — skill files, custom instructions, system prompts, slash commands, project-level rules — across one or more agent platforms (Claude Code, ChatGPT, Codex, Cursor, etc.). Some of these were crutches for past model limitations and the current model doesn't need them anymore. Some overlap. Some are scoped wrong. tare's job is to find the bloat and remove or edit it.

## Step 1: Determine scope (don't ask if you can figure it out)

**If you have file-system tools, just go.** Default to **only the host platform you're running on** — don't reach across to other platforms' scaffolds without the user explicitly asking. Specifically:

- Running inside **Claude Code** → audit only `~/.claude/skills/*` and `~/.claude/commands/*` and `~/.claude/CLAUDE.md`. Do not touch `~/.codex/` or `~/.cursor/` or other platforms unless the user explicitly says so.
- Running inside **Codex CLI** → audit only `~/.codex/skills/*` (excluding `.system/`), `~/.codex/superpowers/` if installed, and `~/.codex/AGENTS.md`. Do not touch `~/.claude/` or other platforms unless the user explicitly says so.
- Running inside **Cursor / Windsurf** → audit only `.cursor/rules/*` or `.cursorrules` in the current project root.
- Anything else with file access → audit only that platform's standard locations.

**Cross-platform audits exist but must be opt-in.** If the user says *"audit all my scaffolds"* / *"include Claude Code too"* / *"check across platforms"*, then expand scope to include those locations. Otherwise, don't surprise the user by modifying files belonging to a platform they didn't invoke you from.

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

Scaffolds come in four fundamentally different kinds, and they deserve different biases:

- **Capability crutch** — written to patch something the model used to fail at, where the only function is to remind the model of something it would otherwise miss. These age out as models improve. Examples: "remember to use the to-do list tool", "list the call sites before refactoring across them".
- **Personal style / values / workflow** — reflects how the user wants the model to think, speak, weight things, or describes their specific setup that the model can't infer. Examples: "always start with the actual problem", "default to first-principles thinking", "be honest about uncertainty", "this project uses Vitest", "save observations to my notes file when X happens".
- **Safety / workflow guard** — a scaffold that *explicitly* enforces a precondition the user wants maintained even when the model "usually" handles it. Examples: "always confirm before deleting", "check git status before commit", "run tests before declaring done", "don't overwrite user changes without asking", "backup before destructive operation", "verify assumptions before answering", "always write a failing test before code". Default **KEEP**. MODIFY only to shorten or clarify. **REMOVE never applies** — "the model usually does this" is exactly the case a guard exists for, because *usually* silently becomes *sometimes doesn't*, and the user has chosen to harden against that.
- **Convention slot** — a file whose value lives in *the filename being a recognized hook*, not in its content. The agent platform looks for this exact name and reads whatever's inside (which may be nothing). Known convention filenames as of 2026 — treat each of these as a slot:
  - **`AGENTS.md`** — the rising cross-tool standard, read by 60+ agentic tools including Claude Code, Codex, Cursor, Aider, Gemini CLI, Windsurf, GitHub Copilot, Devin, Continue, Roo Code, Zed AI, MiniMax, and many more. Protecting `AGENTS.md` alone covers most of the ecosystem.
  - **`CLAUDE.md`** — Claude Code (both `~/.claude/CLAUDE.md` and project-root `CLAUDE.md`)
  - **`GEMINI.md`** — Gemini CLI (hierarchical: `~/.gemini/GEMINI.md` + workspace + file-specific)
  - **`.cursorrules`** (legacy) and **`.cursor/rules/*.mdc`** (current) — Cursor
  - **`.windsurfrules`** — Windsurf
  - **`CONVENTIONS.md`** — Aider
  - **`user_rules.md`** and **`.trae/rules/project_rules.md`** — Trae (ByteDance)
  - And similar platform-specific hook files. **If the filename pattern matches a tool's documented convention, treat it as a slot.**

**The distinction matters because "the model CAN do X" is not the same as "the model WILL do X consistently / announce it / emphasize it."** A style instruction shifts visible model behavior toward what the user wants even when the underlying capability is already there. Erasing it erases the user's voice.

**A convention slot is even more delicate:** an empty `AGENTS.md` looks like dead weight, but it's actually a placeholder the platform hooks into. Removing it loses the slot — the user can't easily get back the "if I drop instructions here, they're picked up automatically" behavior without recreating the file. **Empty does not mean dead.**

**If you can't tell which kind a scaffold is, ask the user one line:** *"Is this an aging workaround, your personal style, or a platform convention slot? Either is fine — I just want to bias the right way."* Don't guess.

Then judge:

- **For capability crutches** — Has the current model internalized this? If yes → REMOVE candidate. If no → KEEP. Assume the user is on the latest model their platform offers; if they've mentioned mixing in older/cheaper models, bias toward keeping crutches the weaker model still needs.
- **For personal style / values / workflow** — Default to KEEP. Only suggest REMOVE if it actively contradicts another scaffold or current default behavior. MODIFY is still on the table if the wording is stale or too broad.
- **For both kinds** — Does it overlap with another installed scaffold? Flag for MODIFY. Is the trigger / wording scoped correctly? Flag for MODIFY.

Read the full body of the scaffold for context, not just its description or title.

## Step 3: Recommend (with hard protected check)

One of three labels:

- **KEEP** — still net positive
- **MODIFY** — has value but needs editing. Include the concrete proposed change (specific lines, before/after, replacement text)
- **REMOVE** — no longer earns its cost, AND you can articulate the concrete user-visible behavior change removal will produce

### Hard protected check — run this before any REMOVE recommendation

If any of the following is true, the recommendation is **KEEP** (or MODIFY, if there's substantive content to edit), regardless of how stale or redundant the scaffold looks:

1. **The filename matches a convention slot** (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursorrules`, `.cursor/rules/*`, `.windsurfrules`, `CONVENTIONS.md`, `user_rules.md`, `.trae/rules/*`, or similar platform hook)
2. **The scaffold is a safety / workflow guard** (encodes a precondition the user wants enforced even when the model "usually" handles it — see Step 2)
3. **The scaffold lives inside a third-party bundle** (e.g., `~/.codex/superpowers/*`, plugin-managed directories) — bundle-internal files are not individually pruneable; for these, the only legitimate bundle-level actions are KEEP / UPDATE / UNINSTALL
4. **You cannot finish the sentence** *"After removing this, the user will no longer experience [specific concrete behavior]."* If the best you can say is "it looks redundant," the burden of proof has not been met — default KEEP.

A REMOVE recommendation that fails any of these four checks is silently destructive. Don't produce it.

## Step 4: Present each scaffold clearly

The user must be able to decide **yes / no / skip from what you show them alone** — don't make them open files or remember what they wrote months ago. Use this exact format per scaffold:

---

### Scaffold N of M: `<path>` (X lines)

**What it does:** one plain-language sentence.

**Recommendation: KEEP / MODIFY / REMOVE**

**For KEEP** — one sentence on *why it's net positive*, framed by the scaffold's class:

- **Convention slot** → why the slot needs to remain (it's a platform hook; presence is the value)
- **Safety / workflow guard** → what risk or precondition it constrains
- **Personal style / values** → what user-chosen behavior it makes explicit
- **Capability gap** → what the current model wouldn't do by default that this scaffold ensures

No quoting needed. End the block with: *"Moving on to the next scaffold."* and proceed without prompting. KEEP has no action to confirm — don't ask the user to make a decision about a non-action. (They can always interrupt if they disagree.)

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

**Before executing REMOVE, run the protected check one more time** (see Step 3). If the path matches a convention slot, is a safety/workflow guard, or lives inside a third-party bundle — abort REMOVE and offer KEEP. Belt-and-suspenders: this catches any case where classification missed but the execution-time check sees the protected path.

**For third-party bundles** (e.g., `~/.codex/superpowers/*`, `~/.claude/plugins/*`): never execute per-file REMOVE on bundle internals. The only bundle-level actions are:
- **KEEP** the bundle as-is
- **UPDATE** the bundle (suggest re-installing latest; defer to the bundle's own update mechanism)
- **UNINSTALL** the entire bundle (suggest the platform's uninstall command, don't `rm -rf` files directly)

Per-file observations *within* a bundle can be reported as commentary ("3 of these skills look outdated"), but the action belongs upstream (open issue/PR) or via platform blocklist mechanisms.

**Before any destructive operation (MODIFY or REMOVE), write a backup first.** Copy the target file to `<path>.tare-backup` (or move the target to `<path>.tare-backup` for REMOVE). After the edit, verify the result looks correct (file not unexpectedly truncated, contains the expected new text). If verification fails or the edit tool returns an error, **restore from the backup and surface the error to the user — do not leave the file in a corrupted state**. On success, delete the backup.

This safeguard exists because edit tools occasionally fail mid-write (encoding errors, partial truncation, permission glitches). Without a backup, a corrupted edit silently destroys the user's content. With a backup, every destructive operation is reversible by definition.

**If you have file-system tools and the scaffold is non-bundle:**
- **REMOVE** → write backup → delete the file or directory → if anything goes wrong, restore from backup
- **MODIFY** → write backup → apply the edit → verify file looks intact → restore from backup if not → on success, delete backup
- **KEEP** → do nothing (no backup needed)

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

**For safety / workflow guards — always KEEP.** When a scaffold says "always confirm before deleting" or "verify before answering" or "run tests before declaring done," its value is *exactly* in the case where the model would "usually" handle it on its own. The user wrote this down because they didn't want to gamble on *usually*. Removing it removes the guarantee. REMOVE never applies; MODIFY only to shorten or sharpen, not to delete the guard.

**For third-party bundles — operate at bundle level, never per-file.** Bundle internals belong upstream. Bundle-level actions: KEEP / UPDATE / UNINSTALL.

**When uncertain which kind a scaffold is — ask before recommending.** Don't guess and risk deleting someone's voice or breaking a convention hook. Tare exists to catch bloat, not to flatten personal style or remove protected slots.

## Never touch — these are out of scope entirely

These files are *not* user instructions and never should be candidates for any tare action (KEEP / MODIFY / REMOVE):

- **Settings files**: `*/settings.json`, `*/settings.local.json`, `~/.codex/config.toml`, `~/.kimi/config.toml`, any platform's main config TOML/JSON
- **MCP server configs**: `.mcp.json`, `mcp.json`, equivalents elsewhere — removing breaks MCP integrations silently
- **Ignore files**: `.cursorignore`, `.aiderignore`, `.cursorindexingignore`, `.gitignore`
- **Plugin / marketplace state**: `blocklist.json`, `installed_plugins.json`, `known_marketplaces.json`, plugin install manifests
- **History / memory / session state**: `history.jsonl`, `memory/` directories, session files, conversation logs — these are state, not scaffolds
- **Hooks**: anything in `*/hooks/` or referenced by settings' hooks config

If you encounter any of these during enumeration, note them in your scope statement as *"out of scope — these aren't tare's concern"* and move on. Don't even produce a recommendation block for them.

## Other out-of-scope rules

- Do not run A/B tests or evals. Your judgment is qualitative, based on reading the scaffold and reasoning about current model capabilities. Be honest when you're uncertain.
- Don't proactively ask which model the user is on — assume latest. They'll mention it if they're mixing models for cost.

## The articulation rule (most important guardrail)

**Before recommending REMOVE, you must be able to articulate the specific user-visible behavior change that removal will produce.** "It looks redundant" or "it's empty" is not enough. You should be able to finish the sentence: *"After removing this, the user will no longer experience [specific concrete behavior]."*

If you cannot finish that sentence with something concrete, default to **KEEP**. This is the guardrail that prevents silently-destructive REMOVE recommendations on files whose value isn't in their content (convention slots, hook files, structural placeholders). The burden of proof is on REMOVE, not KEEP.

## Why the name

A *tare* on a scale subtracts the weight of the container so you can measure what's actually inside. Same here: subtract the accumulated scaffolding so what's left is what's actually doing work.

## Sanity checklist (regression cases)

These are cases tare has gotten wrong in past dogfooding. The agent must produce the expected outcome on each. If any of these would not behave as listed, the SKILL has regressed:

- Empty `~/.codex/AGENTS.md` (or any path's `AGENTS.md`) → **KEEP** (convention slot)
- Empty `~/.claude/CLAUDE.md` (or project-root `CLAUDE.md`) → **KEEP** (convention slot)
- Empty `GEMINI.md`, `.cursorrules`, `.windsurfrules`, `CONVENTIONS.md` → **KEEP** (convention slot)
- A scaffold instructing *"always confirm before deleting"* / *"check git status before commit"* / *"verify assumptions before answering"* / *"run tests before declaring done"* → **KEEP** or **MODIFY** to shorten, **never REMOVE** because "the model usually does this"
- Anything under `~/.codex/skills/.system/*` (or analogous platform-bundled paths) → **SKIPPED in Step 1**, never produces a recommendation block
- A skill inside `~/.codex/superpowers/<name>/` (or any third-party bundle) → audited at **bundle level**, no per-file REMOVE/MODIFY
- A scaffold expressing user identity or style (e.g., *"always lead with the actual problem"*, *"reason from first principles"*, *"be honest about uncertainty"*) → **KEEP**, even when the model now does this by default
- A symlink in the user's skills directory → check `readlink` before acting; if symlinked to a project the user owns, recommend KEEP unless the user explicitly says otherwise
- Running tare inside Codex with files belonging to Claude Code on the same machine → audit **only `~/.codex/`** by default. Do not touch `~/.claude/` files without explicit user instruction
- Edit tool returns an error mid-MODIFY → **restore from the `.tare-backup` immediately and surface the error**, never leave the target file in a partial-write state
