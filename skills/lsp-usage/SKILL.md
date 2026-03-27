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

| Language | File Extensions | LSP Kind | Server Binary |
|----------|----------------|----------|---------------|
| TypeScript/JavaScript | `.ts`, `.tsx`, `.js`, `.jsx` | `typescript` | `typescript-language-server` |
| Go | `.go` | `gopls` | `gopls` |
| Rust | `.rs` | `rust` | `rust-analyzer` |
| Python | `.py` | `python` | `pyright-langserver` |
| Elixir | `.ex`, `.exs` | `elixir` | `expert` |

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

## Tips

- **Start with `documentSymbol`** when exploring unfamiliar code — it gives you the full structure at a glance.
- **Use `workspaceSymbol` over grep** for finding functions and types — it understands language semantics and ignores comments/strings.
- **Combine `goToDefinition` + `findReferences`** for refactoring — definition tells you what it is, references tell you what depends on it.
- **`hover` is underrated** — it shows inferred types, documentation, and function signatures without leaving your current position.
- **Check `incomingCalls` before modifying a function** — know your callers before changing signatures.
