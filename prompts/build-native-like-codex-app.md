# Build A Native-Like Codex App

Copy this prompt into Codex from the project folder where you want the app to live.

````text
Use this repository as the pattern:
https://github.com/yorkartur/codex-desktop-cdp

Create the smallest local web app in this project that proves this flow:

local web app -> local backend -> ChatGPT app CDP -> visible Codex chat

Requirements:
- runs in the Codex in-app browser
- shows backend cwd and whether `AGENTS.md` exists
- discovers local skills from `.agents/skills/*/SKILL.md` and `.codex/skills/*/SKILL.md`
- has one textarea, action selector, skill selector, optional manual skill hint, and Send button
- sends a structured prompt to Codex through a backend route
- creates a repo-local launcher skill at `.agents/skills/start-my-codex-app/SKILL.md`
- documents how to run it

Assume:
- macOS only tested
- ChatGPT desktop app launched with:
  `/Applications/ChatGPT.app/Contents/MacOS/ChatGPT --remote-debugging-port=9222`
- receiving Codex thread uses `GPT-5.5 Medium`

Critical send-path rule:
Do not use `opencli codex send` as the app's primary send mechanism.

Reason:
When the app runs inside the Codex in-app browser, the app textarea may be focused, so OpenCLI can paste back into the app instead of the Codex composer.

Required send path:
- use OpenCLI only as a smoke test
- connect directly to `CODEX_CDP_ENDPOINT` or `OPENCLI_CDP_ENDPOINT`, default `http://127.0.0.1:9222`
- fetch `${endpoint}/json/list`
- select the Codex shell target: `type === "page"`, `url.startsWith("app://")`, and `webSocketDebuggerUrl`
- never evaluate against the local app URL target
- if `CODEX_TARGET_THREAD_ID` or `?threadId=...` exists, first run:
  `open "codex://threads/<thread-id>"`
- focus the visible bottom Codex composer
- insert the structured prompt with `Input.insertText`
- verify composer text exists
- submit with Enter via `Input.dispatchKeyEvent`
- if no thread id exists, warn that it sends to the currently visible Codex composer

Keep it minimal. No framework, auth, database, or production hardening.
````
