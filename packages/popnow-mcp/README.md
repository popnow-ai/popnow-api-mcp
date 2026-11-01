# @popnow/mcp

`@popnow/mcp` is a tiny local stdio bridge to the [PopNow](https://popnow.ai/api) developer MCP server
(`https://api.popnow.ai/api/v1/mcp`). It lets MCP clients that only start local stdio servers use PopNow's
AI image, video, music and speech tools with a PopNow API key, billed in USD from your API balance.

The bridge has no tools of its own: it forwards every MCP message to the remote server with your API key,
so the tool list, prices and confirmations always match the live service.

If your client supports remote (Streamable HTTP) servers, you do not need this package. Add
`https://api.popnow.ai/api/v1/mcp` directly with an `Authorization: Bearer sk-popnow-...` header. See the
[main README](https://github.com/popnow-ai/popnow-api-mcp#readme).

## Usage

Add it to your MCP client config; `npx` downloads and runs it (Node.js 18 or newer):

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

| Variable | Required | Description |
| --- | --- | --- |
| `POPNOW_API_KEY` | yes | A PopNow API key from [popnow.ai/api/console/keys](https://popnow.ai/api/console/keys) |
| `POPNOW_MCP_URL` | no | Override the endpoint (default `https://api.popnow.ai/api/v1/mcp`) |

`npx @popnow/mcp --help` prints the same information. Diagnostics go to stderr; stdout carries only MCP messages.

Version 0.1.0 connected to `https://api.popnow.ai/mcp`. That address accepts API keys until 31 January 2027;
0.2.0 and newer use the developer endpoint by default.

## Programmatic use

```ts
import { createBridge } from "@popnow/mcp";

const bridge = createBridge({ apiKey: process.env.POPNOW_API_KEY! });
await bridge.start();
```

## Development

```bash
npm install
npm test   # builds with tsc, then runs an end-to-end test against a local fake MCP server
```

## License

MIT
