---
name: LSP Usage Guide
description: How to use the molt__LSP tool for code intelligence
---

# LSP Usage Guide

The `molt__LSP` tool provides code intelligence operations powered by language servers. This guide covers available operations, how to use them, and which LSP server handles which languages.

## Available Operations

| Operation | Description |
|-----------|-------------|
| `goToDefinition` | Jump to where a symbol is defined |
| `findReferences` | Find all references to a symbol |
| `hover` | Get type info and documentation for a symbol |
| `documentSymbol` | List all symbols in a file (functions, classes, variables) |
| `workspaceSymbol` | Search for symbols across the entire workspace |
| `goToImplementation` | Find implementations of an interface or abstract method |
| `prepareCallHierarchy` | Get the call hierarchy item at a position |
| `incomingCalls` | Find all callers of a function |
| `outgoingCalls` | Find all functions called by a function |

## Parameters

All operations require:

- **`filePath`** — Absolute path to the file
- **`line`** — Line number (1-based)
- **`character`** — Column number (1-based)

For `workspaceSymbol`, provide a `query` string instead of line/character.

## Language → LSP Server Mapping

### File Extension → LSP Kind

The `molt__LSP` tool uses file extensions to determine which LSP server to route requests to. It checks the project's `project.yaml` first, then falls back to the built-in ServerCatalog.

| File Extension | LSP Kind | Server Binary | Server Command |
|---------------|----------|---------------|----------------|
| `.ts`, `.tsx`, `.mjs`, `.cjs`, `.mts`, `.cts`, `.json` | `typescript` | `typescript-language-server` | `typescript-language-server --stdio` |
| `.js`, `.jsx` | `typescript` | `typescript-language-server` | `typescript-language-server --stdio` |
| `.go`, `.mod`, `.sum`, `.work` | `gopls` | `gopls` | `gopls serve` |
| `.rs` | `rust` | `rust-analyzer` | `rust-analyzer` |
| `.py`, `.pyi` | `python` | `pyright-langserver` | `pyright-langserver --stdio` |
| `.ex`, `.exs`, `.heex`, `.leex`, `.eex` | `elixir` | `expert` | `expert --stdio` |

### LSP Kind → Language IDs

Each LSP kind maps file extensions to LSP language identifiers used in the protocol:

| LSP Kind | Extension → Language ID |
|----------|------------------------|
| `typescript` | `.ts` → `typescript`, `.tsx` → `typescriptreact`, `.js` → `javascript`, `.jsx` → `javascriptreact`, `.mjs`/`.cjs` → `javascript`, `.mts`/`.cts` → `typescript`, `.json` → `json` |
| `gopls` | `.go` → `go`, `.mod` → `go.mod`, `.sum` → `go.sum`, `.work` → `go.work` |
| `rust` | `.rs` → `rust` |
| `python` | `.py` → `python`, `.pyi` → `python` |
| `elixir` | `.ex`/`.exs` → `elixir`, `.heex` → `phoenix-heex`, `.leex`/`.eex` → `html-eex` |

## Server Resolution

The LSP system resolves which server to use through a multi-step process:

1. **Project config first**: Check `lsp_kinds` in `.moltcode/project.yaml` for a matching file extension
2. **Built-in fallback**: If no project config matches, use the built-in `@extension_to_kind` mapping
3. **Process lookup**: Find a configured LSP process of that kind in the project's `processes:` section
4. **ServerCatalog fallback**: If no process is configured, try the built-in ServerCatalog and create a synthetic process ID (`lsp_<kind>`)
5. **Binary discovery**: The binary is resolved from PATH, which includes `~/.moltcode_agents/lsps/<name>/latest/bin/`

### When project.yaml is NOT needed

For simple, single-root projects, you don't need to configure `project.yaml` at all. If the LSP binary is:
- On your system PATH, or
- Installed at `~/.moltcode_agents/lsps/<name>/latest/bin/<binary>`

...the ServerCatalog will find it automatically and use the project root as the working directory.

### When project.yaml IS needed

You should add LSP configuration to `.moltcode/project.yaml` when:
- **Monorepos**: You need different LSP instances with different root paths (e.g., `frontend/` vs `api/`)
- **Custom commands**: You want to run the server via a package manager (e.g., `pnpm exec typescript-language-server --stdio`)
- **Multiple versions**: You need different server configurations for different parts of the project
- **Custom language IDs**: You want to override which file extensions map to which LSP kind

### project.yaml LSP structure

LSP configuration has two parts: `lsp_kinds` (server definitions) and `processes` (instances):

```yaml
# 1. Define the LSP kind with its server command and language mappings
lsp_kinds:
  typescript:                          # The LSP kind identifier
    default_version: frontend          # Which version to use by default
    label: TypeScript Language Server   # Display name
    document:
      language_ids:                    # File extension → LSP language ID
        ".ts": typescript
        ".tsx": typescriptreact
        ".js": javascript
        ".jsx": javascriptreact
    versions:
      frontend:                        # A named version/configuration
        command: pnpm                  # Binary to execute
        args: ["exec", "typescript-language-server", "--stdio"]
        cwd: frontend                  # Working directory (relative to project root)
        description: Frontend TypeScript LSP

# 2. Create process entries that reference the LSP kind
processes:
  frontend-typescript-lsp:
    type: lsp                          # Must be "lsp"
    kind: typescript                   # References the lsp_kinds key above
    version: frontend                  # Optional: selects a specific version
    auto_start: false
    label: Frontend TypeScript LSP
    restart:
      backoff_ms: 2000
      max_restarts: 3
      policy: on_failure
    tags: [group:lsp]
```

## Common Workflows

### Exploring a file's structure

Use `documentSymbol` to get an overview of all symbols in a file. This is useful when you first open a file and want to understand its structure.

### Navigating code

1. Use `goToDefinition` to jump to where a function, type, or variable is defined
2. Use `findReferences` to find everywhere a symbol is used
3. Use `goToImplementation` to find concrete implementations of interfaces

### Understanding call chains

1. Use `prepareCallHierarchy` on a function to get its call hierarchy item
2. Use `incomingCalls` to see what calls this function
3. Use `outgoingCalls` to see what this function calls

### Cross-file search

Use `workspaceSymbol` with a query string to find symbols across the entire workspace. This is faster and more precise than text search for finding functions, types, and variables.

## Handling "Binary Not Found" Errors

If the LSP tool returns a "binary not found" error, the corresponding language server is not installed. Install it using the appropriate skill:

| LSP Kind | Install Skill |
|----------|--------------|
| `typescript` | `install-typescript-lsp` |
| `gopls` | `install-gopls` |
| `rust` | `install-rust-analyzer` |
| `python` | `install-pyright` |
| `elixir` | `install-expert` |

## Monorepo Awareness

The LSP system routes requests to the correct language server based on the file path and the project's `project.yaml` configuration. In monorepos with multiple languages, each file is automatically handled by the appropriate server — no manual routing needed.

For monorepos that need multiple instances of the same LSP kind (e.g., separate TypeScript servers for `frontend/` and `api/`), configure multiple versions under `lsp_kinds` and reference them with separate process entries.

## Tips

- **Start with `documentSymbol`** when exploring unfamiliar code — it gives you the full structure at a glance.
- **Use `workspaceSymbol` over grep** for finding functions and types — it understands language semantics and ignores comments/strings.
- **Combine `goToDefinition` + `findReferences`** for refactoring — definition tells you what it is, references tell you what depends on it.
- **`hover` is underrated** — it shows inferred types, documentation, and function signatures without leaving your current position.
- **Check `incomingCalls` before modifying a function** — know your callers before changing signatures.
