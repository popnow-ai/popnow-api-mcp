<p align="center">
  <a href="https://popnow.ai/api">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="assets/popnow-wordmark-light.png">
      <source media="(prefers-color-scheme: light)" srcset="assets/popnow-wordmark.png">
      <img src="assets/popnow-wordmark.png" alt="PopNow" width="280">
    </picture>
  </a>
</p>

<h1 align="center">PopNow Developer MCP Server: call any AI image, video and audio model from Claude Code, Cursor, Codex and your own code</h1>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href="https://modelcontextprotocol.io"><img src="https://img.shields.io/badge/MCP-Streamable%20HTTP-6f42c1.svg" alt="MCP Streamable HTTP"></a>
  <a href="https://www.npmjs.com/package/@popnow/mcp"><img src="https://img.shields.io/npm/v/@popnow/mcp.svg?label=%40popnow%2Fmcp" alt="npm"></a>
  <a href="https://popnow.ai/docs/mcp"><img src="https://img.shields.io/badge/docs-popnow.ai%2Fdocs-ff5a5f.svg" alt="Docs"></a>
</p>

**The PopNow Developer MCP Server is a hosted Model Context Protocol server at `https://api.popnow.ai/api/v1/mcp` for developers. Add it to Claude Code, Cursor, Codex, VS Code or your own MCP client with a PopNow API key, and your agent can create images, videos, music and voiceovers, run any model by its API ID, follow tasks and install agent skills over one connection.**

