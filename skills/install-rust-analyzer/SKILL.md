---
name: Install rust-analyzer
description: Install rust-analyzer for Rust code intelligence
---

# Install rust-analyzer

Installs `rust-analyzer`, the Rust language server. Prefers `rustup` if available, falls back to downloading from GitHub releases.

## Prerequisites

- `rustup` (preferred) OR `curl`/`wget` for direct download

## File Types

`.rs`

## LSP Kind

`rust`

## Installation Steps (rustup — preferred)

If `rustup` is available:

```bash
# 1. Add the rust-analyzer component
rustup component add rust-analyzer

# 2. Create the install directory
mkdir -p ~/.moltcode_agents/lsps/rust-analyzer/latest/bin

# 3. Symlink the rustup-managed binary into the install directory
ln -sf "$(rustup which rust-analyzer 2>/dev/null || rustup run stable which rust-analyzer)" ~/.moltcode_agents/lsps/rust-analyzer/latest/bin/rust-analyzer
```

## Installation Steps (fallback — direct download)

If `rustup` is not available, download a prebuilt binary from GitHub releases:

```bash
# 1. Detect platform
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)

# Map platform to release target
case "${OS}-${ARCH}" in
  darwin-arm64)  TARGET="aarch64-apple-darwin" ;;
  darwin-x86_64) TARGET="x86_64-apple-darwin" ;;
  linux-aarch64) TARGET="aarch64-unknown-linux-gnu" ;;
  linux-x86_64)  TARGET="x86_64-unknown-linux-gnu" ;;
  *)             echo "Unsupported platform: ${OS}-${ARCH}" && exit 1 ;;
esac

# 2. Create the install directory
mkdir -p ~/.moltcode_agents/lsps/rust-analyzer/latest/bin

# 3. Download and extract
curl -L "https://github.com/rust-lang/rust-analyzer/releases/latest/download/rust-analyzer-${TARGET}.gz" \
  | gunzip > ~/.moltcode_agents/lsps/rust-analyzer/latest/bin/rust-analyzer

# 4. Make executable
chmod +x ~/.moltcode_agents/lsps/rust-analyzer/latest/bin/rust-analyzer
```

## Verification

```bash
~/.moltcode_agents/lsps/rust-analyzer/latest/bin/rust-analyzer --version
```

This should print the rust-analyzer version.

## Configuration

After installing the binary, add an LSP entry to the project's `.moltcode/project.yaml` if the project doesn't already have one. This is optional for simple single-root projects (the built-in ServerCatalog will find the binary automatically), but recommended for monorepos or when you need a custom working directory.

Add to `lsp_kinds:` in `project.yaml`:

```yaml
lsp_kinds:
  rust:
    default_version: default
    label: Rust Analyzer
    document:
      language_ids:
        ".rs": rust
    versions:
      default:
        command: rust-analyzer
        args: []
        cwd: .
        description: Rust language server
```

Then add a process entry under `processes:` that references this LSP kind:

```yaml
processes:
  my-project-rust-lsp:
    type: lsp
    kind: rust
    auto_start: false
    label: My Project Rust LSP
    restart:
      backoff_ms: 2000
      max_restarts: 3
      policy: on_failure
    tags: [group:lsp]
```

For monorepos with multiple Rust workspaces, use `version` on the process to select different `cwd` values:

```yaml
lsp_kinds:
  rust:
    default_version: main
    label: Rust Analyzer
    document:
      language_ids:
        ".rs": rust
    versions:
      main:
        command: rust-analyzer
        args: []
        cwd: .
        description: rust-analyzer for main workspace
      adapter:
        command: rust-analyzer
        args: []
        cwd: adapter_runtimes
        description: rust-analyzer for adapter runtimes
```

## How it works

- The Molt LSP system has a built-in ServerCatalog that knows about common LSP servers. For Rust, it expects `rust-analyzer` with no args.
- If the binary is installed at `~/.moltcode_agents/lsps/rust-analyzer/latest/bin/rust-analyzer`, it will be found automatically via PATH.
- Adding to `project.yaml` is optional but recommended for monorepos where you need multiple LSP instances with different root paths (`cwd`).
- The `molt__LSP` tool automatically routes requests to the right LSP server based on file extension (`.rs` maps to the `rust` LSP kind).

## Troubleshooting

- **`rustup: command not found`**: Rust toolchain not installed via rustup. Use the direct download fallback.
- **`rustup component add` fails**: Your Rust toolchain may be too old. Run `rustup update` first.
- **Download fails (404)**: The release URL may have changed. Check https://github.com/rust-lang/rust-analyzer/releases for the latest assets.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Binary not compatible**: If the downloaded binary crashes, ensure you detected the correct platform. On Apple Silicon, `uname -m` should return `arm64`.
