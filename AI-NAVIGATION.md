# AI-NAVIGATION.md — ECAssistant monorepo map for AI agents

> You are an AI agent working across the ECAssistant repos. This is your map. Each repo has its own `AGENTS.md` with deep detail — read it before touching that repo. Humans: each repo's README.md is your entry point.

## What ECAssistant is (one paragraph)

ECAssistant is a family of five MIT-licensed .NET 8 packages that turn any application into a **tool-using AI agent host** — plus a ready-made terminal app. Inference runs local (GGUF via a bundled OpenAI-compatible server) or remote (any OpenAI-compatible endpoint); the code path is identical, one config switch. Everything is permission-gated, streamed, and visible. Version 15.0.0, all packages in lockstep.

## The five repos + landing

| Repo | NuGet / Tool | One-liner | AGENTS.md |
|---|---|---|---|
| [ECAssistantCore](https://github.com/SideDevEC/ECAssistantCore) | `ECAssistant.Core` | The embeddable agent library: sessions, 11 tools, memory, orchestrator | [link](https://github.com/SideDevEC/ECAssistantCore/blob/main/AGENTS.md) |
| [ECAssistantLLM](https://github.com/SideDevEC/ECAssistantLLM) | `ECAssistant.LLM.Server` | Self-contained OpenAI-compatible local server (LLamaSharp); standalone product | [link](https://github.com/SideDevEC/ECAssistantLLM/blob/main/AGENTS.md) |
| [ECAssistantTUI](https://github.com/SideDevEC/ECAssistantTUI) | `ECAssistant.TUI` | Reusable terminal UI: streaming chat, tabs, tool rendering, wizards | [link](https://github.com/SideDevEC/ECAssistantTUI/blob/main/AGENTS.md) |
| [ECAssistantConsole](https://github.com/SideDevEC/ECAssistantConsole) | `ECAssistant.Console` (tool `ecassistant`) | Reference host / end-user CLI, thin (~100 lines of wiring) | [link](https://github.com/SideDevEC/ECAssistantConsole/blob/main/AGENTS.md) |
| [ECAssistantTestSupport](https://github.com/SideDevEC/ECAssistantTestSupport) | `ECAssistant.TestSupport` | Test harness: MockEngine, user-experience E2E, real-server journey tests | [link](https://github.com/SideDevEC/ECAssistantTestSupport/blob/main/AGENTS.md) |
| [ECAssistant](https://github.com/SideDevEC/ECAssistant) (this repo) | — | Landing: overview, docs/, demo | AI-NAVIGATION.md (this file) |

## Dependency direction (never violates this)

```
Console ──▶ TUI ──▶ Core ──▶ LLM.Server (HTTP client boundary only)
                        ▲
TestSupport ────────────┘        (test suites only — NEVER runtime projects)
Landing/docs ──▶ describes all of the above
```

- Core knows the server ONLY as an OpenAI-compatible HTTP endpoint. No LLamaSharp types leak into Core.
- No runtime project ever references TestSupport.
- Chain rule: when an upstream package ships a new version, ALL downstream packages bump their reference and ship too.

## How to choose the right repo for a task

| Your task | Work in | Because |
|---|---|---|
| "Add/fix a tool, orchestrator behavior, memory, config, permission" | ECAssistantCore | All agent logic lives there |
| "Streaming/rendering/tabs/wizard UI behavior" | ECAssistantTUI | All terminal rendering lives there |
| "CLI UX, wizard flow, packaging, tool install" | ECAssistantConsole | Host concerns |
| "Server endpoints, KV cache, GBNF decoding, model loading, VRAM" | ECAssistantLLM | Server internals |
| "Test infra: MockEngine, E2E harness, journey tests" | ECAssistantTestSupport (+ tests in Core) | Harness lives there |
| "Docs, positioning, landing README" | this repo | — |

## Bounded-context protocol (for large repos)

Core is 552 types — never read it whole. Per repo, load in this order and stop when you have what you need:

1. Repo's `AGENTS.md` (~1–2K tokens) — orientation + non-negotiables
2. `API-INDEX.md` — every public type, one line each (Core ~7.7K, LLM ~2.6K, TUI ~0.5K)
3. `RELATIONSHIP-GRAPH.md` — dependency edges only (Core ~1.8K)
4. `docs/packages/<Package>.API.md` — the package you're editing
5. The 1–3 source files you'll actually touch

Total per task: ~12–18K tokens, regardless of repo size.

## Build & verify (works in every repo)

```bash
export EcaUseProjectRefs=true      # sibling ProjectReferences — no NuGet auth, tests latest sources
export DOTNET_NODE_REUSE=false
dotnet build <project>.csproj      # 0 errors expected

dotnet test --filter "FullyQualifiedName~<TestClass>"   # targeted tests only
```

Full E2E (journey tests) against a real server: env-var gates + tiers documented in TestSupport's AGENTS.md. Local tier = small model (qwen35-4b), remote tier = large model (glm-5.3-flash:cloud). Verified green both tiers 2026-09-24.

## Versioning & release law (non-negotiable)

- **Unified versioning:** every package carries the same version (15.0.0). Bump all in lockstep — own version + all inter-package `PackageReference`s + `ServerInstallCoordinator.RequiredServerVersion` — one commit per repo, BEFORE the wave.
- **Train shipping:** tag → CI green → nuget.org indexed → confirm live → next. Order: TestSupport → LLM → Core → TUI → Console. Never all at once.
- **Never auto-tag.** Present the wave and wait for the owner's explicit "ship it".
- Pre-tag checklist per repo: ARCHITECTURE.md updated · LDC artifacts regenerated + enforcement PASSED · README/docs touched if user-facing · ALL projects (main + tests) build 0 errors locally.

## LDC artifacts (generated per repo, keep in sync)

Each repo carries: `ARCHITECTURE.md` (blueprint), `API-INDEX.md` (public surface), `RELATIONSHIP-GRAPH.md` (edges), `PACKAGE-MAP.md` (packages + token budgets), `.ldc/enforcement-report.json` (boundary violations = must be 0). Regenerate after any interface/public-type change; enforcement must PASS before release.

## Tone & house rules for agents

- Strict OOP: interfaces, constructor injection, no statics (pure functions/factories excepted), one type per file, tests alongside code (`{ClassName}Tests.cs`).
- Privacy: no telemetry, nothing phones home; never commit personal paths/names/tokens.
- Config sections are snake_case JSON (`llm_provider`, `model_tier`, `tool_output_limits`…).
- Small models get scaffolding + tight sampling; large models get headroom — never hardcode model assumptions in logic; the tier system owns it.
