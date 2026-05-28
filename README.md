# Codex Desktop CDP

Build local apps that feel native inside Codex Desktop.

The trick is simple:

```text
local web app -> local backend -> Codex Desktop CDP -> visible Codex chat
```

Your app runs in the Codex in-app browser. When the user clicks a button, selects text, or triggers an action, your local backend sends a structured prompt into the visible Codex conversation. From there Codex can use the current project, files, terminal, browser tools, approvals, and skills.

This is not an official Codex plugin API. It is an unstable, experimental local bridge for building your own native-like Codex workflows.

Do not use this for production apps. The bridge depends on local CDP access and Codex Desktop UI structure, so it can break when Codex Desktop, OpenCLI, Electron, or macOS behavior changes.

## Tested Setup

This repo has only been tested on macOS with Codex Desktop.

Recommended Codex thread setting:

- `GPT-5.5 Medium`

The examples use the macOS app-bundle executable:

```sh
/Applications/Codex.app/Contents/MacOS/Codex --remote-debugging-port=9222
```

That launch path can be brittle across Codex Desktop or macOS installs. If it fails, quit Codex, relaunch with the command above, then verify the CDP endpoint before debugging your app.

## Recommended Setup

This pattern is strongest when the app lives next to a project, knowledge base, or Second Brain that Codex already understands.

The best setup has:

- local files Codex can read,
- an `AGENTS.md` or project instructions,
- local skills or repeatable workflows,
- a small local skill that starts the app,
- a place to preserve raw handoffs,
- a clear write policy for durable changes.

The app should not only send text. It should show the user which directory it is running from, list the local skills it can find, classify the user's action, and send enough context for Codex to choose or invoke the right skill.

## Why This Is Useful

Chat is not always the best UI.

Sometimes you want a tiny local interface for a specific workflow:

- read a paper and send selected passages to Codex,
- edit Markdown and run writing skills on selected text,
- build a project dashboard that can ask Codex to run commands,
- create a skill launcher for repeated project actions,
- connect a local knowledge base or Second Brain to Codex.

The app handles the interface. Codex handles the reasoning and actions.

## Quick Start

Optional OpenCLI smoke test:

```sh
npm install -g @jackwener/opencli
```

Quit Codex Desktop, then relaunch it with a local CDP port:

```sh
/Applications/Codex.app/Contents/MacOS/Codex --remote-debugging-port=9222
```

This command is the macOS-tested path. Other platforms have not been tested in this repo.

Point OpenCLI at Codex:

```sh
export OPENCLI_CDP_ENDPOINT="http://127.0.0.1:9222"
opencli codex status
```

Send a smoke test:

```sh
opencli codex send "hi from my local app"
```

If the message appears in the visible Codex conversation, the CDP port and Desktop visibility are working. Generated apps should still use the direct CDP composer path documented below.

## Build Your First App

The fastest path is to let Codex generate the starter inside your own project folder.

From the project where you want the app to live, paste this into Codex:

```text
Use this repository as a pattern for building a native-like Codex Desktop app:
https://github.com/yorkartur/codex-desktop-cdp

Create a minimal local web app in this project that:
1. runs in the Codex in-app browser,
2. shows the current working directory from the backend,
3. shows whether `AGENTS.md` exists in that directory,
4. lists discovered local skills from `.agents/skills/*/SKILL.md` and `.codex/skills/*/SKILL.md` when present,
5. has one textarea, one action selector, a skill selector populated from discovered skills, an optional manual skill hint field, and one "Send to Codex" button,
6. has a local backend route that receives the text, action, selected skill, manual skill hint, working directory, and optional target thread id,
7. focuses the target Codex thread before sending when a thread id is configured,
8. sends a structured prompt into the visible Codex Desktop chat through direct CDP composer insertion,
9. creates a repo-local Codex skill that only starts and opens this app,
10. documents how to run it locally.

Use this handoff shape:

local web app -> local backend -> Codex Desktop CDP -> visible Codex chat

Assume Codex Desktop is launched with:
/Applications/Codex.app/Contents/MacOS/Codex --remote-debugging-port=9222

Assume the Codex thread is using `GPT-5.5 Medium`.

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

The app is meant to run inside a project, knowledge base, or Second Brain where Codex can use local instructions and skills.

Add a minimal context route:
- `GET /api/context` returns the backend working directory,
- whether `AGENTS.md` exists,
- discovered skills with `{ name, description, path, hint }`,
- the configured target thread id when present.

Add a minimal local skill:
- create `.agents/skills/start-my-codex-app/SKILL.md`,
- keep it concise,
- include the app dev command,
- include the local URL,
- require or configure the target Codex thread id for this app session,
- pass the target thread id to the app as `CODEX_TARGET_THREAD_ID` or `?threadId=...`,
- when used, the skill should start the app and open it in the Codex in-app browser when browser tooling is available.

Keep it minimal. Do not build a framework. I only want the smallest working app that proves the pattern.
```

