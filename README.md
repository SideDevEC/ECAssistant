# ECAssistant

> **The small, open-source .NET agent library.** Add intelligent, tool-using agents to any application — in a few lines of code.

Your models. Your keys. Your machine. Local GGUF or any OpenAI-compatible endpoint — same code, one config change. No telemetry. MIT.

[![NuGet](https://img.shields.io/nuget/v/ECAssistant.Core?label=ECAssistant.Core)](https://www.nuget.org/packages/ECAssistant.Core)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

<!-- TODO: hero GIF here before launch — 10–20s TUI screencast, no sound -->

## Why

Most agent frameworks assume you'll ship your data to someone else's API — or lock you into one provider. ECAssistant is built the other way around:

- 🧩 **Embeddable by design** — one NuGet package. Console app, desktop app, web backend: same library, same API.
- 🧠 **Bring your own brain** — local GGUF models or any OpenAI-compatible endpoint. Switch by config, not code.
- 🔧 **12 built-in tools** — files, shell, git, dotnet, web, code editing, sub-agents. Permission-gated (approve / always / never per tool). Add your own via one interface.
- 🪶 **Lightweight** — .NET 8, HTTP-based inference, zero native dependencies in your project, zero embedded blobs. Our CLI is 2.8 MB.
- 🔒 **Private by default** — everything runs on your machine. Nothing phones home.

## Quick Start — build an agent into your app

```bash
dotnet add package ECAssistant.Core
```

```csharp
using ECAssistant.Core.Composition;
using ECAssistant.Core.Engine;
using ECAssistant.Core.Session;

// 1. One call wires config, model resolution, tools, memory and inference
var root = new EcaCompositionRoot(userConfigDir, args);
var services = root.Build();

// 2. Create a session and register an output listener (streamed tokens + tool events)
var sessions = new SessionManager(services.Config, services.ModelPath, workingDir, services.Logger);
var session = sessions.CreateSession("main");
session.AddListener(myListener);          // implements IOutputListener

await services.SessionBuilder.BuildAsync(session, externalTools: null);

// 3. Run the agent — it plans, calls tools, and streams its answer
var loop = new EDecisionLoop(session.Engine, session);
await loop.ExecuteInteractiveLoop("Summarize the docs in this folder");
```

**A complete, runnable wiring example lives in [ECAssistantConsole](https://github.com/SideDevEC/ECAssistantConsole) — the full host in ~100 lines.**

## Quick Start — just use it in the terminal

```bash
dotnet tool install -g ECAssistant.Console
ecassistant
```

First run asks local or remote, downloads only what you pick, and you're chatting — with the agent able to read files, run shell commands, and search the web on your behalf.

## The ecosystem

| Repo | What it is | Use it when… |
|---|---|---|
| **[ECAssistantCore](https://github.com/SideDevEC/ECAssistantCore)** | The agent library | You're building an intelligent application |
| **[ECAssistantLLM](https://github.com/SideDevEC/ECAssistantLLM)** | Self-contained local inference server (GGUF) | You want local models — also usable standalone |
| **[ECAssistantTUI](https://github.com/SideDevEC/ECAssistantTUI)** | Reusable terminal UI layer | You're building your own host |
| **[ECAssistantConsole](https://github.com/SideDevEC/ECAssistantConsole)** | Reference host / end-user CLI | You just want a working agent today |

## Documentation

- **Getting started** — install, configure, first conversation
- **Agent lifecycle** — composition root → session → decision loop
- **Tools & permissions** — the 12 built-ins, writing custom tools
- **Local vs Remote** — GGUF vs OpenAI-compatible, switching
- **Security & Privacy** — what runs where, what leaves your machine

*(docs land with the first public release wave — each repo ships its own ARCHITECTURE.md in the meantime)*

## Contributing

Issues and PRs welcome — start with the [Core architecture](https://github.com/SideDevEC/ECAssistantCore/blob/main/ARCHITECTURE.md).

## License

MIT — see [LICENSE](LICENSE).
