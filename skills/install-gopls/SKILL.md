---
name: Install gopls
description: Install gopls (Go language server) for Go code intelligence
---

# Install gopls

Installs `gopls`, the official Go language server, using `go install`.

## Prerequisites

- `go` must be available on PATH (Go 1.21+ recommended)

## File Types

`.go`

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

## Troubleshooting

- **`go: command not found`**: Go is not installed or not on PATH. Check with `which go`.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Network errors**: `go install` needs to download the module. Check internet connectivity and Go proxy settings (`GOPROXY`).
- **Old Go version**: gopls requires Go 1.21+. Check with `go version`.
