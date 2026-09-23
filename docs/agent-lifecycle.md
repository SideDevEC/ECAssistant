# Agent Lifecycle

How an ECAssistant agent goes from `new` to answering — the three moving parts are the **composition root**, the **session**, and the **decision loop**.

## 1. Composition root

```csharp
var root = new EcaCompositionRoot(userConfigDir, args);
var services = root.Build();
```

One call resolves and wires everything: configuration, model resolution (local or remote), tools, memory, and the inference transport. No service-locator magic, no hidden singletons — dependencies are explicit and swappable.

## 2. Session

```csharp
var sessions = new SessionManager(services.Config, services.ModelPath, workingDir, services.Logger);
var session = sessions.CreateSession("main");
session.AddListener(myListener);
await services.SessionBuilder.BuildAsync(session, externalTools: null);
```

- A **session** owns one conversation: its history, its memory, its tool state. Multiple concurrent sessions are supported.
- `AddListener` registers an `IOutputListener` — that's how you receive streamed tokens, tool calls, and tool results. (The terminal host uses this to render chat; your app can do anything: UI, logging, forwarding.)
- `BuildAsync` attaches the built-in tools and any **external tools** you register (implement one interface — that's the whole custom-tool API).

## 3. Orchestrator

```csharp
var orchestrator = session.Orchestrator;
var result = await orchestrator.ExecuteMultiStep("Summarize the docs in this folder");
Console.WriteLine(result.FinalOutput);
```

The orchestrator is the agent's brain:

1. Model proposes an answer or a tool call
2. Tool calls are checked against the **permission policy** (approve / always / never)
3. Results feed back into the model; it iterates until done
4. Everything streams to your listeners as it happens

## Memory

- **Vector memory** — embeddings-backed recall across sessions (local or remote embeddings)
- **Daily notes + curated long-term memory** — the agent writes down what matters and distills it over time

## Sub-agents

A session can spawn **sub-agents** for delegated work: clean child sessions with their own context, used for focused tasks. The parent stays lean; results come back as events.

## Where to go next

- [Tools & permissions](tools-and-permissions.md)
- [Local vs Remote](local-vs-remote.md)
- [Getting started](getting-started.md)
