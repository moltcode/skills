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

## Troubleshooting

- **`rustup: command not found`**: Rust toolchain not installed via rustup. Use the direct download fallback.
- **`rustup component add` fails**: Your Rust toolchain may be too old. Run `rustup update` first.
- **Download fails (404)**: The release URL may have changed. Check https://github.com/rust-lang/rust-analyzer/releases for the latest assets.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Binary not compatible**: If the downloaded binary crashes, ensure you detected the correct platform. On Apple Silicon, `uname -m` should return `arm64`.
