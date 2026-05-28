# Use Cases

The point is not "send text to a chatbot."

The point is to create **Codex-native local apps**: small local interfaces that are worth opening inside the Codex environment because Codex can immediately use the conversation, project context, filesystem, terminal, browser, skills, approvals, automations, and Computer Use-style UI control when available.

This is the frame: Codex Desktop is not merely where the prompt appears. It is the action layer. After the prompt lands, Codex can inspect files, run code, calculate, automate Chrome, operate a browser, create new code, edit local artifacts, call skills, ask for approval, and keep the conversation alive.

The shape:

```text
local UI captures the moment -> backend inserts it into visible Codex chat through CDP -> Codex reasons, uses tools, edits files, schedules work, or asks for approval
```

Use this pattern when the local UI is better at interaction, but Codex Desktop is better as the action environment.

## What Makes A Use Case Codex-Native?

A good use case should use at least one of these Codex Desktop advantages:

- **Visible conversation continuity**: the user sees the prompt land in the active chat and can keep talking.
- **Project context**: Codex already knows the repo, files, docs, and current branch.
- **Skills**: a prompt can invoke a repeatable workflow, not just a generic answer.
- **Filesystem writes**: Codex can create or edit local files with review.
- **Terminal and tests**: Codex can run scripts, games, builds, tests, and diagnostics.
- **Code execution and calculation**: Codex can use scripts and local runtimes to compute, transform, simulate, or validate.
- **In-app browser and Chrome automation**: the user can interact with a local web UI beside the chat, and Codex can automate browser workflows when appropriate.
- **Approvals**: dangerous or durable actions can pause for explicit user confirmation.
- **Automations**: Codex can schedule follow-ups or recurring checks when supported.
- **Computer Use / UI control**: for workflows that need actual desktop interaction, not just text.
- **Multi-thread work**: a local app can spawn or guide separate lines of work in Codex.

If a use case only needs a stateless chatbot response, this pattern is probably overkill.

## Games

1. **Codex Dungeon Master**
   A tabletop RPG UI runs in the Codex browser with map, inventory, dice, and party state. Sending an action to Codex is valuable because Codex becomes the visible DM, writes campaign notes, tracks continuity, generates encounters, and can schedule reminders for unresolved quests.

2. **Codebase Escape Room**
   A puzzle game built around a real repository. Each puzzle is a failing test, hidden bug, or architectural clue. Codex is worth using because it can inspect files, run tests, explain hints, and apply fixes when the player asks.

3. **Game Jam Director**
   A local game prototype sends player telemetry, notes, and current files to Codex. Codex proposes mechanics, edits the code, generates assets through skills, runs the build, and hands changes back for review.

4. **AI Referee Sandbox**
   A simulation or multiplayer game sends disputed turns to Codex. Codex acts as referee, checks rules, updates a match log, and can use files as persistent game state.

5. **Procedural Quest Workshop**
   A quest-board UI sends world constraints to Codex. Codex writes quest JSON, dialogue files, item tables, and TODOs directly into the project.

## Creative Tools

6. **Worldbuilding Studio**
   A canvas of characters, factions, maps, and timelines hands selected context to Codex. Codex is valuable because it can maintain lore files, find contradictions, write scenes, and update project docs.

7. **Design Critic Console**
   A screenshot annotation UI sends marked regions to Codex. Codex can inspect the frontend repo, edit components/CSS, run the app, and use the in-app browser for verification.

8. **Video Review Director**
   A timeline UI captures timestamped notes. Codex turns them into edit scripts, captions, launch clips, or Remotion changes, then runs local render commands with approval.

9. **Asset Generation Board**
   A moodboard UI sends selected references and desired style to Codex. Codex can invoke image-generation skills, save assets into the repo, and update usage docs.

10. **Prompt Lab With Execution**
   A prompt-testing UI sends variants to Codex. Codex can turn the winning prompt into a skill, update tests, create fixtures, or run eval scripts.

## Developer Workflows

11. **Skill Builder Studio**
   A structured UI for skill name, trigger, instructions, scripts, and examples. Codex is worth using because it can create the skill files, install dependencies, test scripts, and register the workflow.

12. **Frontend QA Console**
   A checklist UI for responsive states, accessibility, screenshots, and copy issues. Codex can inspect the repo, patch UI code, run builds, and verify in the browser.

13. **AI Pair-Programming HUD**
   A tiny local panel captures objective, files, tests, and acceptance criteria. Codex receives a precise implementation task and can edit files, run tests, and show diffs in the Desktop thread.

14. **Repo Onboarding Quest**
   A guided app asks the user to explore codebase areas. Codex can answer with actual file citations, generate architecture maps, and save onboarding notes.

15. **Agent Control Panel**
   A dashboard of scripts, logs, deployments, and test buttons. Pressing a button sends Codex the current operational context so it can run commands with approval and explain results.

## Knowledge And Operations

16. **Second Brain Garden**
   A graph UI for concepts, projects, decisions, and sources. Codex matters because it can read the vault, update Markdown pages, preserve raw logs, and ask before durable project writes.

17. **Decision Board**
   A decision-log UI sends one card into Codex. Codex can inspect project files, argue tradeoffs, write ADRs, update docs, or schedule a follow-up check.

18. **Research Cockpit**
   A local app captures claims, links, screenshots, and questions. Codex can browse when needed, cite sources, write memos, and file them into a local research folder.

19. **Personal CRM Agent**
   A relationship dashboard sends a contact or company context to Codex. Codex can draft follow-ups, save notes, create reminders/automations, or prepare meeting briefs.

20. **Daily Operator Console**
   A local command center for tasks, inbox, calendar notes, project status, and scripts. Codex can triage, run local commands, schedule future work, and escalate actions for approval.

## The Strong Rule

Build these when Codex Desktop adds a real advantage.

Bad fit:

```text
button -> generic chatbot answer
```

Good fit:

```text
interactive local state -> visible Codex conversation -> skills/files/terminal/browser/Chrome/Computer Use/code/approvals/automations
```

The local app should make the interaction rich. Codex should make the moment actionable.
