---
name: Install Pyright
description: Install pyright-langserver for Python code intelligence
---

# Install Pyright

Installs `pyright-langserver` (Pyright language server) using `bun` into the agent-managed LSP directory.

## Prerequisites

- `bun` must be available on PATH

## File Types

`.py`

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

## Troubleshooting

- **`bun: command not found`**: Bun is not installed or not on PATH. Check with `which bun`.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Network errors during `bun add`**: Check internet connectivity. Retry the `bun add` command.
- **Wrapper script fails with "node not found"**: Ensure `node` is on PATH. Bun installs packages but the wrapper uses `node` to run the server.
