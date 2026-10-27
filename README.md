# Agent Verifier

mcp-name: io.github.zenoix888/agent-verifier

**Independent arithmetic verification for AI agents — before they deliver their work.**

An agent submits a finished table or breakdown. Agent Verifier recomputes every total two independent ways, lists each cell that does not add up (with the recomputed value and the difference), and returns a signed verification receipt: verification ID, UTC timestamp, and a SHA-256 hash of what was checked.

- **Live service:** https://agent-verifier.agent-verifier.workers.dev/
- **Official MCP Registry:** `io.github.zenoix888/agent-verifier`
- **Status:** live prototype; payments run on the Base Sepolia test network (test funds only, no real money yet)

## What it checks

- Tables and spreadsheets (markdown, CSV, TSV)
- Cost breakdowns, invoices, quotations, budgets
- Row totals, column totals, subtotals, and line items against a stated total

## What it does not check

Arithmetic consistency only. It does **not** verify that the source data itself is true — and that wording is printed on every receipt:

> Arithmetic consistency checked by two independent recomputation paths. Source data accuracy is NOT verified.

It is not a calculator: it does not evaluate expressions, it audits finished numbers.

## Connect

Remote MCP server (streamable HTTP), no authentication required:

- **MCP endpoint:** `POST https://agent-verifier.agent-verifier.workers.dev/mcp` — tool: `verify_numbers`
- **HTTP API:** `POST https://agent-verifier.agent-verifier.workers.dev/v1/verify` with a JSON body `{ "content": "<your table or breakdown>" }`

Example client configuration:

```json
{
  "mcpServers": {
    "agent-verifier": {
      "url": "https://agent-verifier.agent-verifier.workers.dev/mcp"
    }
  }
}
```

## Pricing

- Free: 20 checks per day, per caller.
- After that: US$0.02 per check, paid automatically by the calling agent via the x402 protocol (currently testnet).

## Privacy

**Zero retention.** Submissions are processed in memory only and are never stored or logged. Usage is counted in aggregate only — see the public counters at https://agent-verifier.agent-verifier.workers.dev/stats

## This repository

This repo is the public listing home for the hosted service: it holds the registry manifest (`server.json`) and this documentation. The service itself is hosted at the address above; there is nothing to install.
