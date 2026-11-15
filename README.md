# Docker Desktop MCP Toolkit

Connects [DeepSeek Harness](https://github.com/deepseek-ai) (DSH) to the Docker MCP
gateway that [Docker Desktop](https://docs.docker.com/ai/mcp-catalog-and-toolkit/)
runs, so the MCP servers enabled in your Docker MCP Toolkit profile become agent tools.

## What it does

The bundle inserts the installed `@deepseek-ai/dsh-mcp-client` plugin with a stdio
transport that spawns the Docker MCP gateway:

```
docker mcp gateway run
```

Tools exposed by the gateway are registered under the server name `docker`, so they
appear to the agent as `mcp__docker__<tool>`.

This is the same shape Docker Desktop writes into its own MCP client
configurations, so profile selection, secrets and OAuth handling stay in Docker
Desktop rather than being duplicated in the bundle.

## Prerequisites

- Docker Desktop is running, with the **MCP Toolkit** enabled.
- At least one MCP server is enabled in a profile. Check with:

  ```
  docker mcp profile list
  docker mcp profile server ls
  ```

- `docker` is on `PATH` for the Harness process. If it is not, replace
  `command: docker` in `cordis.patch.yml` with the absolute path to the
  `docker` executable.

## Install

Copy this directory somewhere permanent, then in DSH open **Plugins** and use
**Add plugin** to install that directory. The profile links the checkout, so the
files stay where you put them.

## Choosing a profile

The patch runs the gateway without `--profile`, so Docker uses the default
profile. To pin a specific one, add the flag pair to `args`:

```yaml
        args:
          - mcp
          - gateway
          - run
          - --profile
          - your-profile-id
```

If you change the set of servers later, update nothing here — the gateway picks
up the profile's current contents on the next connection.

### Tool name prefix

`serverName: docker` controls the `mcp__docker__` prefix. Change it in the patch
if you prefer a different name; it must match `^[A-Za-z0-9_-]{1,32}$` and be
unique across the profile.

## Path mounting

The gateway only mounts host paths under a fixed set of temporary roots. A
server whose profile config mounts a path outside them fails to start with
`unsafe docker volume`. Grant access explicitly, for example:

```
MCP_GATEWAY_DOCKER_BIND_ALLOWED_PATHS=/path/to/dir
```

for read-only access, or `MCP_GATEWAY_DOCKER_BIND_ALLOW_WRITABLE_PATHS` for
read-write, set in the environment of the process that launches the Harness.
Prefer the narrowest path that works.

## Turning it off

Disable the plugin, or this bundle, in **Plugins**. The gateway process stops and
the `mcp__docker__*` tools disappear.

## Limitations

- Tools act with your Docker Desktop permissions.
- Startup can be slow while images are verified or pulled, so this bundle allows
  180 s to connect and 120 s per tool call.
- `failOnStartupError` is `false`, so the Harness still starts when Docker
  Desktop is closed; the client retries and connects once the gateway answers.
