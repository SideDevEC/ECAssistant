# Local vs Remote

ECAssistant doesn't lock you into either side. Same code, one config change.

## Local (GGUF)

Models run on your machine via the bundled [ECAssistantLLM server](https://github.com/SideDevEC/ECAssistantLLM) — an OpenAI-compatible server powered by LLamaSharp/llama.cpp.

- **Private by construction** — inference never leaves `localhost`
- **Offline-capable** — after setup, nothing downloads at chat time
- **Multi-model** — chat, vision (mmproj), and embedding models load side by side with a VRAM budget
- **Multi-client** — several ECAssistant apps share one server instance; it shuts down when the last client disconnects
- **Structured decoding** — GBNF grammar-constrained output for reliable JSON

The setup wizard handles everything: server install (from NuGet), model selection from the built-in catalog with SHA-256 verification, and backend runtimes for special model formats.

## Remote (OpenAI-compatible)

Point `llm_provider` at any OpenAI-compatible endpoint — OpenAI, OpenRouter, a corporate gateway, or your own ECAssistantLLM server elsewhere on the network. Only a config entry changes; the code path is identical.

## Switching

Local ↔ remote is a config flip, not a refactor. Build against `ECAssistant.Core` once, and:

- develop locally on a small GGUF model
- demo against a frontier remote model
- ship whichever your users want

Embeddings work the same way — local GGUF embeddings or a remote endpoint.

## Where to go next

- [Getting started](getting-started.md)
- [Security & Privacy](security-and-privacy.md)
