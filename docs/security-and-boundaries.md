# Security And Boundaries

> ⚠️ **Experimental only**
>
> This technique gives a local process the ability to operate your visible Codex Desktop UI. Treat it with care.
>
> This is unstable and experimental. It is not recommended for production apps, unattended agents, multi-user services, or any workflow that needs a stable integration contract.

## Safe Defaults

- Bind CDP to localhost only: `127.0.0.1`.
- Use a fixed local port such as `9222`.
- Do not expose the port through ngrok, a tunnel, Docker port forwarding, or a public interface.
- Do not accept arbitrary remote text and pass it straight into the Codex composer.
- Keep user-specific paths and secrets in ignored local config files.
- Keep private Codex thread ids in ignored local config files, not public docs or committed examples.
- Keep raw artifacts local unless the user explicitly chooses to share them.

## Recommended App Boundary

A local app should:

1. accept input only from the local user,
2. save any important raw context first,
3. generate a bounded prompt,
4. call the Codex Desktop CDP endpoint from a backend process,
5. display the result or error,
6. keep project/wiki writes behind approval unless the user opts in.

## Do Not Document Or Build

Do not use this pattern to:

- bypass authentication,
- control someone else's Codex Desktop,
- scrape private conversations without consent,
- modify Codex Desktop internals,
- open the CDP port to a network,
- claim this is an official supported plugin API.

## Threat Model

If a malicious local process can reach your CDP endpoint, it may be able to inspect or operate the Electron UI.

That is why the bridge should be used only on trusted machines and only with trusted local tools.

When you are done using the bridge, quit ChatGPT and relaunch it normally.
