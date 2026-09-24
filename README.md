# ECAssistant

> **The small, open-source .NET agent library.** Add intelligent, tool-using agents to any application — in a few lines of code.
>
> We named it ECAssistant because we believe AI is there to **assist** people — enhance your productivity, not automate you out of the loop. ECAssistant brings that assistance *into your applications*: it works alongside you and your users, not headless in the dark.

Your models. Your keys. Your machine. Local GGUF or any OpenAI-compatible endpoint — same code, one config change. No telemetry. MIT.

[![NuGet](https://img.shields.io/nuget/v/ECAssistant.Core?label=ECAssistant.Core)](https://www.nuget.org/packages/ECAssistant.Core)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

![ECAssistant — real terminal session: install, launch, ask, answer](demo.gif)

## Why

Most agent frameworks assume you'll ship your data to someone else's API — or lock you into one provider. ECAssistant is built the other way around:

- 🧩 **Embeddable by design** — one NuGet package. Console app, desktop app, web backend: same library, same API. Give *your* application intelligence.
- 🧠 **Bring your own brain** — local GGUF models or any OpenAI-compatible endpoint. Switch by config, not code.
- 🔧 **11 built-in tools** — files, shell, git, dotnet, code editing, sub-agents, vision structure. Permission-gated (approve / always / never per tool). Add your own via one interface.
- 🪶 **Lightweight** — .NET 8, HTTP-based inference, zero native dependencies in your project, zero embedded blobs. Our CLI is 2.8 MB.
- 🔒 **Private by default** — everything runs on your machine. Nothing phones home.
- 🤝 **Assist-first** — the agent is a colleague, not a daemon: interactive, permission-gated, always showing its work. You stay in control.
- 👁️ **Vision structure extraction** — screenshots, UI mockups, and scanned PDFs in; a fixed, versioned JSON schema out (elements, bounding boxes, label↔control associations, semantic groups). Grammar-enforced at the sampler level — the shape is physically guaranteed, not hoped for.
- 🎯 **Grammar-forced reliability** — agent decisions and tool calls are token-level constrained (GBNF): valid JSON with typed parameters on even a 4B local model.

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

// 3. Run the agent — the orchestrator plans, calls tools, and returns when done
var orchestrator = session.Orchestrator;
var result = await orchestrator.ExecuteMultiStep("Summarize the docs in this folder");
Console.WriteLine(result.FinalOutput);
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

- **[Getting started](docs/getting-started.md)** — install, configure, first conversation
- **[Agent lifecycle](docs/agent-lifecycle.md)** — composition root → session → orchestrator
- **[Embedding the agent](docs/embedding-guide.md)** — add ECAssistant to *your* app, step by step (all hosting options)
- **[Custom tools](docs/custom-tools.md)** — teach the agent new capabilities in one file (schema, permissions, projections, dataflow)
- **[Testing your agent](docs/testing.md)** — three levels: MockEngine → user-experience E2E → real-model journey tests (both tiers)
- **[Tools & permissions](docs/tools-and-permissions.md)** — the 11 built-ins, writing custom tools
- **[Local vs Remote](docs/local-vs-remote.md)** — GGUF vs OpenAI-compatible, switching
- **[Security & Privacy](docs/security-and-privacy.md)** — what runs where, what leaves your machine

**For AI agents & LLM tools:** [AI-NAVIGATION.md](AI-NAVIGATION.md) — machine-readable map of all six repos, dependency rules, bounded-context protocol, release law. Every repo also ships its own `AGENTS.md` plus `ARCHITECTURE.md` / `API-INDEX.md` / `RELATIONSHIP-GRAPH.md` (LDC artifacts, kept in sync, enforcement-checked).

## Contributing

Issues and PRs welcome — start with the [Core architecture](https://github.com/SideDevEC/ECAssistantCore/blob/main/ARCHITECTURE.md).

## License

MIT — see [LICENSE](LICENSE).
