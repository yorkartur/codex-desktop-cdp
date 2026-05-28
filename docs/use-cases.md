# Use Cases

The point is not to install someone else's app.

The point is to copy the pattern and build your own local interface:

```text
local UI -> local backend -> Codex Desktop CDP -> visible Codex chat
```

Use this when a human benefits from a focused UI, but Codex is the better place for reasoning, skills, file edits, terminal work, or follow-up actions.

## What Makes A Use Case Codex-Native?

A good use case has:

- a human selecting, writing, reviewing, approving, or triggering something,
- local context Codex can use,
- a reason for the result to land in the visible Codex conversation,
- a useful next action Codex can take after receiving the handoff.

If the workflow is fully automatic and nobody needs to inspect or steer it from a UI, a script, skill, CLI, or backend integration is usually simpler.

## Simple Examples

- **Reading Companion**: select a passage from a local document, add a note, and send it to Codex with source context.
- **Writing Companion**: select draft text, invoke a writing skill, or ask Codex to edit the local Markdown file after approval.
- **Knowledge Base Browser**: browse local notes, select relevant context, and send a structured handoff to Codex.
- **Dungeon Master Interface**: a human DM uses buttons, state, and notes while Codex generates events, NPCs, summaries, or consequences.

The available examples in this repo are Reading Companion and Writing Companion. Treat them as reference patterns, not finished products.
