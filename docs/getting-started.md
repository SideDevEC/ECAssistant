# Getting Started

ECAssistant is a small, open-source .NET agent library — plus a ready-made terminal app. Your models, your keys, your machine.

## Just use it in the terminal

```bash
dotnet tool install -g ECAssistant.Console
ecassistant
```

First run asks **local or remote**:

- **Local** — the wizard installs the LLM server to `~/.ECAssistantLLM/server/` (from NuGet, no manual steps), then you pick chat / vision / embedding models from the built-in catalog. Downloads are SHA-256 verified. After setup nothing downloads at chat time.
- **Remote** — point it at any OpenAI-compatible endpoint (OpenAI, OpenRouter, a self-hosted server, …). Only a config entry — no code.

Then start chatting. The agent can read files, run shell commands, edit code, use git and dotnet, and search the web — each tool permission-gated (approve / always / never per tool).

## Build an agent into your app

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

## Next steps

- [Agent lifecycle](agent-lifecycle.md) — composition root → session → orchestrator
- [Tools & permissions](tools-and-permissions.md) — the built-ins and how gating works
- [Local vs Remote](local-vs-remote.md) — GGUF vs OpenAI-compatible, switching
- [Security & Privacy](security-and-privacy.md) — what runs where, what leaves your machine
