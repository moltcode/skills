---
name: Install TypeScript LSP
description: Install typescript-language-server for TypeScript/JavaScript code intelligence
---

# Install TypeScript LSP

Installs `typescript-language-server` and `typescript` using `bun` into the agent-managed LSP directory.

## Prerequisites

- `bun` must be available on PATH

## File Types

`.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs`, `.mts`, `.cts`, `.json`

## LSP Kind

`typescript`

## Installation Steps

```bash
# 1. Create the install directory
mkdir -p ~/.moltcode_agents/lsps/typescript-language-server/latest/bin

# 2. Move into the install directory
cd ~/.moltcode_agents/lsps/typescript-language-server/latest

# 3. Initialize a package
bun init -y

# 4. Install typescript-language-server and typescript
bun add typescript-language-server typescript
```

Create a wrapper script at `~/.moltcode_agents/lsps/typescript-language-server/latest/bin/typescript-language-server` with the following content:

```bash
#!/usr/bin/env bash
exec node "$(dirname "$0")/../node_modules/.bin/typescript-language-server" --stdio "$@"
```

Make it executable:

```bash
chmod +x ~/.moltcode_agents/lsps/typescript-language-server/latest/bin/typescript-language-server
```

## Verification

```bash
~/.moltcode_agents/lsps/typescript-language-server/latest/bin/typescript-language-server --version
```

This should print the version number of typescript-language-server.

## Configuration

After installing the binary, add an LSP entry to the project's `.moltcode/project.yaml` if the project doesn't already have one. This is optional for simple single-root projects (the built-in ServerCatalog will find the binary automatically), but recommended for monorepos or when you need a custom working directory.

Add to `lsp_kinds:` in `project.yaml`:

```yaml
lsp_kinds:
  typescript:
    default_version: default
    label: TypeScript Language Server
    document:
      language_ids:
        ".ts": typescript
        ".tsx": typescriptreact
        ".js": javascript
        ".jsx": javascriptreact
        ".mjs": javascript
        ".cjs": javascript
        ".mts": typescript
        ".cts": typescript
        ".json": json
    versions:
      default:
        command: typescript-language-server
        args: ["--stdio"]
        cwd: .
        description: TypeScript/JavaScript language server
```

Then add a process entry under `processes:` that references this LSP kind:

```yaml
processes:
  my-project-typescript-lsp:
    type: lsp
    kind: typescript
    auto_start: false
    label: My Project TypeScript LSP
    restart:
      backoff_ms: 2000
      max_restarts: 3
      policy: on_failure
    tags: [group:lsp]
```

For monorepos with multiple TypeScript packages, use `version` on the process to select different `cwd` values. You can also use a project-local `pnpm exec` or `npx` to run the server from a specific package:

```yaml
lsp_kinds:
  typescript:
    default_version: frontend
    label: TypeScript Language Server
    document:
      language_ids:
        ".ts": typescript
        ".tsx": typescriptreact
        ".js": javascript
        ".jsx": javascriptreact
        ".mjs": javascript
        ".cjs": javascript
        ".mts": typescript
        ".cts": typescript
        ".json": json
    versions:
      frontend:
        command: pnpm
        args: ["exec", "typescript-language-server", "--stdio"]
        cwd: frontend
        description: TypeScript LSP for frontend package
      api:
        command: pnpm
        args: ["exec", "typescript-language-server", "--stdio"]
        cwd: api
        description: TypeScript LSP for API package
```

## How it works

- The Molt LSP system has a built-in ServerCatalog that knows about common LSP servers. For TypeScript/JavaScript, it expects `typescript-language-server --stdio`.
- If the binary is installed at `~/.moltcode_agents/lsps/typescript-language-server/latest/bin/typescript-language-server`, it will be found automatically via PATH.
- Adding to `project.yaml` is optional but recommended for monorepos where you need multiple LSP instances with different root paths (`cwd`).
- The `molt__LSP` tool automatically routes requests to the right LSP server based on file extension (`.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs`, `.mts`, `.cts`, `.json` all map to the `typescript` LSP kind).

## Troubleshooting

- **`bun: command not found`**: Bun is not installed or not on PATH. Check with `which bun`.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Network errors during `bun add`**: Check internet connectivity. Retry the `bun add` command.
- **Wrapper script fails with "node not found"**: Ensure `node` is on PATH. Bun installs packages but the wrapper uses `node` to run the server.
