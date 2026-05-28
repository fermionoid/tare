# tare

> Subtract the scaffolding that's no longer earning its keep.

A small skill I made for myself, because Superpowers had started feeling cluttered and a Lenny podcast with Cat Wu about how AI scaffolding ages into bloat reminded me I could just build a tool for it.

tare walks through your installed agent scaffolds — skill files, custom instructions, slash commands, project rules — and suggests what to do with each.

Works wherever you run agents: Claude Code, Codex CLI, ChatGPT, Cursor.

## What it does

When you invoke tare, the agent walks through your scaffolds and judges each one:

- Is it still solving a problem the current model has?
- Does it overlap with something else you've installed?
- Is its trigger or wording correctly scoped?

You get **KEEP / MODIFY / REMOVE** per scaffold with reasoning. KEEPs proceed without asking. For MODIFY or REMOVE, you type the action you want (`modify`, `remove`, `keep`, or describe a custom edit). The agent does the file edits and deletions where it has tools, or outputs the exact change for you to apply elsewhere.

Want to see the whole audit before any changes happen? Say *"preview my scaffolds"* or *"dry run"* — tare produces a one-line-per-scaffold summary and asks at the end which (if any) to apply.

> **A note on pacing.** tare pauses on every **MODIFY** or **REMOVE** recommendation, waiting for you to type `modify` / `keep` / `remove` (or describe what you want instead). **KEEP** recommendations auto-proceed — no input needed, tare just announces the reason and moves on. So if tare seems to "stop halfway," it didn't crash — it's waiting on you to decide.

## When to run it

- After a major model version ships
- When your context feels cluttered or instructions fire when they shouldn't
- When you've layered multiple skill packs or prompt sources and suspect duplication

Not for: small prompt tweaks, single-shot prompting, general debugging.

Manual trigger only — tare doesn't watch for model updates. Run it when you feel like checking in.

## Install

### Claude Code

```bash
mkdir -p ~/.claude/skills/tare
curl -fsSL https://raw.githubusercontent.com/fermionoid/tare/main/SKILL.md \
  -o ~/.claude/skills/tare/SKILL.md
```

Then in a session, say *"run tare"* or *"audit my skills"*.

### Codex CLI

```bash
mkdir -p ~/.codex/skills/tare
curl -fsSL https://raw.githubusercontent.com/fermionoid/tare/main/SKILL.md \
  -o ~/.codex/skills/tare/SKILL.md
```

Then in a Codex session, say *"run tare"* or *"audit my skills"*. Codex's skill directory layout mirrors Claude Code's, so no other config needed.

### ChatGPT (web or desktop)

Two options:

1. **In a project or custom GPT** — paste the body of `SKILL.md` (everything below the `---` frontmatter) into the instructions field. Then start any chat by saying *"run tare on the scaffolds I'm about to paste"* and paste in the prompts/instructions you want audited.
2. **Ad-hoc** — paste the body of `SKILL.md` at the top of a fresh chat, then ask it to audit whatever you paste next.

Since ChatGPT can't edit files on your machine, tare will tell you what to change and you apply it yourself in ChatGPT settings or your prompt source.

### Cursor / Windsurf / other agents

Drop `SKILL.md` (with or without the frontmatter) into your rules / instructions directory. The body is plain markdown — no platform-specific syntax.

## Why "tare"

The *tare* button on a scale subtracts the weight of the container so you measure what's actually inside. Same idea here.

## Take it or rewrite it

I made tare for myself. If it fits your taste, install it. If you'd rather lift the idea and write your own version, that's also fine — the concept matters more than this particular file.

## License

MIT
