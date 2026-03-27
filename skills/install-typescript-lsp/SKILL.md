---
name: Install TypeScript LSP
description: Install typescript-language-server for TypeScript/JavaScript code intelligence
---

# Install TypeScript LSP

Installs `typescript-language-server` and `typescript` using `bun` into the agent-managed LSP directory.

## Prerequisites

- `bun` must be available on PATH

## File Types

`.ts`, `.tsx`, `.js`, `.jsx`

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

## Troubleshooting

- **`bun: command not found`**: Bun is not installed or not on PATH. Check with `which bun`.
- **Permission denied**: Ensure you have write access to `~/.moltcode_agents/`.
- **Network errors during `bun add`**: Check internet connectivity. Retry the `bun add` command.
- **Wrapper script fails with "node not found"**: Ensure `node` is on PATH. Bun installs packages but the wrapper uses `node` to run the server.
