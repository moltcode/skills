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

`.ex`, `.exs`

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

## Troubleshooting

- **Download fails (404)**: The release URL may have changed. Check https://github.com/elixir-lang/expert/releases for the latest assets and correct naming convention.
- **`tar: Error opening archive`**: The download may have failed silently. Check the downloaded file size and re-download.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Binary not compatible / exec format error**: Ensure you detected the correct platform. On Apple Silicon Macs, `uname -m` returns `arm64` which should be mapped to `aarch64`.
- **Missing shared libraries (Linux)**: The prebuilt binary may require system libraries. Check `ldd ~/.moltcode_agents/lsps/expert/latest/bin/expert` for missing dependencies.
