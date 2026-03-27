---
name: Install Expert
description: Install Expert (Elixir language server) for Elixir code intelligence
---

# Install Expert

Installs `expert`, the official Elixir language server from elixir-lang, by downloading a prebuilt binary from GitHub releases.

## Prerequisites

- `curl` or `wget` for downloading
- `tar` for extraction

## File Types

`.ex`, `.exs`, `.heex`, `.leex`, `.eex`

## LSP Kind

`elixir`

## Installation Steps

```bash
# 1. Detect platform
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)

# Map arm64 to aarch64
if [ "$ARCH" = "arm64" ]; then
  ARCH="aarch64"
fi

# 2. Create the install directory
mkdir -p ~/.moltcode_agents/lsps/expert/latest/bin

# 3. Download the prebuilt binary
curl -L "https://github.com/elixir-lang/expert/releases/latest/download/expert-${OS}-${ARCH}.tar.gz" \
  -o /tmp/expert.tar.gz

# 4. Extract into the bin directory
tar xzf /tmp/expert.tar.gz -C ~/.moltcode_agents/lsps/expert/latest/bin/

# 5. Make executable
chmod +x ~/.moltcode_agents/lsps/expert/latest/bin/expert

# 6. Clean up
rm -f /tmp/expert.tar.gz
```

## Verification

```bash
~/.moltcode_agents/lsps/expert/latest/bin/expert --version
```

This should print the Expert version.

## Configuration

After installing the binary, add an LSP entry to the project's `.moltcode/project.yaml` if the project doesn't already have one. This is optional for simple single-root projects (the built-in ServerCatalog will find the binary automatically), but recommended for monorepos or when you need a custom working directory.

Add to `lsp_kinds:` in `project.yaml`:

```yaml
lsp_kinds:
  elixir:
    default_version: default
    label: Expert (Elixir)
    document:
      language_ids:
        ".ex": elixir
        ".exs": elixir
        ".heex": phoenix-heex
        ".leex": html-eex
        ".eex": html-eex
    versions:
      default:
        command: expert
        args: ["--stdio"]
        cwd: .
        description: Elixir language server
```

Then add a process entry under `processes:` that references this LSP kind:

```yaml
processes:
  my-project-elixir-lsp:
    type: lsp
    kind: elixir
    auto_start: false
    label: My Project Elixir LSP
    restart:
      backoff_ms: 2000
      max_restarts: 3
      policy: on_failure
    tags: [group:lsp]
```

For monorepos with multiple Elixir sub-projects, use `version` on the process to select different `cwd` values:

```yaml
lsp_kinds:
  elixir:
    default_version: umbrella
    label: Expert (Elixir)
    document:
      language_ids:
        ".ex": elixir
        ".exs": elixir
        ".heex": phoenix-heex
        ".leex": html-eex
        ".eex": html-eex
    versions:
      umbrella:
        command: expert
        args: ["--stdio"]
        cwd: apps/umbrella
        description: Elixir LSP for umbrella app
      hub:
        command: expert
        args: ["--stdio"]
        cwd: apps/hub
        description: Elixir LSP for hub app
```

## How it works

- The Molt LSP system has a built-in ServerCatalog that knows about common LSP servers. For Elixir, it expects `expert --stdio`.
- If the binary is installed at `~/.moltcode_agents/lsps/expert/latest/bin/expert`, it will be found automatically via PATH.
- Adding to `project.yaml` is optional but recommended for monorepos where you need multiple LSP instances with different root paths (`cwd`).
- The `molt__LSP` tool automatically routes requests to the right LSP server based on file extension (`.ex`, `.exs`, `.heex`, `.leex`, `.eex` all map to the `elixir` LSP kind).

## Troubleshooting

- **Download fails (404)**: The release URL may have changed. Check https://github.com/elixir-lang/expert/releases for the latest assets and correct naming convention.
- **`tar: Error opening archive`**: The download may have failed silently. Check the downloaded file size and re-download.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Binary not compatible / exec format error**: Ensure you detected the correct platform. On Apple Silicon Macs, `uname -m` returns `arm64` which should be mapped to `aarch64`.
- **Missing shared libraries (Linux)**: The prebuilt binary may require system libraries. Check `ldd ~/.moltcode_agents/lsps/expert/latest/bin/expert` for missing dependencies.
