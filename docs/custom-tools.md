# Custom Tools

The most common extension point: teach the agent a new capability in one file. A tool is a class implementing the tool contract — register it, and the agent can call it like any built-in.

## Minimal tool

```csharp
using System.Text.Json.Nodes;
using ECAssistant.Core.Tools;

public sealed class EWeatherTool : EToolBase
{
    public override string Name => "EWeather";
    public override string Description => "Get current weather for a city";

    // JSON Schema for the arguments — drives GBNF grammar generation server-side,
    // so the model's tool call is always typed and valid.
    protected override JsonObject BuildSchema() => new()
    {
        ["type"] = "object",
        ["properties"] = new JsonObject
        {
            ["city"]  = new JsonObject { ["type"] = "string" },
            ["units"] = new JsonObject { ["type"] = "string", ["enum"] = new JsonArray("c", "f") }
        },
        ["required"] = new JsonArray("city")
    };

    protected override async Task<EToolResult> ExecuteCoreAsync(
        JsonElement args, CancellationToken ct)
    {
        var city  = args.GetProperty("city").GetString()!;
        var units = args.TryGetProperty("units", out var u) ? u.GetString() : "c";
        // ... your API call, file work, anything ...
        return EToolResult.Ok($"Weather in {city}: 21°{units}, clear");
    }
}
```

Register it when building the session:

```csharp
await services.SessionBuilder.BuildAsync(session, externalTools: new[] { new EWeatherTool() });
```

That's the whole API. The tool appears in the model's tool list, its calls are grammar-constrained to your schema, and results stream to every listener.

## Permission gating

Tools are permission-gated: **approve once / always this session / deny**. Default policy per tool via config:

```jsonc
{ "tools": { "weather": { "default_policy": "always" } } }
```

Sensitive built-ins (shell, git, dotnet) default to **approve** — the user sees exactly what will run before it runs. Session-scoped only; nothing persists across restarts.

## Typed model-facing output (projections)

Tools "speak for themselves": each tool renders the result the *model* sees — not the raw dump the *user* sees. Build output collapses to verdict + parsed errors; a long file read collapses to the relevant window. Override one virtual method:

```csharp
protected override string ProjectForModel(EToolResult result) =>
    $"{result.Success} · {result.Summary}";   // keep it small, keep it decisive
```

Smaller projections → smaller context → sharper next decisions. This matters most on small local models.

## Dataflow chains (large models)

A later tool call in the same decision can reference an earlier call's output with `{{0}}`:

```json
{ "tool": "EGit",   "args": { "cmd": "status --porcelain" } },
{ "tool": "EShell", "args": { "command": "wc -l {{0}}" } }
```

Sequential execution + argument substitution — no model round-trip between steps. Taught to large-tier models only; small models keep single calls.

## Where to go

- Built-in tool list: [tools-and-permissions.md](tools-and-permissions.md)
- MCP (external tool servers, zero code): see the Core README's MCP section
- Full class reference: [API-INDEX.md](https://github.com/SideDevEC/ECAssistantCore/blob/main/API-INDEX.md) in ECAssistantCore
