# Testing Your Agent

How to test ECAssistant-based agents at three levels — unit, user-experience, and full E2E — using [ECAssistant.TestSupport](https://github.com/SideDevEC/ECAssistantTestSupport) (the same harness that tests ECAssistant itself).

## Level 1 — MockEngine: logic without infrastructure

The fake engine feeds queued decisions; everything else (orchestrator, tools, permissions) is the real product code.

```csharp
var engine = new MockEngine(
    cycleResponses: new[]
    {
        MockEngine.ToolCall("EShellAgent", new { command = "echo hello" }),
        MockEngine.Answer("done — echoed hello")
    },
    config: new AppConfig());

var runner = new TestRunner(engine, workingDir: tempDir);
var result = await runner.RunAsync(TestScenario.Create("shell echo"));
Assert.True(result.Passed);
```

Fast enough to run thousands; no model download, no GPU, no server.

## Level 2 — UserExperienceHarness: assert on the transcript

The user doesn't see engine internals — they see the transcript. This harness drives `AgentSession.Prompt` (the real entry point), captures every visible line, answers approval prompts like a user, and registers the **real tool set**:

```csharp
var harness = new UserExperienceHarness(serverUrl, config);
var (session, dir) = await harness.CreateAsync();

await session.Prompt("Create a file named notes.md containing 'journey marker 42'");

Assert.Contains("notes.md", harness.VisibleTranscript);                 // what the user saw
Assert.Equal("journey marker 42",
             File.ReadAllText(Path.Combine(dir, "notes.md")));          // what the tool did
```

## Level 3 — Journey E2E: real server, real models, both tiers

The full feature matrix against a live [ECAssistantLLM](https://github.com/SideDevEC/ECAssistantLLM) server. Environment-gated — **silently skipped when unset**, so CI without secrets stays green.

```bash
# LOCAL SMALL — small local model, tier "small" (default qwen35-4b)
export ECA_E2E_SERVER=http://localhost:48321
export ECA_E2E_MODEL=qwen35-4b

# REMOTE LARGE — hosted large model, tier "large"
export ECA_E2E_REMOTE_ENDPOINT=https://ollama.com/v1
export ECA_E2E_REMOTE_MODEL=glm-5.3-flash:cloud
export ECA_E2E_REMOTE_KEYFILE=~/.secrets/ollama.key

export EcaUseProjectRefs=true
dotnet test --filter "FullyQualifiedName~JourneySuiteE2E"
```

Handy knobs: `ECA_JOURNEY_DEBUG=1` (per-turn transcripts + context snapshots), `prepareWorkingDir` (isolate file access per test).

### Test tier conventions

- **Local = small model** (`qwen35-4b`): validates scaffolding, strict recipes, permission flow under tight sampling.
- **Remote = large model** (`glm-5.3-flash:cloud`): validates slim profile, dataflow chains, longer reasoning.
- Run BOTH before releases — the tiers exercise different harness paths by design.

## House rules (learned the hard way)

- Targeted test runs only during interactive work: `--filter` to one class. Full suites are for release checkpoints.
- Kill leftover server processes after E2E: `pkill -f "ECAssistant.LLM.dll"`.
- The server binds `localhost` (IPv6) — use `http://localhost:PORT`, not `127.0.0.1`.
- The harness is test-suites-only: never reference `ECAssistant.TestSupport` from runtime projects.

## Where to go

- Harness reference: [TestSupport README](https://github.com/SideDevEC/ECAssistantTestSupport)
- Journey suite source: [JourneySuiteE2E.cs](https://github.com/SideDevEC/ECAssistantCore/blob/main/Tests/E2E/JourneySuiteE2E.cs)
- [Agent lifecycle](agent-lifecycle.md) · [Getting started](getting-started.md)
