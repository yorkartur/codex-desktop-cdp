# Example Patterns

These examples are inspiration, not required architecture.

The important pattern is:

```text
local UI captures a workflow moment -> local backend sends structured context -> Codex acts in the visible chat
```

This is most useful when the app is opened from a project, knowledge base, or Second Brain where Codex has loaded instructions and skills. The local app should classify the workflow moment, then send a skill hint or action label with the handoff.

A useful app should also show:

- the working directory the backend is running from,
- whether project instructions such as `AGENTS.md` exist,
- which local skills were discovered,
- which thread will receive the handoff.

For a project people will actually reuse, add a small local skill that starts the app. The skill should live in the consuming project, not in this repository:

```text
.agents/skills/start-my-codex-app/SKILL.md
```

That launcher skill should also establish the target Codex thread for the app session. The app should open with a known thread id and use that id for every handoff:

```text
start skill -> app opens with target thread id -> handoff focuses that thread -> backend inserts through CDP
```

If the app cannot determine a target thread id, it should clearly warn that it will send to the currently visible Codex composer.

## Reading Companion

Reading Companion is a source-first app.

```text
source -> selected passage -> note -> Codex
```

Use this pattern when the user is reading something and wants to send evidence into Codex:

- papers,
- books,
- articles,
- PDFs,
- source documents,
- codebase notes.

The local app should capture:

- source title,
- source path or URL,
- selected passage,
- user note,
- target knowledge base or project,
- action or ingest skill hint,
- target Codex thread id,
- write policy.

Codex can then discuss, summarize, connect to a knowledge base, or run an ingest skill.

## Writing Companion

Writing Companion is a draft-first app.

```text
draft -> selected text -> skill/action -> Codex
```

Use this pattern when the user is writing and wants Codex to work on a bounded part of the text:

- blog posts,
- essays,
- PR descriptions,
- sales emails,
- project notes,
- scripts.

The local app should capture:

- draft path,
- selected text,
- selection offsets when possible,
- full draft as context when useful,
- action or skill name,
- target Codex thread id,
- explicit write policy.

Codex can then critique, rewrite, invoke a skill, or make a controlled edit if the action allows it.

## The General Rule

Use this bridge when chat is the wrong UI but Codex is the right reasoning layer.

Good:

```text
specific local workflow -> structured context -> Codex skill/action
```

Bad:

```text
generic textarea -> generic chatbot response
```
