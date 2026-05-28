# Thread Targeting

The visible-chat handoff needs two concepts:

1. **Direct shell-target CDP** to operate the visible Codex Desktop composer.
2. **A target thread id** to make sure the visible composer belongs to the right conversation.

OpenCLI can send a smoke-test message without a thread id:

```sh
opencli codex send "hello"
```

But that sends to whatever Codex Desktop currently exposes as the active visible composer. For a real app, that is too loose and can paste into the local app if the in-app browser input has focus. The app should know or create a target thread id, focus it first, then send through direct CDP against the Codex shell target.

## Why The Thread Id Matters

The thread id lets a local app:

- focus an existing Codex Desktop conversation,
- resume a specific conversation through CLI or App Server flows,
- preserve traceability in raw artifacts,
- include the intended target in the prompt payload,
- avoid sending a prompt to the wrong visible chat.

## Focus A Known Thread

On macOS:

```sh
open "codex://threads/<thread-id>"
```

In Node:

```js
import { spawn } from "node:child_process";

export function focusCodexThread(threadId) {
  const child = spawn("open", [`codex://threads/${encodeURIComponent(threadId)}`], {
    detached: true,
    stdio: "ignore",
  });
  child.unref();
}
```

Then send through direct CDP:

```text
connect to app:// shell target -> focus bottom composer -> Input.insertText -> Enter
```

## Resume A Known Thread Through Codex CLI

For workflows that intentionally continue an existing Codex thread from the CLI, the thread id is required:

```sh
codex exec resume <thread-id> "continue from this local-app handoff"
```

This is different from visible CDP handoff. CLI resume is thread-addressed. Direct CDP handoff is visible-composer addressed.

## Create A Thread Through App Server

An App Server style flow can create a thread and return an id:

```text
thread/start -> thread id -> thread/name/set -> turn/start
```

Once a thread id exists, a Desktop workflow can attempt to focus it:

```sh
open "codex://threads/<thread-id>"
```

Then the visible handoff can use direct shell-target CDP.

## Store It Locally

Do not hardcode private thread ids in public docs or committed config.

Use ignored local config, for example:

```env
CODEX_TARGET_THREAD_ID=
CODEX_CDP_ENDPOINT=http://127.0.0.1:9222
```

Also include the thread id in local raw artifacts when traceability matters:

```yaml
codex_thread_id: <thread-id>
```

## Practical Rule

Use this rule:

```text
If the app cares which conversation receives the message, require a thread id.
```

Without a thread id, the app is only sending to "whatever Codex chat is visible right now."
