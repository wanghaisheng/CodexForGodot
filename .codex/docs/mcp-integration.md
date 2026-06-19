# MCP Integration Guide

This repository supports three optional MCP integrations for Godot/Codex work.
They are intentionally documented as opt-in local tools: the project does not
vendor third-party MCP servers, does not commit local secrets, and does not
require every contributor to run all servers at once.

## Supported MCP servers

| Server | Purpose | Default endpoint/config | When to use |
|---|---|---|---|
| `godot-mcp-enhanced` | Node-based Godot MCP server with editor/headless workflows, project inspection, screenshots, GDScript execution, validation, export, performance, and batch editing tools. | `npx -y godot-mcp-enhanced` | Use as the primary automation bridge when Codex needs to run Godot checks, inspect scenes/scripts, capture screenshots, or execute GDScript-backed validation. |
| `godot-mcp-native` | Godot editor plugin that runs an MCP server natively inside Godot without a Node server dependency. | `http://localhost:9080/mcp` | Use when the Godot editor is open and an in-editor native HTTP MCP bridge is preferred for scene/node/script operations. |
| `devspace` | Self-hosted MCP workspace bridge for ChatGPT/Codex-style local file, search, edit, shell, and worktree workflows. | `http://127.0.0.1:7676/mcp` or a private HTTPS tunnel `/mcp` URL | Use only for trusted remote/client access to approved local project roots. Treat it like giving a coding partner shell access inside the allowed workspace. |

## Recommended local Codex configuration

Add only the servers you actually run to your local Codex config. Do not commit
real tunnel URLs, passwords, bearer tokens, or machine-specific Godot paths.

```toml
# See .codex/mcp.example.toml for a copyable project example.
# ~/.codex/config.toml or another local Codex config file

[mcp_servers.godot_enhanced]
command = "npx"
args = ["-y", "godot-mcp-enhanced"]

[mcp_servers.godot_native]
type = "streamableHttp"
url = "http://localhost:9080/mcp"

[mcp_servers.devspace]
type = "streamableHttp"
url = "http://127.0.0.1:7676/mcp"
# For remote ChatGPT-style access, replace with your private tunnel URL:
# url = "https://your-tunnel-host.example.com/mcp"
```

### Port and naming policy

- Keep `godot_enhanced`, `godot_native`, and `devspace` as distinct server names
  so tool calls make the backing transport obvious.
- `godot_native` defaults to port `9080`; override only when running multiple
  Godot editor instances.
- `devspace` defaults to local port `7676`; expose it publicly only through a
  tunnel you control.

## Setup checklist

### `godot-mcp-enhanced`

1. Confirm Node.js/npm are available locally.
2. Add the `godot_enhanced` stanza from the Codex config example above.
3. If the server cannot find the correct Godot executable, configure the Godot
   path locally through the MCP server's supported mechanisms rather than
   committing machine-specific paths.
4. For a project-specific Godot version manager flow, keep the pinned engine
   version aligned with `docs/engine-reference/godot/VERSION.md`.

### `godot-mcp-native`

1. Install the Godot plugin from the Asset Library or copy the upstream
   `addons/godot_mcp` plugin into the actual game project that needs editor
   control.
2. Enable the plugin in Godot: `Project > Project Settings > Plugins`.
3. Use HTTP mode on `http://localhost:9080/mcp` for local Codex access.
4. If running headlessly, launch Godot with the plugin's MCP server flag and a
   project path, for example:

   ```bash
   godot --editor --path /path/to/project -- --mcp-server --mcp-port=9080
   ```

### `devspace`

1. Install or run the CLI with npm.
2. Initialize DevSpace and allow only the local roots that should be accessible
   to the MCP client.
3. Start the server before connecting a client.
4. Prefer local `http://127.0.0.1:7676/mcp` for same-machine use. Use a private
   HTTPS tunnel only when the client cannot reach localhost.
5. Keep the owner password private and rotate it if it is exposed.

## Security and operational guardrails

- Never commit MCP credentials, owner passwords, bearer tokens, tunnel hostnames
  for private machines, or absolute local Godot executable paths.
- Do not expose Godot MCP or DevSpace endpoints on public networks without auth,
  firewalling, and a deliberate session window.
- Prefer read/inspect/smoke-test MCP calls before write-capable scene or script
  edits.
- Record any MCP-assisted runtime evidence in the relevant lane/work-order
  report, including the server used, tool name, project path, and command/test
  result.
- If multiple MCP servers can edit the same file or scene, use only one writer
  for that task and keep the others read-only to avoid conflicting changes.

## Source links

- `godot-mcp-enhanced`: https://github.com/wgt19861219/godot-mcp-enhanced
- `devspace`: https://github.com/Waishnav/devspace
- `godot-mcp-native`: https://github.com/yurineko73/Godot-MCP-Native
