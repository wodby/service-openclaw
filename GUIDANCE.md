# OpenClaw on Wodby

What Wodby sets up for OpenClaw on this service. It runs the `wodby/openclaw` image, which starts the OpenClaw gateway on port 18789. The port is private.

## Generated configuration

On every start the container renders the template `/etc/gotpl/openclaw.json.tmpl` to `~/.openclaw/openclaw.json` (`OPENCLAW_CONFIG_PATH`), from its environment variables. The result is rewritten on each start: never edit it. The template itself is the config file "OpenClaw config" of the service; edit it there to change anything the variables below do not cover.

## What switches features on

| Variable on the service | Effect in the generated configuration |
| --- | --- |
| `OPENAI_API_KEY` | adds the agents `openai` and `openai-code` |
| `ANTHROPIC_API_KEY` | adds the agent `claude` |
| `GEMINI_API_KEY` or `GOOGLE_API_KEY` | adds the agent `gemini` |
| `TELEGRAM_BOT_TOKEN` | enables the Telegram channel |
| `DISCORD_BOT_TOKEN` | enables the Discord channel |

The first provider present, in the order above, is the default agent. Models and workspaces of the agents have their own optional variables: `OPENCLAW_OPENAI_MODEL`, `OPENCLAW_OPENAI_CODE_MODEL`, `OPENCLAW_CLAUDE_MODEL`, `OPENCLAW_GEMINI_MODEL`, `OPENCLAW_AGENTS_WORKSPACE` and `OPENCLAW_<AGENT>_WORKSPACE`. Without a provider key the configuration has no agent list.

## Access

- The service token `gateway_token` is passed to the container as `OPENCLAW_GATEWAY_TOKEN`. Clients of the gateway and the Control UI authenticate with it.
- `OPENCLAW_GATEWAY_CONTROLUI_ALLOWED_ORIGIN_JSON` lists the origins allowed to open the Control UI: the environment's primary host, the service's address inside the cluster and the local address. It follows the primary host.
- The gateway binds to all interfaces of the container (`bind: "lan"`).

## Data

The `data` volume is mounted at `/data`, the OpenClaw state directory (`OPENCLAW_STATE_DIR`). Agent workspaces default to directories under `~/.openclaw/`, which are outside that volume unless the workspace variables point elsewhere.

## Changing configuration

Add or change environment variables on the service, or edit the "OpenClaw config" template, then deploy the service.

## Check the result

`openclaw health --json` inside the container reports `"ok": true` when the gateway is up. The container's `health` command runs this check.
