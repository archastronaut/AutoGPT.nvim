We are building a Neovim plugin that wraps a self-hosted AutoGPT backend, mediated by a Go-based service. It's not just a chat tool - it's a programmable AI interface that can reason, edit, and refactor code seamlessly and interactively.
On a high level architecture overview it looks like this:
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
#### Component Breakdown (Detailed)

1. Neovim Frontend (Lua)
    - Commands:
        - `agentRefactor:` - sends selected code
        - `:AgentExplain` - asks for explanation
        - `:AgentGenerateText` - creates unit test
    - File: `lua/gptagent/init.lua`, `commands.lua`
    - Responsibilities:
        - Capture current buffer state, filetype, function boundaries (via Treesitter)
        - Serialize into a payload (Using RAG or parallelization remains to be considered)
        - Display results in floating windows
        - Apply inline edits via diff-style preview
    - Communication:
        - Uses `plenary.job` or `vim.fn.jobstart` to send HTTP `Post`requests to Go backend

2. Go Backend
    - Entry point: `go-backend/main.go`
    - Route Example: `/refactor`, `/explain`
    - Core Tasks:
        - Validate and deserialize payload from Lua
        - Perform code context analysis (pull neighboring functions, etc.)
        - Construct a structured prompt with your custom template logic
        - Call AutoGPT backend (via REST or CLI interface)
        - Receive and process the AutoGPT output.
        - Return back structured response to Lua plugin
    - Internals:
        - `parser/prompt.go:` Prompt formatting and template filling
        - `routes/refactor.go:` Handles `/refactor` endpoint.
        - `memory/context.go:` In-memory cache of last edits or previous completions (for local memory simulation)

3. AutoGPT (Self-hosted)
    - Run Mode: Basically a CLI-based agent(`./run agent start`), locally hosted
    - Interface with Go:
        - Either via REST API(`localhost:8000`)-- if AutoGPT is running with HTTP frontend
        - Or by CLU subprocess + stdout read (advanced, less stable. optionally: ...)
    - Responsibilities: 
        - Plan: Break task into substeps
        - Act: Execute tools(internal commands or code tools)
        - Request: Call GPT-4.1 API with full context + objective
        - Respond: Generate human-readable (or machine-parseable) output

4. Flow of Data(refactor(lintering?) example)
    1. User visually selects a function in Neovim and calls `:AgentRefactor`.
    2. Lua gathers:
        - Selected code
        - Buffer metadata(file name, filetype, cursor context)
    3. Sends POST to `localhost:5050/refactor:`
        ```json:
        {
        "file": "user_handler.go",
        "language": "go",
        "selected_code": "func HandleUser(...) {...}",
        "task": "Refactor for readability"
        }
        ```
    4. Go backend receives it:
        - Looks up extra context (previous related functions or usages)
        - Fills prompt template:
        ```sql:
        You are an expert Go developer...
        Here is a function that needs refactoring:
        <code>
        ``` 
    5. Receives response:
        - Clean refactored code
        - Maybe suggestions or summary
        - Optional diff
    6. Sends JSON back to Lua:
        ```json:
        {
        "modified_code": "func HandleUser(...) { // cleaned code }",
        "summary": "Refactored to use defer and simplified error blocks."
        }
        ```
    7. Lua plugin:
        - Applies code in-place
        - Shows floating preview
        - Stores this interaction in local session

#### Future Modules:
    - `graph/:` knowledge graph or entity relationship extractor
    - `models/` we stick with GPT-4.1 for now.


#### Steps to reproduce:
... TBC