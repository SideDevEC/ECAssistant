# Security & Privacy

ECAssistant is built to **assist** — and assistance you can't trust is worthless. This page is the straight answer to "what runs where, and what leaves my machine?"

## The short version

- **Local mode:** nothing leaves your machine. Inference, embeddings, memory — all on `localhost`.
- **Remote mode:** only your chat requests go to the endpoint you configured. Nothing else, no telemetry.
- **Either way:** ECAssistant itself phones home to no one. No analytics, no usage tracking, no license checks.

## Tool permissions

Agent actions are the main privacy surface — an agent that can run shell commands is powerful, so it's controlled:

- Every tool call is gated: **approve / always / never**, per tool
- Sensitive tools (shell, git, dotnet) default to **approve** — you see exactly what will run
- Every call, input, and result is rendered inline — the agent never works "headless in the dark"

## What's on disk

| Path | What |
|---|---|
| `~/.ECAssistantLLM/` | LLM server binaries, models, server config (local mode) |
| app config dir | `appsettings.json` and your agent's memory files |

Model files are SHA-256 verified against pinned checksums during install. After setup, chat time involves zero downloads.

## Open source, MIT

Everything is inspectable: [ECAssistantCore](https://github.com/SideDevEC/ECAssistantCore), [ECAssistantLLM](https://github.com/SideDevEC/ECAssistantLLM), [ECAssistantTUI](https://github.com/SideDevEC/ECAssistantTUI), [ECAssistantConsole](https://github.com/SideDevEC/ECAssistantConsole). Read the code that runs your conversations.

## Where to go next

- [Local vs Remote](local-vs-remote.md)
- [Tools & permissions](tools-and-permissions.md)
