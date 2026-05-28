# Build A Native-Like Codex App

Copy this prompt into Codex from the project folder where you want the app to live.

````text
I want to build a small local app that feels native inside Codex Desktop.

Use this pattern:

local web app -> local backend -> Codex Desktop CDP -> visible Codex chat

The app should run in the Codex in-app browser and send structured context into the visible Codex conversation.

This app should be useful inside a project, knowledge base, or Second Brain where Codex has local instructions and skills. The app should classify the user's action and include a skill hint when relevant.

Create the smallest working app in this project that:
1. starts with one local dev command,
2. opens in the browser,
3. shows a small context panel with the backend working directory,
4. shows whether `AGENTS.md` exists in that working directory,
5. discovers local skills from `.agents/skills/*/SKILL.md` and `.codex/skills/*/SKILL.md` when those folders exist,
6. displays discovered skills in the UI,
7. has one textarea,
8. has one action selector,
9. has one skill selector populated from discovered skills,
10. has one optional manual skill hint field,
11. has one optional target thread id field or supports `?threadId=...`,
12. has one "Send to Codex" button,
13. sends the textarea content, action, selected skill, manual skill hint, working directory, available skills, and thread id to a local backend route,
14. focuses the target Codex thread before sending when a thread id is configured,
15. has the backend send through direct Codex Desktop CDP composer insertion,
16. creates a repo-local Codex skill that only starts and opens this app,
17. includes a short README section explaining setup and run steps.

Add a minimal context route:
- `GET /api/context`
- returns `{ cwd, hasAgentsMd, skills, targetThreadId }`
- `cwd` should be the backend process working directory unless a local env override is provided
- `skills` should be a best-effort list of local skills with `{ name, description, path, hint }`
- parse `name` and `description` from SKILL.md frontmatter when available
- `hint` can be the folder name

Add a minimal local skill:
- create `.agents/skills/start-my-codex-app/SKILL.md`
- the skill should have valid YAML frontmatter with `name` and `description`
- the skill body should be concise
- it should include the dev command for this app
- it should include the local URL
- it should require or configure the target Codex thread id for the app session
- it should pass the target thread id to the app as `CODEX_TARGET_THREAD_ID` or `?threadId=...`
- when triggered, it should start the dev server and open the local URL in the Codex in-app browser when browser tooling is available
- it should not explain the UI controls or encode app-specific behavior, because the user may change the app over time

Assume Codex Desktop is launched with:

/Applications/Codex.app/Contents/MacOS/Codex --remote-debugging-port=9222

Assume this is running on macOS. This repo has only tested the Codex Desktop CDP bridge on macOS.

Recommend that the user runs the receiving Codex thread with:

GPT-5.5 Medium

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

The launcher skill is responsible for setting the target thread id for the app session when the user wants reliable thread targeting.

The message sent to Codex should be structured like this:

Local Codex app handoff.

Action:
- {{ACTION}}
- Skill hint: {{SKILL_HINT}}

Context:
- Project: current project
- Working directory: {{WORKING_DIRECTORY}}
- App: minimal native-like Codex app
- Knowledge base: current project or configured knowledge root
- Target thread id: {{THREAD_ID}}
- Available local skills: {{AVAILABLE_SKILLS}}
- Selected local skill: {{SELECTED_SKILL}}

User input:
```text
{{USER_INPUT}}
```

Instructions:
1. Treat this as context from my local app.
2. Classify the request and use the relevant local Codex skill if one applies.
3. Do not edit files unless I explicitly ask.
4. End with the smallest useful next action.

Keep the implementation minimal. Do not build a framework, auth, database, or deployment flow. The only goal is proving that a local app can send structured context into the visible Codex chat.
````
