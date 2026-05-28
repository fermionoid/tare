# tare

> Subtract the scaffolding that's no longer earning its keep.

A Claude Code skill that audits your installed scaffolds — skills, slash commands, custom instructions — and prunes the ones that have become bloat. Past scaffolding tends to overfit to past model behavior; tare finds the overfit and removes or edits it on your behalf.

## What it does

When you invoke tare, it walks through each scaffold you've installed and judges:

- Is it still solving a problem the current model has?
- Does it overlap with something else you've installed?
- Is its trigger description correctly scoped?

For each scaffold it gives you a **KEEP / MODIFY / REMOVE** recommendation with reasoning. You confirm the direction (one keystroke per scaffold); tare does the file edits and deletions for you. You don't open an editor.

## When to run it

- After a major model version ships — old crutches turn into overfit
- When your context feels cluttered, or skills fire when they shouldn't
- When you've layered multiple skill packs and suspect duplication

Not for: small prompt tweaks, single-shot prompting, or general debugging.

## Install (Claude Code)

```bash
mkdir -p ~/.claude/skills/tare
curl -fsSL https://raw.githubusercontent.com/fermionoid/tare/main/SKILL.md \
  -o ~/.claude/skills/tare/SKILL.md
```

Then in a Claude Code session, say *"run tare"* or *"audit my skills"*.

## Why "tare"

The *tare* button on a scale subtracts the weight of the container so you measure what's actually inside. Same idea here.

## License

MIT
