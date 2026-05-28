# Sources

Last checked: 2026-05-23.

Primary references:

- OpenAI Codex App Server README: https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md
- OpenCLI repository: https://github.com/jackwener/opencli
- OpenCLI Codex adapter docs: https://opencli.info/docs/adapters/desktop/codex.html

Useful facts from these sources:

- Codex App Server exposes thread and turn operations such as `thread/start`, `thread/resume`, and `turn/start`.
- OpenCLI documents Desktop app control through CDP and includes Codex commands such as `opencli codex status` and `opencli codex send`.
- OpenCLI's Codex adapter requires Codex Desktop to be launched with `--remote-debugging-port=9222` or another configured CDP port.

Local observation from a local app to Codex Desktop prototype:

- Testing so far has been macOS-only.
- App Server style thread/turn writes can persist state but do not guarantee instant live-refresh in the already-open Codex Desktop UI.
- Direct CDP handoff is faster for the visible-chat experience because it operates the actual Desktop composer.
- `opencli codex send` is useful for smoke tests, but generated apps should target the `app://` Codex Desktop shell page directly, then use `Input.insertText` against the bottom visible composer. Thread ids are still necessary for reliable targeting, `codex://threads/<thread-id>` focus, CLI/App Server resume flows, and local traceability.
