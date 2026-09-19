# Tools & Permissions

An agent is only as useful as what it's allowed to *do* — and as trustworthy as how you control it. ECAssistant ships 12 built-in tools, all permission-gated, plus a one-interface API for your own.

## The built-ins

| Tool | What it does |
|---|---|
| **Shell** | Run shell commands (with your approval) |
| **Files** | Read, write, and edit files in the working directory |
| **Code editing** | Targeted, precise code changes |
| **Git** | Status, diff, commit, push, branch operations |
| **dotnet** | Build, test, and project operations |
| **Web search** | Search the web for current information |
| **Web fetch** | Fetch and read pages/URLs |
| **Background** | Long-running commands with completion wake-ups |
| **Research** | Structured multi-source research tasks |
| **Reader** | Deep reading of documents and codebases |
| **Sub-agents** | Spawn clean child sessions for delegated work |
| **Build** | Structured build/run orchestration |

## Permission gating

Every tool call goes through the policy before it executes:

- **approve** — the agent asks, you decide (default for sensitive tools like Shell)
- **always** — you trust this tool; it runs without asking
- **never** — disabled; the agent knows it can't use it

The policy is **per tool**, so a typical setup looks like: Files = always, Shell = approve, Web = approve. The agent *shows its work* — every call, input, and result is visible to you before or as it happens. That's the assist-first principle: you stay in the loop.

## Custom tools

Implement one interface, register it, done:

```csharp
// 1. Implement the tool interface (IExternalTool)
// 2. Register it when building the session:
await services.SessionBuilder.BuildAsync(session, externalTools: new[] { new MyTool() });
```

External tools get the same permission gating, streaming, and rendering as built-ins — no special cases.

## Where to go next

- [Agent lifecycle](agent-lifecycle.md)
- [Security & Privacy](security-and-privacy.md)