There is also a copyable prompt here:

- [Build a native-like Codex app](prompts/build-native-like-codex-app.md)

## Thread Targeting

A raw `opencli codex send` sends to whatever composer the adapter considers visible. That is useful for a smoke test, but too loose for a real local app and can paste into the in-app browser textarea when your app has focus.

For reusable apps, the launcher skill should establish the target thread when the app starts. Treat that thread id as session configuration:

```text
start skill runs -> target thread id is configured -> app opens with that thread id -> every handoff targets that thread
```

For an app opened inside the Codex in-app browser, prefer an explicit target thread id:

```sh
open "codex://threads/<thread-id>"
```

Then have the backend connect to the `app://` Codex shell CDP target, insert into the bottom visible Codex composer, verify insertion, and submit.

Practical options:

- store `CODEX_TARGET_THREAD_ID` in ignored local env,
- accept `?threadId=<thread-id>` in the local app URL,
- show the configured thread id in the UI,
- warn when no thread id is configured.

Hard constraint:

```text
If no target thread id is configured, the app must clearly warn that it will send to the currently visible Codex composer.
```

The local app should not assume it can automatically discover the parent Codex thread from the in-app browser. If the thread matters, make it explicit in the launcher skill, pass it to the app, and focus it before sending.

## Add A Local Start Skill

The most useful generated apps include a repo-local launcher skill. That lets the user ask Codex to start the app later without remembering the commands.

Minimal shape:

```text
.agents/skills/start-my-codex-app/SKILL.md
```

The skill should do only two things:

1. Start the local dev server with a configured target thread id.
2. Open the app URL in the Codex in-app browser with that thread id.

The skill is where the target thread for the app session is established. It should pass that value to the app through local env or the URL:

```sh
CODEX_TARGET_THREAD_ID=<thread-id> npm run dev
```

or:

```text
http://127.0.0.1:<port>/?threadId=<thread-id>
```

Keep this skill local to the project. It should not explain the UI or encode product-specific behavior, because the user will keep evolving the app.

## The Core Backend Call

Your browser UI should not spawn local processes directly. Use a local backend.

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

## What To Send

Do not send random text. Send structured context.

Example:

````markdown
Local Codex app handoff.

Action:
- Review this note.
- Skill hint:

Context:
- Project:
- Working directory:
- Source:
- Current file:
- Knowledge base:
- Target thread id:
- Available local skills:
- Selected local skill:

User content:
```text
...
```

Instructions:
1. Treat this as context from my local app.
2. Classify the request and use the relevant local skill if one applies.
3. Ask before editing durable files.
4. End with the smallest useful next action.
````

That structure is what makes the app feel like part of Codex instead of a generic chatbot box.

## Example Patterns

### Reading Companion

Pattern: `source -> selected passage -> note -> Codex`

Use this when the user is reading a paper, book, article, or local document. The local app is good at selection and annotation. Codex is good at connecting the passage to a knowledge base, project, or skill.

### Writing Companion

Pattern: `draft -> selected text -> skill/action -> Codex`

Use this when the user is writing and wants to run a Codex skill against part of the draft: critique, rewrite, find weak arguments, preserve voice, or make a controlled edit.

## Safety

Keep this local.

- Do not expose the CDP port to a network.
- Do not control someone else's Codex app.
- Do not silently edit important files.
- Preserve raw handoff artifacts when the workflow matters.
- Ask for approval before durable write-back.

## More Notes

The old detailed notes are still useful if you want the deeper mechanics:

- [Manual](docs/manual.md)
- [Architecture](docs/architecture.md)
- [App Server vs CDP](docs/app-server-vs-cdp.md)
- [Thread Targeting](docs/thread-targeting.md)
- [Troubleshooting](docs/troubleshooting.md)

But the main idea is enough to start:

```text
build a tiny local UI for one workflow you already do in Codex
```
