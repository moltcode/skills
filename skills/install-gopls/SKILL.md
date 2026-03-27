---
name: Install gopls
description: Install gopls (Go language server) for Go code intelligence
---

# Install gopls

Installs `gopls`, the official Go language server, using `go install`.

## Prerequisites

- `go` must be available on PATH (Go 1.21+ recommended)

## File Types

`.go`, `.mod`, `.sum`, `.work`

## LSP Kind

`gopls`

## Installation Steps

```bash
# 1. Create the install directory
mkdir -p ~/.moltcode_agents/lsps/gopls/latest/bin

# 2. Install gopls directly into the target bin directory
GOBIN=$(echo ~/.moltcode_agents/lsps/gopls/latest/bin) go install golang.org/x/tools/gopls@latest
```

## Verification

```bash
~/.moltcode_agents/lsps/gopls/latest/bin/gopls version
```

This should print the gopls version and build information.

## Configuration

After installing the binary, add an LSP entry to the project's `.moltcode/project.yaml` if the project doesn't already have one. This is optional for simple single-root projects (the built-in ServerCatalog will find the binary automatically), but recommended for monorepos or when you need a custom working directory.

Add to `lsp_kinds:` in `project.yaml`:

```yaml
lsp_kinds:
  gopls:
    default_version: default
    label: gopls
    document:
      language_ids:
        ".go": go
        ".mod": go.mod
        ".sum": go.sum
        ".work": go.work
    versions:
      default:
        command: gopls
        args: ["serve"]
        cwd: .
        description: Go language server
```

Then add a process entry under `processes:` that references this LSP kind:

```yaml
processes:
  my-project-gopls:
    type: lsp
    kind: gopls
    auto_start: false
    label: My Project Go LSP
    restart:
      backoff_ms: 2000
      max_restarts: 3
      policy: on_failure
    tags: [group:lsp]
```

For monorepos with multiple Go modules, use `version` on the process to select different `cwd` values:

```yaml
lsp_kinds:
  gopls:
    default_version: main
    label: gopls
    document:
      language_ids:
        ".go": go
        ".mod": go.mod
        ".sum": go.sum
        ".work": go.work
    versions:
      main:
        command: gopls
        args: ["serve"]
        cwd: .
        description: gopls for main module
      tools:
        command: gopls
        args: ["serve"]
        cwd: tools
        description: gopls for tools module
```

## How it works

- The Molt LSP system has a built-in ServerCatalog that knows about common LSP servers. For Go, it expects `gopls serve`.
- If the binary is installed at `~/.moltcode_agents/lsps/gopls/latest/bin/gopls`, it will be found automatically via PATH.
- Adding to `project.yaml` is optional but recommended for monorepos where you need multiple LSP instances with different root paths (`cwd`).
- The `molt__LSP` tool automatically routes requests to the right LSP server based on file extension (`.go`, `.mod`, `.sum`, `.work` all map to the `gopls` LSP kind).

## Troubleshooting

- **`go: command not found`**: Go is not installed or not on PATH. Check with `which go`.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Network errors**: `go install` needs to download the module. Check internet connectivity and Go proxy settings (`GOPROXY`).
- **Old Go version**: gopls requires Go 1.21+. Check with `go version`.
