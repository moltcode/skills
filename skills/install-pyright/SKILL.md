---
name: Install Pyright
description: Install pyright-langserver for Python code intelligence
---

# Install Pyright

Installs `pyright-langserver` (Pyright language server) using `bun` into the agent-managed LSP directory.

## Prerequisites

- `bun` must be available on PATH

## File Types

`.py`, `.pyi`

## LSP Kind

`python`

## Installation Steps

```bash
# 1. Create the install directory
mkdir -p ~/.moltcode_agents/lsps/pyright/latest/bin

# 2. Move into the install directory
cd ~/.moltcode_agents/lsps/pyright/latest

# 3. Initialize a package
bun init -y

# 4. Install pyright
bun add pyright
```

Create a wrapper script at `~/.moltcode_agents/lsps/pyright/latest/bin/pyright-langserver` with the following content:

```bash
#!/usr/bin/env bash
exec node "$(dirname "$0")/../node_modules/.bin/pyright-langserver" --stdio "$@"
```

Make it executable:

```bash
chmod +x ~/.moltcode_agents/lsps/pyright/latest/bin/pyright-langserver
```

## Verification

```bash
~/.moltcode_agents/lsps/pyright/latest/bin/pyright-langserver --version
```

This should print the Pyright version number.

## Configuration

After installing the binary, add an LSP entry to the project's `.moltcode/project.yaml` if the project doesn't already have one. This is optional for simple single-root projects (the built-in ServerCatalog will find the binary automatically), but recommended for monorepos or when you need a custom working directory.

Add to `lsp_kinds:` in `project.yaml`:

```yaml
lsp_kinds:
  python:
    default_version: default
    label: Pyright
    document:
      language_ids:
        ".py": python
        ".pyi": python
    versions:
      default:
        command: pyright-langserver
        args: ["--stdio"]
        cwd: .
        description: Python language server
```

Then add a process entry under `processes:` that references this LSP kind:

```yaml
processes:
  my-project-python-lsp:
    type: lsp
    kind: python
    auto_start: false
    label: My Project Python LSP
    restart:
      backoff_ms: 2000
      max_restarts: 3
      policy: on_failure
    tags: [group:lsp]
```

For monorepos with multiple Python packages, use `version` on the process to select different `cwd` values:

```yaml
lsp_kinds:
  python:
    default_version: backend
    label: Pyright
    document:
      language_ids:
        ".py": python
        ".pyi": python
    versions:
      backend:
        command: pyright-langserver
        args: ["--stdio"]
        cwd: backend
        description: Pyright for backend package
      scripts:
        command: pyright-langserver
        args: ["--stdio"]
        cwd: scripts
        description: Pyright for scripts package
```

## How it works

- The Molt LSP system has a built-in ServerCatalog that knows about common LSP servers. For Python, it expects `pyright-langserver --stdio`.
- If the binary is installed at `~/.moltcode_agents/lsps/pyright/latest/bin/pyright-langserver`, it will be found automatically via PATH.
- Adding to `project.yaml` is optional but recommended for monorepos where you need multiple LSP instances with different root paths (`cwd`).
- The `molt__LSP` tool automatically routes requests to the right LSP server based on file extension (`.py` and `.pyi` map to the `python` LSP kind).

## Troubleshooting

- **`bun: command not found`**: Bun is not installed or not on PATH. Check with `which bun`.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Network errors during `bun add`**: Check internet connectivity. Retry the `bun add` command.
- **Wrapper script fails with "node not found"**: Ensure `node` is on PATH. Bun installs packages but the wrapper uses `node` to run the server.