There is nothing to host. Every paid call is quoted in USD first and charged to your [API balance](https://popnow.ai/docs/billing) only after you approve it. The key's model allow-list, IP allow-list and spend caps apply exactly as on the [PopNow REST API](https://popnow.ai/docs/quickstart).

This repository contains the setup guide, copy-paste client configs in [`examples/`](examples), an [MCP registry manifest](server.json), and [`@popnow/mcp`](packages/popnow-mcp), a small stdio bridge for clients that only start local servers.

> Want PopNow inside the ChatGPT or Claude apps with your PopNow account and website credits? Use [popnow-mcp](https://github.com/popnow-ai/popnow-mcp) and [popnow.ai/mcp](https://popnow.ai/mcp) instead.

## Contents

- [At a glance](#at-a-glance)
- [Quickstart](#quickstart)
- [Connect your client](#connect-your-client): [Claude Code](#claude-code) · [Cursor](#cursor) · [VS Code](#vs-code) · [Windsurf](#windsurf) · [Codex](#codex) · [Cline](#cline) · [Claude Desktop and stdio-only clients](#claude-desktop-and-stdio-only-clients) · [Other clients](#other-clients) · [From your own code](#from-your-own-code)
- [Tools](#tools)
- [Quotes and approval in USD](#quotes-and-approval-in-usd)
- [Results and long jobs](#results-and-long-jobs)
- [Limits](#limits)
- [Moving from the old address](#moving-from-the-old-address)
- [The stdio bridge (`@popnow/mcp`)](#the-stdio-bridge-popnowmcp)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

## At a glance

| | |
| --- | --- |
| Endpoint | `https://api.popnow.ai/api/v1/mcp` |
| Transport | Streamable HTTP, stateless (`POST`; `GET` and `DELETE` return `405`) |
| Authentication | `Authorization: Bearer sk-popnow-...` with a key from the [API console](https://popnow.ai/api/console/keys). There is no OAuth sign-in on this endpoint |
| Billing | USD from your API balance. Every paid call returns a quote first and runs only with `max_usd` |
| Key rules | The key's model allow-list, IP allow-list and daily, monthly and lifetime spend caps |
| Tools | Media tools, `create_task` for any model ID, tasks, balance, saved recipes and agent skills ([full list](#tools)) |
| Server name | `popnow-api` |

## Quickstart

1. **Create an API key** in the [API console](https://popnow.ai/api/console/keys). Giving your agent a key of its own with a low daily cap is a good habit: you can revoke it without touching your other integrations. If your API balance is $0, add funds in [API billing](https://popnow.ai/api/console/billing).
2. **Put the key in your environment.** The configs below read it from `POPNOW_API_KEY`, so it stays out of files you commit:
   ```bash
   export POPNOW_API_KEY="sk-popnow-..."
   ```
3. **Add the server.** In Claude Code it is one command:
   ```bash
   claude mcp add --transport http popnow-api https://api.popnow.ai/api/v1/mcp \
     --header "Authorization: Bearer $POPNOW_API_KEY"
   ```
4. **Ask for something.** Ask your agent for an image, a clip or a soundtrack. It picks a PopNow tool, shows you the USD price and runs it once you approve.

## Connect your client

Every client below talks to the same remote server with an API key. Ready-made files are in [`examples/`](examples).

### Claude Code

```bash
claude mcp add --transport http popnow-api https://api.popnow.ai/api/v1/mcp \
  --header "Authorization: Bearer $POPNOW_API_KEY"
```

Your shell fills in `$POPNOW_API_KEY` when you run the command, and Claude Code stores the header in your local MCP settings. To share the server with a team through a project's `.mcp.json`, use the [generic config](#other-clients) instead: Claude Code expands `${POPNOW_API_KEY}` from each person's environment.

### Cursor

Add PopNow to `.cursor/mcp.json` in your project, or to `~/.cursor/mcp.json` for every project ([`examples/cursor.mcp.json`](examples/cursor.mcp.json)):

```json
{
  "mcpServers": {
    "popnow-api": {
      "url": "https://api.popnow.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer ${env:POPNOW_API_KEY}" }
    }
  }
}
```

### VS Code

VS Code names the top-level key `servers`. This config asks for the key once and stores it securely ([`examples/vscode.mcp.json`](examples/vscode.mcp.json)):

```json
{
  "inputs": [
    { "type": "promptString", "id": "popnow-api-key", "description": "PopNow API key (sk-popnow-...)", "password": true }
  ],
  "servers": {
    "popnow-api": {
      "type": "http",
      "url": "https://api.popnow.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer ${input:popnow-api-key}" }
    }
  }
}
```

### Windsurf

Remote servers use `serverUrl` in `~/.codeium/windsurf/mcp_config.json` ([`examples/windsurf.mcp_config.json`](examples/windsurf.mcp_config.json)). Press **Refresh** in the MCP panel after saving.

```json
{
  "mcpServers": {
    "popnow-api": {
      "serverUrl": "https://api.popnow.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer ${env:POPNOW_API_KEY}" }
    }
  }
}
```

### Codex

Edit `~/.codex/config.toml` ([`examples/codex.config.toml`](examples/codex.config.toml)); `bearer_token_env_var` reads the key from your environment:

```toml
[mcp_servers.popnow-api]
url = "https://api.popnow.ai/api/v1/mcp"
bearer_token_env_var = "POPNOW_API_KEY"
```

### Cline

Open **MCP Servers → Configure** and add PopNow to `cline_mcp_settings.json`. Set `type` to `streamableHttp` explicitly; without it Cline falls back to the legacy SSE transport ([`examples/cline_mcp_settings.json`](examples/cline_mcp_settings.json)):

```json
{
  "mcpServers": {
    "popnow-api": {
      "type": "streamableHttp",
      "url": "https://api.popnow.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer sk-popnow-..." },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### Claude Desktop and stdio-only clients

Clients that can only start local stdio servers can run the [`@popnow/mcp`](#the-stdio-bridge-popnowmcp) bridge with `npx`. From version 0.2.0 it connects to the developer endpoint by default. In Claude Desktop, edit `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`, Windows: `%APPDATA%\Claude\claude_desktop_config.json`) and restart the app ([`examples/claude-desktop.json`](examples/claude-desktop.json)):

```json
{
  "mcpServers": {
    "popnow-api": {
      "command": "npx",
      "args": ["-y", "@popnow/mcp"],
      "env": { "POPNOW_API_KEY": "sk-popnow-..." }
    }
  }
}
```

The bridge needs Node.js 18 or newer.

### Other clients

Most clients that support remote MCP servers accept this shape. It is also the format of a Claude Code project `.mcp.json` ([`examples/generic.json`](examples/generic.json)). Check how your client reads environment variables before you paste a key into a file that is committed anywhere.

```json
{
  "mcpServers": {
    "popnow-api": {
      "type": "http",
      "url": "https://api.popnow.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer ${POPNOW_API_KEY}" }
    }
  }
}
```

### From your own code

With the official Python SDK (`pip install mcp`):

```python
import asyncio, os

from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def main():
    headers = {"Authorization": f"Bearer {os.environ['POPNOW_API_KEY']}"}
    async with streamablehttp_client("https://api.popnow.ai/api/v1/mcp", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()
            result = await session.call_tool("check_balance", {})
            for part in result.content:
                print(part.type, getattr(part, "text", ""))

asyncio.run(main())
```

With the official TypeScript SDK (`npm install @modelcontextprotocol/sdk`):

```javascript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js';

const transport = new StreamableHTTPClientTransport(new URL('https://api.popnow.ai/api/v1/mcp'), {
  requestInit: { headers: { Authorization: `Bearer ${process.env.POPNOW_API_KEY}` } },
});
const client = new Client({ name: 'my-app', version: '1.0.0' });
await client.connect(transport);

const result = await client.callTool({ name: 'check_balance', arguments: {} });
console.log(result.content[0].text);
await client.close();
```

Or with nothing but `curl`. The server is stateless, so every request stands alone; send both media types in `Accept`. This call asks for a quote and charges nothing:

```bash
curl -X POST "https://api.popnow.ai/api/v1/mcp" \
  -H "Authorization: Bearer $POPNOW_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc": "2.0", "id": 1, "method": "tools/call", "params": {"name": "create_task", "arguments": {"model": "google/nano-banana-2.1/text-to-image", "input": {"prompt": "A paper boat on a rainy window sill, soft morning light", "aspect_ratio": "4:5"}}}}'
```

For production code that calls models directly, the [REST API](https://popnow.ai/docs/quickstart) gives you more control: webhooks, idempotency keys and per-task retention. SDKs: [Python](https://github.com/popnow-ai/popnow-python) · [JavaScript / TypeScript](https://github.com/popnow-ai/popnow-js) · [Go](https://github.com/popnow-ai/popnow-go) · [Java](https://github.com/popnow-ai/popnow-java) · [PHP](https://github.com/popnow-ai/popnow-php).

## Tools

Typical prices are for the default model and settings, in USD. Every paid tool quotes the exact amount before it runs. The full parameter reference is in the [developer MCP docs](https://popnow.ai/docs/mcp#parameters).

| Tool | What it does | Default | Typical price |
| --- | --- | --- | --- |
| `create_image` | A new picture from a description | Nano Banana 2.1, 1K, square | about $0.048 |
| `edit_image` | Change a photo with words: style, outfit, background, add or remove things | Nano Banana 2.1 | about $0.048 |
| `create_video` | A short video from a description, or a photo brought to life | MiniMax H3, 768p, 5 seconds | about $0.185 |
| `create_music` | An original song or instrumental | Suno v6 | about $0.072 |
| `create_speech` | Text read aloud in many voices and languages | Gemini 3.8 Flash TTS, voice Kore | a fraction of a cent, by length |
| `apply_effect` | Turn a photo into an effect video or a photo makeover | The effect's own model | $0.125 to $0.34 for video, $0.03 to $0.05 for photos |
| `remove_background` | Cut out the subject as a transparent PNG | Background Remover | $0.0048 |
| `upscale` | Make a photo or a video sharper and bigger | Photo 4K, video 1080p | about $0.012 for a photo; video by length |
| `add_sound` | Sound effects and ambience that match a silent video | The tool's own model | quoted first |
| `extend_video` | Continue a video beyond its last frame | `minimax-h3` | quoted first |
| `edit_video` | Edit a video with words while keeping its motion | `kling-o3` | quoted first |
| `copy_motion` | Animate a character picture with motion from a video | `kling-3-0` | quoted first |
| `create_video_from_references` | A video that keeps subjects from reference pictures | `minimax-h3` | quoted first |
| `transcribe_audio` | Turn recorded speech into text | `grok-stt` | quoted first |
| `create_sound_effects` | Sound effects, ambience and spoken lines from a description | `seed-audio-1-0` | quoted first |
| `split_layers` | Separate a picture into editable transparent PNG layers | `seedream-5-0-pro` | quoted first |
| `erase_object` | Remove an object or person and fill the gap | `object-eraser-v1` | quoted first |
| `expand_image` | Extend a photo beyond its edges | `image-expander-v1` | quoted first |
| `change_camera_angle` | Show the scene from a different viewpoint | `camera-angle-changer-v1` | quoted first |
| `change_pose` | Match a person's pose to a reference picture | `pose-changer-v1` | quoted first |
| `relight_photo` | Change the lighting in a photo | `relight-v1` | quoted first |
| `remove_shadow` | Remove harsh shadows from a photo | `shadow-remover-v1` | quoted first |
| `anime_to_real` | Turn an anime or cartoon picture into a photorealistic image | `anime-to-real-v1` | quoted first |
| `compose_video` | Cut earlier results into one finished MP4 with optional music, narration, captions and a title | | free |
| `get_job` | Check a task that is still running and return it when ready | Waits up to 30 s | free |
| `list_my_creations` | Recent tasks made with this key, with links | Last 10 | free |
| `check_balance` | Available API balance and funds held for running tasks, in USD | | free |
| `list_models` | Image, video and audio models with their exact API IDs, or every effect; with `model`, that model's inputs | image | free |

Only tools that expose `model` accept a model from `list_models`; omit it to use the default. All tools carry the standard MCP hints (`readOnlyHint`, `destructiveHint`, `openWorldHint`), and no tool deletes anything.

**Developer tools.** Four tools exist only on this endpoint:

| Tool | What it does | Parameters |
| --- | --- | --- |
| `create_task` | Run any model by its exact API model ID with the model's own `input`, like `createTask` on the REST API | `model` (required, for example `google/nano-banana-2.1/text-to-image`), `input`, `max_usd`, `currency` |
| `get_task` | Same as `get_job`: check a task and collect its result | `job_id` (required), `wait_seconds` (0 to 50, default 30) |
| `list_tasks` | Same as `list_my_creations`: recent tasks made with this key | `kind` (`image`, `video`, `audio`), `limit` (1 to 20, default 10) |
| `install_skill` | The agent skill you picked: playbook, short-lived download link and install commands for your agent | `slug` (required), `agent` |

Call `list_models` with a `category` to see every model's exact IDs, then with `model` for that model's inputs. `create_task` validates `input` against the model's schema before it quotes, so a missing or unknown field comes back as a message, not a charge.

**Saved recipes and skills.** `list_my_recipes`, `run_my_recipe`, `get_recipe_run`, `resume_recipe_run`, `list_my_skills` and `use_my_skill` reach the recipes, canvas workflows and skills you saved on [popnow.ai](https://popnow.ai), with the same USD quote and approval. See [Your recipes, workflows and skills](https://popnow.ai/docs/saved-recipes).

**Agent skills.** `find_skills` returns a few [agent skills](https://popnow.ai/api/skills) that match a request (preview video, cover image and what each needs); `install_skill` returns the chosen skill's `SKILL.md`, a package link that works for a few minutes, the install commands for your agent and which PopNow tools make its media. Both are free and read-only. More skills: [popnow-skills](https://github.com/popnow-ai/popnow-skills).

## Quotes and approval in USD

1. **Quote.** Call a paid tool without `max_usd`. Nothing starts and nothing is held. The result has `needs_confirmation: true`, `currency: "USD"`, `cost_usd` and `available_usd`.
2. **Approve.** Show the price to the user. After they approve, call the tool again with the same arguments plus `max_usd` set to the quoted `cost_usd`.
3. **Run.** If the current price is within `max_usd`, the task starts. If it went up, you get a new quote to approve instead; nothing is charged.

`max_usd` is a nonnegative decimal **string** with up to nine decimal places, such as `"0.012345678"`. JSON numbers, exponent notation and negative values are rejected. `currency` is optional and can only be `USD`. Every paid call needs this approval, including a zero-price quote; there is no automatic approval threshold.

When the balance cannot cover a quote, the tool says "Not enough API balance" with the cost and what is available, and charges nothing. Tasks follow the same rules as REST tasks: the highest possible price is held when a task starts, you pay what it actually used (never more than the hold), and a failed or expired task releases its hold. Identical calls with the same key within ten seconds create one task, so an agent that retries does not pay twice.

```text
You:    Make a 9:16 clip of a vintage car cruising along the coast at golden hour, 10 seconds.
Agent:  (calls create_video → "Needs confirmation, nothing started yet. This costs $0.37 USD; API balance available: $12.40 USD.")
        That's $0.37 from your PopNow API balance. Go ahead?
You:    Yes.
Agent:  (calls create_video again with max_usd: "0.37" → job_id, then get_task)
        Done: coast.mp4. It's in your API logs too.
```

## Results and long jobs

- Images come back as a preview plus a download link. Videos and audio come back as links.
- Download links are signed and expire after a while. Each result also has a `popnow://g/<job>/<n>` link that never expires and can be passed to another tool, so "now animate that picture" just works.
- A tool waits up to about 40 seconds for the result. Video usually takes longer, so the reply carries a `job_id` and the agent calls `get_task` to collect it.
- Every task is listed in the API console's [logs](https://popnow.ai/api/console/logs) with its key, model and USD amount.

## Limits

| What | Limit |
| --- | --- |
| Tool calls per account (including `get_task` polling) | 240 per minute |
| New tasks per account | 20 per minute |
| `find_skills` / `install_skill` per account | 240 / 120 per hour |

A task over the limit is not charged; the tool says how many seconds to wait. The key's spend caps apply to every call. REST API limits are separate and described in [Rate limits](https://popnow.ai/docs/limits).

## Moving from the old address

Until 1 November 2026, API keys connected to `https://api.popnow.ai/mcp`. That address keeps accepting API keys until **31 January 2027 (00:00 UTC)** and serves the same developer tools, with `Deprecation`, `Sunset` and `Link` headers on every response. After that it answers API keys with `401`.

To move, change the URL in your MCP config to `https://api.popnow.ai/api/v1/mcp`. The key, the tools and USD billing stay the same. If you run the stdio bridge, upgrade to `@popnow/mcp` 0.2.0 or newer (`npx -y @popnow/mcp` picks it up), or set `POPNOW_MCP_URL` to the new address.

## The stdio bridge (`@popnow/mcp`)

[`packages/popnow-mcp`](packages/popnow-mcp) is a minimal TypeScript package (bin `popnow-mcp`) for MCP clients that can only launch local stdio servers. It connects a `StdioServerTransport` to a `StreamableHTTPClientTransport` from the official MCP SDK and forwards every message unchanged, so tools and prices always match the live server. It is published on npm as [`@popnow/mcp`](https://www.npmjs.com/package/@popnow/mcp); see [Claude Desktop and stdio-only clients](#claude-desktop-and-stdio-only-clients) for the config.

| Environment variable | Required | Description |
| --- | --- | --- |
| `POPNOW_API_KEY` | yes | PopNow API key (`sk-popnow-...`), sent as a Bearer token |
| `POPNOW_MCP_URL` | no | Remote endpoint, default `https://api.popnow.ai/api/v1/mcp` |

To build and run it from source instead (Node.js 18 or newer):

```bash
git clone https://github.com/popnow-ai/popnow-api-mcp.git
cd popnow-api-mcp
npm --prefix packages/popnow-mcp install
npm --prefix packages/popnow-mcp test    # builds with tsc, then runs an end-to-end test against a local fake server
node packages/popnow-mcp/dist/cli.js --help
```

## MCP registry manifest

[`server.json`](server.json) describes the server in the [MCP registry](https://github.com/modelcontextprotocol/registry) format (schema `2025-12-11`) as `io.github.popnow-ai/popnow-api-mcp`: the remote Streamable HTTP endpoint with a required `Authorization` header, and the `@popnow/mcp` npm package.

## Troubleshooting

| You see | Why | Fix |
| --- | --- | --- |
| `401` "This endpoint takes PopNow API keys" | No `Authorization` header, or a sign-in token instead of an API key | Send `Authorization: Bearer` with an active `sk-popnow-` key |
| `401` "This API key isn't valid" | The key is wrong, revoked or expired | Create a key in the [API console](https://popnow.ai/api/console/keys) |
| `403` or a tool error about the key | The key's model or IP allow-list blocks the call | Adjust the key's restrictions in the API console |
| *Needs confirmation, nothing started yet* | The tool is quoting first | Approve, then repeat the call with `max_usd` |
| "Not enough API balance" | The API balance cannot cover the quote | Add funds in [API billing](https://popnow.ai/api/console/billing), or choose a cheaper model or a shorter video |
| A spend-cap message | The call would go over one of the key's caps | Raise the cap or wait for it to reset |
| "Too many creations at once. Nothing was charged." | More than 20 new tasks in a minute | Wait the number of seconds in the message |
| `405 Method Not Allowed` | `GET` or `DELETE` to the endpoint; the server is stateless | Use `POST` |
| Cline connects but nothing works | `type` missing, so Cline used legacy SSE | Set `"type": "streamableHttp"` |

## FAQ

### Does the MCP server cost extra?
No. Connecting is free, and so are reading tools such as `list_models`, `get_task` and `find_skills`. Media tasks spend your API balance at the same model prices as the REST API, and every one is quoted first.

### Can I connect with OAuth instead of an API key?
No. This endpoint only takes API keys, so every call is billed to the API balance and governed by the key's restrictions and caps. To use PopNow in the ChatGPT or Claude apps with your PopNow account, see [popnow-mcp](https://github.com/popnow-ai/popnow-mcp).

### Which models can I call?
Every image, video and audio model in `list_models`, including Kling, Seedance, Wan, Vidu, MiniMax, Nano Banana, GPT Image, Flux and Suno. `create_task` takes any of their exact API IDs with the model's own inputs; the named tools cover the common jobs with sensible defaults.

### Is there anything to install?
No. The server is remote; you only add its address. Clients that can only start local stdio servers can run the bridge with `npx -y @popnow/mcp`.

### Where do my results go?
Results come back as links in the tool response. Every task also appears in the API console's logs, and `list_tasks` lists the recent ones for the key.

## Links

- Developer MCP: [popnow.ai/api/mcp](https://popnow.ai/api/mcp) · [Docs](https://popnow.ai/docs/mcp) · [API console](https://popnow.ai/api/console/keys) · [API billing](https://popnow.ai/docs/billing)
- REST API: [Quickstart](https://popnow.ai/docs/quickstart) · [API reference](https://popnow.ai/docs/api-reference) · [Models](https://popnow.ai/docs/models)
- Other PopNow developer repos: [popnow-skills](https://github.com/popnow-ai/popnow-skills) · [popnow-cli](https://github.com/popnow-ai/popnow-cli) · [popnow-python](https://github.com/popnow-ai/popnow-python) · [popnow-js](https://github.com/popnow-ai/popnow-js) · [popnow-api](https://github.com/popnow-ai/popnow-api)

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). For security issues, see [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE) © PopNow

## Model catalog update — 2026-11-03

Model IDs and input fields come from the live PopNow catalog. This release covers the new GPT chat families, Gemini Omni and Veo 3.1 video variants, GPT/Whisper transcription, TTS-1/HD, realtime voice task models, and the updated Tripo, Topaz and editing models. Fetch the selected model schema before submitting; use the normal quote, create and poll flow for task models.

Embedding tasks return `output.embeddings.data` (input `index` plus the numeric `embedding`) and measured `output.embeddings.inputTokens`, with no media files required. OpenAI-compatible embeddings use `POST /v1/embeddings`; binary speech uses `POST /v1/audio/speech`. The API wallet and quotes are in USD.

The remote developer MCP catalog now supports `list_models` categories `chat` and `utility` in addition to media. `create_task` uses the exact model ID and model-specific input; `get_task` returns vector data in structured content. Always quote first and confirm `max_usd` before a paid task. The local package forwards the remote server tools automatically.
