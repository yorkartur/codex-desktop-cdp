# Manual

This manual explains how to make a local tool send a message into the visible Codex Desktop chat.

The important distinction:

- If you need durable thread state, use Codex App Server or SDK-style APIs.
- If you need the message to appear in the currently visible Desktop composer, use direct CDP composer insertion against the Codex Desktop shell page.
- If you need reliability, carry the target Codex thread id through your app even when the final send uses the visible composer.

Tested setup:

- macOS only so far.
- Codex Desktop receiving thread set to `GPT-5.5 Medium`.
- Codex Desktop launched from the macOS app bundle with `--remote-debugging-port=9222`.

## 1. Optional OpenCLI Smoke Test

OpenCLI is useful for quick status checks and manual smoke tests.

```sh
npm install -g @jackwener/opencli
```

Check that it is available:

```sh
opencli --version
```

## 2. Relaunch Codex Desktop With CDP Enabled

Quit Codex Desktop first. Then start it from a terminal:

```sh
/Applications/Codex.app/Contents/MacOS/Codex --remote-debugging-port=9222
```

Keep this process running. If you open Codex normally from the dock, it may not expose the debug port.

This direct app-bundle command is the macOS-tested path. It may be brittle across Codex Desktop or macOS installs, so verify the CDP endpoint before debugging the app.

## 3. Verify The CDP Endpoint

Check the low-level endpoint:

```sh
curl http://127.0.0.1:9222/json/version
```

If this fails, Codex Desktop is not listening on that port.

Then point OpenCLI at the same endpoint:

```sh
export OPENCLI_CDP_ENDPOINT="http://127.0.0.1:9222"
```

Verify OpenCLI can see Codex:

```sh
opencli codex status
```

## 4. Pick Or Create A Target Thread

For quick manual testing, open the Codex conversation you want to target and keep it visible.

For a real local app, store a target thread id in local config or create one through App Server. The thread id matters because it lets you:

- focus the intended Desktop conversation,
- resume the same conversation through CLI/App Server paths,
- write the thread id into local raw artifacts for traceability,
- avoid accidentally sending to whatever thread happens to be focused.

On macOS, a known thread can be focused with a deep link:

```sh
open "codex://threads/<thread-id>"
```

The deep link is how you make the right composer visible before sending, whether you are running a smoke test or using the direct CDP app path.

## 5. Send A Smoke-Test Message

Open or focus the Codex Desktop conversation you want to receive the message.

Then run:

```sh
opencli codex send "hi from OpenCLI"
```

Expected result:

- OpenCLI finds the active Codex Desktop window.
- It finds the visible thread composer.
- It injects the message.
- It submits the message.
- The message appears in the visible conversation.

If this pastes into your local app's textarea, do not copy that behavior into your app. The app backend should use the direct CDP shell-target pattern below.

## 6. Integrate From A Local App

Do not call OpenCLI directly from a browser frontend. Call it from a trusted local backend process.

Critical send-path rule:
Do not use `opencli codex send` as the app's primary send mechanism when the app runs inside the Codex in-app browser.

Reason:
The local app's textarea may be the focused element, so `opencli codex send` can paste the prompt back into the app instead of the Codex chat composer.

Required implementation:

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

Recommended local app flow:

1. Capture user input in your app.
2. Build a concise prompt.
3. Save any raw artifact locally first if traceability matters.
4. Include `threadId` in the payload when you have one.
5. Focus the target thread with `codex://threads/<thread-id>`.
6. Backend connects to the Codex shell CDP target.
7. Backend inserts into the visible Codex composer and submits.
8. User continues inside the visible Codex Desktop chat.

## 7. Prompt Shape

Keep the message small enough to be useful in a chat composer.

Good structure:

```markdown
Context:
- Target thread id:
- Source:
- Locator:
- Raw artifact:
- Intended action:

User note:
...

Please respond with:
1. Short acknowledgement.
2. Useful interpretation.
3. Recommended next action.

Do not edit durable files unless I explicitly approve.
```

For a knowledge-base workflow, include:

- Codex thread id,
- knowledge root path,
- raw artifact path,
- durable note or artifact index path,
- target project or topic,
- write policy.

## 8. Operating Pattern

This works best when the local tool and Codex Desktop are both visible.

Example layout:

```text
Codex Desktop chat | local app in Codex browser
```

The user performs the lightweight action in the local app, then continues the reasoning in Codex.

## 9. Reset

When done, quit Codex Desktop and reopen it normally if you do not want a CDP port open:

```sh
open -a Codex
```

Or relaunch again with the debug port when you need the bridge.
