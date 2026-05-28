# App Server vs CDP

The key lesson: there are two different integration goals.

The CDP bridge described here is unstable and experimental. Use it for local prototypes and personal workflows, not production apps.

## Goal 1: Persist A Turn

Use Codex App Server when you want to create or resume a thread and start a real Codex turn programmatically.

The App Server model is thread-oriented:

```text
thread/start or thread/resume -> turn/start -> stream events
```

This is the right layer when building a custom client or durable integration.

Tradeoff observed in Desktop workflows:

- The turn may be persisted.
- A custom client may see it.
- The already-open Codex Desktop UI may not immediately live-refresh to show it.

## Goal 2: Put Text Into The Visible Desktop Chat

Use CDP against the Codex Desktop shell when the user is already looking at Codex Desktop and expects the message to appear there now.

The CDP path is UI-oriented:

```text
visible Desktop shell target -> bottom Codex composer -> insert text -> submit
```

This is the right layer when the UX depends on continuity in the current visible chat.

For a robust app, combine CDP with thread targeting:

```text
known thread id -> codex://threads/<thread-id> -> shell CDP target -> visible composer -> Input.insertText
```

OpenCLI is useful for smoke tests, but generated apps should not use `opencli codex send` as the app's primary send mechanism. If the in-app browser textarea has focus, a generic active-composer adapter can paste into the local app instead of the Codex chat. The robust path is to connect to the `app://` Codex shell target and operate the bottom visible composer there.

Tradeoffs:

- It depends on Codex Desktop UI structure.
- It can break if the UI structure changes.
- It requires launching Codex Desktop with a remote-debugging port.
- It is local-machine automation, not a server API.

## Practical Rule

Use App Server for durable protocol work.

Use direct shell-target CDP for visible Desktop handoff.

For many prototypes, the strongest pattern is hybrid:

1. Save raw local context yourself.
2. Carry a target Codex thread id in the local payload.
3. Focus the target thread with `codex://threads/<thread-id>` when possible.
4. Use direct shell-target CDP to send the message into the visible chat.
5. Let Codex perform the user-approved durable write.

This keeps the UX fast while avoiding hidden writes to important knowledge files.
