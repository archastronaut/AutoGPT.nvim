We are building a Neovim plugin that wraps a self-hosted AutoGPT backend, mediated by a Go-based service. It's not just a chat tool - it's a programmable AI interface that can reason, edit, and refactor code seamlessly and interactively.
On a high level architecture overview it looks like this:
```text:
            ┌────────────────────────────┐
            │        Neovim User         │
            └────────────┬───────────────┘
                         │
                         ▼
            ┌────────────────────────────┐
            │      Lua Frontend UI       │
            │ - Commands (e.g. :Refactor)│
            │ - Code selection buffer    │
            │ - Floating window display  │
            └────────────┬───────────────┘
                         │
                [Local HTTP Request]
                         │
                         ▼
            ┌────────────────────────────┐
            │         Go Backend         │
            │ - HTTP API server          │
            │ - Prompt parsing logic     │
            │ - Context construction     │
            │ - Memory / caching (in mem)│
            └────────────┬───────────────┘
                         │
                [Structured Prompt]
                         │
                         ▼
            ┌────────────────────────────┐
            │       AutoGPT Engine       │
            │ - Planning loop            │
            │ - LLM request logic        │
            │ - Tool execution           │
            └────────────────────────────┘
```
