---
name: tare
description: Use this skill when the user explicitly asks to audit, prune, clean, or "tare" their installed Claude Code scaffolds — phrases like "run tare", "audit my skills", "tare my scaffolds", "clean up my skills", "my skills feel bloated". Also appropriate to surface after a major model version has shipped, since past scaffolding tends to become overfit to past model behavior. The skill walks through each scaffold under ~/.claude/skills/, judges whether it still earns its cost given current model capabilities, and proposes KEEP / MODIFY / REMOVE per scaffold for the user to confirm — then executes the change. Do NOT use for general coding tasks, single-shot prompts, debugging individual outputs, or anything other than explicit scaffold auditing.
---

# tare

You are running a scaffold audit. The user has installed various skills, custom instructions, and commands over time. Some of these were crutches for past model limitations and the model no longer needs them. Some overlap with each other. Some are scoped wrong and fire when they shouldn't. The user wants to find and prune the bloat — they don't want to do the file edits themselves, you will.

## Procedure

### 1. Announce and enumerate

Tell the user you're starting a tare audit. Then list everything under `~/.claude/skills/` — each subdirectory's `SKILL.md` (or top-level `.md` file), with the `description` frontmatter line shown truncated. Also list `~/.claude/commands/*.md` if any exist.

Skip `tare` itself.

If a skill directory contains files beyond `SKILL.md` (e.g., a multi-file plugin), flag it visually — those need extra care before removal.

Tell the user roughly how many scaffolds you found and that you'll walk through them one at a time.

### 2. For each scaffold, judge three things

- **Is it still solving a problem the current model has?** If the capability has been internalized by the model, the scaffold is dead weight.
- **Does it overlap with another installed scaffold?** Duplicates accumulate when users layer multiple plugin packs.
- **Is its trigger description correctly scoped?** Too broad → fires when not wanted. Too narrow → never fires. Stale wording → may misfire on the current model's behavior.

Read the actual `SKILL.md` body for context. Don't judge by description alone.

### 3. Produce one of three recommendations

- **KEEP** — still net positive. Move on.
- **MODIFY** — has value but needs editing. Show the specific proposed diff (which lines to change, exact replacement text).
- **REMOVE** — no longer earns its cost. State the reason in one sentence.

### 4. Present one scaffold at a time

For each scaffold, show:

- Path
- First line of its current description (truncated to ~100 chars)
- Your recommendation in bold: **KEEP** / **MODIFY** / **REMOVE**
- 2-3 sentences of reasoning
- For **MODIFY**, the concrete proposed change as a diff or before/after

Then wait for the user to either confirm your recommendation or override it. Accept short answers ("ok", "yes", "k", "skip", "remove instead", "modify like this: …").

### 5. Execute the confirmed action

- **REMOVE** → delete the skill's directory (or its single file). Use Bash.
- **MODIFY** → apply the edit with the Edit tool.
- **KEEP** → do nothing, move on.

If the user overrides with a different change, apply that instead.

### 6. Move to the next scaffold

Repeat until done. If the user says "skip the rest" or similar, stop gracefully.

### 7. Final summary

When all scaffolds are processed, summarize:

- Count: kept N, modified M, removed R
- Anything you couldn't judge confidently — flag these by path for the user to revisit later
- If you noticed patterns across multiple scaffolds (e.g., "three skills all assumed the model couldn't use sub-agents, which is no longer true"), mention them — that's signal for the user about their own scaffolding habits

## Bias

When in doubt, lean **REMOVE**. Bloat is the failure mode tare exists to address. A scaffold that "might still be useful sometimes" adds context-window cost and can misfire — the user can always reinstall if a removal turns out wrong. Do not preserve scaffolds out of politeness.

## Out of scope (v1)

- Do not audit project-level `CLAUDE.md` files unless the user explicitly asks.
- Do not touch `~/.claude/settings.json`, hooks, or MCP configs.
- Do not run A/B tests or evals. Your judgment is qualitative, based on reading the scaffold and reasoning about current model capabilities. Be honest when you're uncertain rather than fabricating confidence.

## Why the name

A *tare* on a scale subtracts the weight of the container so you can measure what's actually inside. Same here: subtract the scaffolding so what's left is what's actually doing work.
