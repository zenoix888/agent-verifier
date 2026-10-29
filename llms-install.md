# Agent Verifier — install for agents

Agent Verifier is a **remote** MCP server. There is nothing to install, clone or run locally: point your MCP client at the hosted endpoint and the `verify_numbers` tool is available immediately.

## What it does

Independent arithmetic verification for AI agents. Submit a finished table (markdown, CSV, TSV), cost breakdown, invoice, quotation or budget; the service recomputes every row total, column total and subtotal two independent ways, lists each mismatched cell with the recomputed value and difference, and returns a signed verification receipt (verification ID, UTC time, SHA-256 content hash). It checks arithmetic consistency only — it does not verify source data, and it is not a calculator (it does not evaluate expressions).

## Connect

Add this to your MCP client configuration:

```json
{
  "mcpServers": {
    "agent-verifier": {
      "url": "https://agent-verifier.agent-verifier.workers.dev/mcp"
    }
  }
}
```

- Endpoint: `POST https://agent-verifier.agent-verifier.workers.dev/mcp` (streamable HTTP, no authentication)
- Tool: `verify_numbers` — one input, `content` (string): the finished table or breakdown text
- HTTP API alternative: `POST https://agent-verifier.agent-verifier.workers.dev/v1/verify` with JSON body `{ "content": "..." }`

## Pricing

Free: 20 checks per day per caller. After that, US$0.02 per check paid by the calling agent via the x402 protocol. Payments currently run on the Base Sepolia **test network** (test funds only — no real money yet); the free tier is fully functional.

## Privacy

Zero retention. Submissions are processed in memory only and are never stored or logged. Usage counters are aggregate only: https://agent-verifier.agent-verifier.workers.dev/stats

## Listings

- Official MCP Registry: `io.github.zenoix888/agent-verifier`
- Live service and demo: https://agent-verifier.agent-verifier.workers.dev
