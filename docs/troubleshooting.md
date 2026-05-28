# Troubleshooting

## `CDP not reachable`

Codex Desktop was probably launched normally, the port does not match, or the macOS app-bundle launch path is different on your machine.

This repo has only been tested on macOS with Codex Desktop launched through the app-bundle executable.

Fix:

```sh
/Applications/Codex.app/Contents/MacOS/Codex --remote-debugging-port=9222
export OPENCLI_CDP_ENDPOINT="http://127.0.0.1:9222"
opencli codex status
```

Also check:

```sh
curl http://127.0.0.1:9222/json/version
```

## OpenCLI Connects But No Message Appears

Check:

- Is Codex Desktop visible?
- Is a conversation open?
- Is the composer enabled?
- Is another modal or review panel focused?
- Did OpenCLI target a different Codex window?

Try focusing the desired conversation manually, then run:

```sh
opencli codex send "visible chat smoke test"
```

For generated apps, this command is only a smoke test. The app backend should use direct CDP against the Codex shell target where `url.startsWith("app://")`.

## The Message Appears In The Local App Textarea

This means the send path targeted the active/focused element instead of the Codex chat composer.

Fix:

- Do not use `document.activeElement`.
- Do not evaluate against the local app target such as `http://127.0.0.1:<port>`.
- Fetch `http://127.0.0.1:9222/json/list` and connect to the Codex shell target where `url.startsWith("app://")`.
- In that shell page, find the bottom visible `.ProseMirror[contenteditable="true"]` or `.ProseMirror` composer.
- Insert with `Input.insertText`, verify insertion, then dispatch Enter.

## The Message Appears In The Wrong Conversation

For smoke tests, `opencli codex send` acts on the active visible context unless you use selection flags supported by the adapter. Generated apps should avoid this ambiguity by using the direct shell-target CDP path.

Open the desired conversation first. If you know the thread id, focus it before sending:

```sh
open "codex://threads/<thread-id>"
opencli codex send "visible chat smoke test"
```

Then inspect available OpenCLI Codex options:

```sh
opencli codex --help
opencli codex projects
opencli codex history
```

## Missing Thread Id

You can still run a smoke test without a thread id, but you cannot reliably target a specific conversation unless it is already focused.

For a real app, store the target thread id in local config or get it from a thread-creation flow. Use it to:

- focus `codex://threads/<thread-id>`,
- resume existing work through CLI/App Server paths,
- include `threadId` in raw artifacts and prompts,
- debug which conversation received the handoff.

## New Conversations Are Slow To Appear

Creating or persisting thread state is not the same as live-refreshing the Desktop sidebar.

If your goal is immediate visible UX, prefer sending to the already-open visible conversation.

## App Server Turn Exists But Desktop Does Not Refresh

That is the core reason this manual exists.

App Server can be correct at the protocol layer while Desktop does not immediately show the new turn in the visible UI. Use direct shell-target CDP when the UX requires the visible composer.

## OpenCLI Breaks After A Codex Update

This is expected risk for UI automation.

Try:

```sh
npm update -g @jackwener/opencli
opencli codex status
opencli codex dump
```

Then check whether the OpenCLI Codex adapter has updated.
