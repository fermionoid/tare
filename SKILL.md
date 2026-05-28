---
name: tare
description: Use this skill before building or scaffolding any AI workflow — when the user is designing a multi-step prompt, an agent pipeline, a chain of tool calls, or any structured way of getting an LLM to do a task. Also use when an existing AI workflow is stuck and the user is wondering why the model "isn't doing the obvious thing." Walks the user through five questions that expose where they are unnecessarily imposing human process on the model. Do NOT use for single-shot prompts, debugging individual outputs, small prompt tweaks, or general coding tasks unrelated to AI workflow design.
---

# tare

Five questions to ask yourself before scaffolding an AI workflow. The goal is to subtract assumptions you brought from "how a human does this," so what's left is what the AI actually needs.

Walk through them as *questions*, not as a checklist with right answers. The point is to slow down and notice where your framing is doing more work than the task itself requires.

## The questions

1. **Is this step a human constraint, or task essence?**
   Many steps in your design exist because *humans* needed them — stages, handoffs, intermediate artifacts. An AI may not need any of that. Mark which steps survive if you remove the human-process residue.

2. **Are you decomposing the problem, or decomposing the process?**
   Decomposing the *problem* (goal → subgoals) is fine. Decomposing the *process* (feeding the AI your manual procedure step by step) is just imposing your slower way of working on the model. If your subgoals read like "first do A, then do B, then do C," reconsider.

3. **Is your knowledge going into evaluation criteria, or into steps?**
   Your expertise should land in *what counts as a good output* (criteria), not *do A then B* (steps). Criteria give direction and leave the path open. Steps lock the path and hide the criteria. If you find yourself writing steps, ask: what evaluation criterion would have made this step unnecessary?

4. **Does this really have to be serial?**
   Most serial structures are inertia, not necessity. Where can things run in parallel? Where can the model decide its own order? Don't hardcode sequence that doesn't earn its keep.

5. **What's the worst case if you give the model more freedom?**
   Most over-constraint comes from imagined risk. Write the worst case out concretely. If it's small and reversible, the constraint isn't earning its place — defer to the model and watch what happens.

## When to invoke

- Designing a new AI workflow → run before building
- An existing workflow is stuck on something the model *should* handle → run as audit
- A new major model version just shipped → re-run on the workflows you still use, because past harness becomes present overfit

## When not to invoke

- Single-shot prompts
- Debugging individual outputs
- Small prompt tweaks
- General coding tasks unrelated to designing AI workflows

## Why it's called tare

A *tare* button on a scale subtracts the weight of the container so you can measure what's actually inside. These questions don't add anything to your design — they remove the assumptions you brought from your own habits, so what's left is what the AI actually needs.
