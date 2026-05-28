# tare

> Subtract the scaffolding that's no longer earning its keep.

A general agent skill that audits the scaffolds you've accumulated — skill files, custom instructions, system prompts, slash commands, project rules — and prunes the ones that have become bloat. Works wherever you run agents: Claude Code, Codex CLI, ChatGPT, Cursor, and others.

Past scaffolding tends to overfit to past model behavior. tare finds the overfit and removes or edits it on your behalf where the agent has tools to do so, or tells you what to change where it doesn't.

## What it does

When you invoke tare, the agent walks through your scaffolds and judges each one:

- Is it still solving a problem the current model has?
- Does it overlap with something else you've installed?
- Is its trigger or wording correctly scoped?

You get **KEEP / MODIFY / REMOVE** per scaffold with reasoning. You confirm the direction; the agent does the file edits and deletions where it has tools, or outputs the exact change for you to apply elsewhere.

## When to run it

- After a major model version ships
- When your context feels cluttered or instructions fire when they shouldn't
- When you've layered multiple skill packs or prompt sources and suspect duplication

Not for: small prompt tweaks, single-shot prompting, general debugging.

> **tare doesn't auto-detect model updates in v1.** You trigger it manually when you notice a release or feel the bloat. A future version may add a hook that surfaces a reminder when your installed agent's model has changed since the last audit, but that's a separate piece of infrastructure — not part of the skill itself.

## Install

### Claude Code

Install the skill (auto-triggers when you mention auditing):

```bash
mkdir -p ~/.claude/skills/tare
curl -fsSL https://raw.githubusercontent.com/fermionoid/tare/main/SKILL.md \
  -o ~/.claude/skills/tare/SKILL.md
```

Optional: also install the slash command for explicit invocation:

```bash
mkdir -p ~/.claude/commands
curl -fsSL https://raw.githubusercontent.com/fermionoid/tare/main/commands/tare.md \
  -o ~/.claude/commands/tare.md
```

Then in a session, either say *"run tare"* / *"audit my skills"* (natural language, triggers the skill) or type `/tare` (slash command, hands off to the same skill).

### Codex CLI

Drop `SKILL.md` into your Codex skills directory (check your Codex config for the exact path), or paste its contents into your `AGENTS.md` / instructions file.

### ChatGPT (web or desktop)

Two options:

1. **In a project or custom GPT** — paste the body of `SKILL.md` (everything below the `---` frontmatter) into the instructions field. Then start any chat by saying *"run tare on the scaffolds I'm about to paste"* and paste in the prompts/instructions you want audited.
2. **Ad-hoc** — paste the body of `SKILL.md` at the top of a fresh chat, then ask it to audit whatever you paste next.

Since ChatGPT can't edit files on your machine, tare will tell you what to change and you apply it yourself in ChatGPT settings or your prompt source.

### Cursor / Windsurf / other agents

Drop `SKILL.md` (with or without the frontmatter) into your rules / instructions directory. The body is plain markdown — no platform-specific syntax.

## Why "tare"

The *tare* button on a scale subtracts the weight of the container so you measure what's actually inside. Same idea here.

## License

MIT
