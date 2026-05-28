# tare

> Tare your assumptions before building an AI workflow.

**tare** /tɛə(r)/ — *verb.* on a balance scale, to subtract the weight of the container so you can measure what's actually inside.

A single Claude Code skill. Five questions to ask yourself before you scaffold an AI workflow, so you don't quietly impose your own habits on a model that doesn't share them.

## What it's for

Most failures in agentic workflows aren't from the model being weak — they're from over-specifying. You wrote out the steps you would take, fed them to the model, and now the model is doing your less-good version of the task.

tare is five questions to catch that before you commit.

## When it triggers

- Designing a new AI workflow
- An existing workflow is stuck on something the model "obviously" should handle
- A major model version just shipped — re-run on the workflows you still use, because past harness becomes present overfit

Not for: single-shot prompting, debugging individual outputs, general coding.

## Install (Claude Code)

```bash
mkdir -p ~/.claude/skills/tare
curl -fsSL https://raw.githubusercontent.com/fermionoid/tare/main/SKILL.md -o ~/.claude/skills/tare/SKILL.md
```

Or clone the repo and copy `SKILL.md` over.

Claude will surface the skill when relevant. You can also invoke it explicitly: *"run tare on this design."*

## Install (other agents)

Copy `SKILL.md` to wherever your agent loads skills or instructions. The body is plain markdown — only the YAML frontmatter is Claude-skill flavored, and any agent that ignores frontmatter will still read the body correctly.

## Why "tare"

A tare button on a scale subtracts the weight of the container. It doesn't measure something new; it removes something old so the measurement is honest.

That's what these questions do: subtract the assumptions you brought from *"how a human does this,"* so what's left is what the AI actually needs.

## License

MIT
