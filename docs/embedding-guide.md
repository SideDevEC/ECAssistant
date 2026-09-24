# Embedding the Agent in Your App

The Core library's reason to exist: your application gains a tool-using agent in ~15 lines. Same wiring the Console uses — read [ConsoleApplication.cs](https://github.com/SideDevEC/ECAssistantConsole/blob/main/ConsoleApplication.cs) as the reference.

## The wiring, explained step by step

```csharp
using ECAssistant.Core.Composition;
using ECAssistant.Core.Session;

// 1. Composition root — reads appsettings.json from userConfigDir, resolves the
//    model (local catalog or remote endpoint), builds tool + memory services.
var root     = new EcaCompositionRoot(userConfigDir, args);
var services = root.Build();

// 2. Session manager — scope: one working directory per session group.
var sessions = new SessionManager(services.Config, services.ModelPath, workingDir, services.Logger);
var session  = sessions.CreateSession("main");

// 3. Output listener — you receive every streamed token, tool call, approval
//    request and status hint. Render them however you like (TUI, desktop, web).
session.AddListener(new MyListener());

// 4. Build the session — registers the 11 built-in tools, memory, policies.
await services.SessionBuilder.BuildAsync(session, externalTools: null);

// 5. Run. The orchestrator plans, calls tools (permission-gated), self-corrects,
//    and returns when the goal is done.
var result = await session.Orchestrator.ExecuteMultiStep("Summarize the docs in this folder");
Console.WriteLine(result.FinalOutput);
```

### What each piece owns

| Piece | Owns |
|---|---|
| `EcaCompositionRoot` | Config load, model resolution, DI wiring — the only constructor you need |
| `SessionManager` | Session lifetime, per-session context windows, idle handling |
| `AgentSession` | The unit of conversation: `Prompt` (single user turn) / `Orchestrator.ExecuteMultiStep` (goal) |
| `IOutputListener` | Your render hook: streamed tokens, thinking blocks, tool events, status hints |
| `SessionBuilder` | Registers built-in + custom tools, permissions, verification gates |

### Hosting options

- **Console app** — the reference: [ECAssistantConsole](https://github.com/SideDevEC/ECAssistantConsole)
- **Desktop / service** — same code; swap the listener for your UI framework
- **Headless** — implement the approval policy programmatically (auto-approve whitelisted tools, deny the rest); the agent still never works "in the dark" — every decision lands in the transcript

## Model tiers: small vs large, same code

Set `model_tier` in config — the harness adapts automatically:

```jsonc
{ "llm_provider": { "mode": "local",  "model_id": "qwen35-4b" },
  "model_tier":   { "mode": "small" } }        // scaffolding, tight sampling, strict recipes

{ "llm_provider": { "mode": "remote", "model_id": "glm-5.3-flash:cloud",
                    "endpoint": "https://ollama.com/v1" },
  "model_tier":   { "mode": "large" } }        // slim profile, dataflow chains, more headroom
```

Small models get step-by-step scaffolding and tighter sampling; large models get a slim profile with more headroom. Verified: the same 6-journey E2E suite passes 6/6 on a 4B local model *and* on a hosted large model.

## Where to go

- [Getting started](getting-started.md) — install paths
- [Agent lifecycle](agent-lifecycle.md) — what the orchestrator does each turn
- [Custom tools](custom-tools.md) — teach it new capabilities
- [Local vs remote](local-vs-remote.md) — inference backends
