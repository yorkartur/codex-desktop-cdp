# Core Backend Call

Your browser UI should not spawn local processes directly. Use a local backend.

## Critical Send-Path Rule

Do not use `opencli codex send` as the app's primary send mechanism when the app runs inside the Codex in-app browser.

Reason:

The local app's textarea may be the focused element, so `opencli codex send` can paste the prompt back into the app instead of the Codex chat composer.

## Required Implementation

- Use OpenCLI only as an optional smoke test.
- For the real app handoff, connect directly to the Codex Desktop CDP endpoint:
  `OPENCLI_CDP_ENDPOINT=http://127.0.0.1:9222`
  or
  `CODEX_CDP_ENDPOINT=http://127.0.0.1:9222`
- Fetch `${endpoint}/json/list`.
- Select the Codex Desktop shell target where:
  - `target.type === "page"`
  - `target.url.startsWith("app://")`
  - `target.webSocketDebuggerUrl` exists
- Never evaluate against the local app URL target.
- If `CODEX_TARGET_THREAD_ID` or `?threadId=...` is configured, first focus:
  `open "codex://threads/<thread-id>"`
- Through CDP, focus the visible bottom Codex composer, usually a `.ProseMirror[contenteditable="true"]` near the bottom of the window.
- Insert the structured prompt using `Input.insertText`.
- Verify the composer contains text.
- Submit with `Input.dispatchKeyEvent` for Enter.
- If no thread id is configured, clearly warn that the app will send to the currently visible Codex composer.

This is UI automation, not a stable API contract.
