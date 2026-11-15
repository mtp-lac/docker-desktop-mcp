# Docker Desktop MCP Toolkit

Connects DeepSeek Harness to the Docker MCP gateway that Docker Desktop runs.

## What it does

The bundle inserts the installed `@deepseek-ai/dsh-mcp-client` plugin with a stdio
transport that spawns the Docker MCP gateway:

```
docker mcp gateway run --profile <profile>
```

The tools that the gateway exposes are registered under the server name
`desktop-commander`, so they appear to the agent as
`mcp__desktop-commander__<tool>`.

This is the same shape Docker Desktop's own MCP client integration writes
(for example into Gemini's `settings.json`), so the profile selection,
secrets and OAuth handling stay in Docker Desktop.

## Prerequisites

- Docker Desktop is running, with the **MCP Toolkit** enabled.
- At least one MCP server is enabled in a profile. Check with
  `docker mcp profile list` and `docker mcp profile server ls`.
- `docker` is on `PATH` for the Harness process. If it is not, replace
  `command: docker` in `cordis.patch.yml` with the absolute path to
  `docker.exe`.

## Choosing a profile

`cordis.patch.yml` pins `terminal_control`. To use a different profile, change
the value after `--profile`, or drop the flag pair entirely to use Docker's
default profile.

## Path mounting

The gateway only mounts host paths under a fixed set of temporary roots. A
server whose profile config mounts a path outside them fails to start with
`unsafe docker volume`. Grant access explicitly, for example:

```
MCP_GATEWAY_DOCKER_BIND_ALLOWED_PATHS=D:\some\dir
```

(read-only) or `MCP_GATEWAY_DOCKER_BIND_ALLOW_WRITABLE_PATHS` (read-write),
in the environment of the process that launches the Harness.

## Turning it off

Disable the plugin, or this bundle, in **Plugins**. The gateway process stops
and the `mcp__desktop-commander__*` tools disappear.

## Limitations

- Tools act with your Docker Desktop permissions.
- Startup can be slow while images are verified or pulled, so the bundle
  allows 180 s to connect and 120 s per tool call.
- `failOnStartupError` is `false`, so the Harness still starts when Docker
  Desktop is closed; the client retries and connects once the gateway answers.
