# Agent Verifier — install for agents

Agent Verifier is a **remote** MCP server. There is nothing to install, clone or run locally: point your MCP client at the hosted endpoint and the `verify_numbers` tool is available immediately.

## What it does

Independent arithmetic verification for AI agents. Submit a finished table (markdown, CSV, TSV), cost breakdown, invoice, quotation or budget; the service recomputes every row total, column total and subtotal two independent ways, lists each mismatched cell with the recomputed value and difference, and returns a signed verification receipt (verification ID, UTC time, SHA-256 content hash, and a public verification URL anyone can use to confirm the check happened). It checks arithmetic consistency only — it does not verify source data, and it does not build new calculations for you. Tables may include formulas: supported formulas (`+ − * /`, `SUM`, `AVERAGE`, `MIN`, `MAX`) are evaluated for the arithmetic check, and every formula is audited for pattern deviations, hardcoded values among formulas and broken references, returned as advisory findings. Common international formats are accepted: accounting negatives in parentheses, currency symbols, semicolon-delimited tables with decimal commas, and percent values. Anyone holding a receipt can confirm the check at its verification URL, or test a table against the receipt via `POST /v1/check-receipt`. A real .xlsx workbook can also be audited as a file (HTTP only): POST the raw bytes to `POST /v1/verify-file` (≤ 2 MB, ≤ 20,000 cells) and every evaluable formula is recomputed and compared with the value stored in the file — stale or wrong stored values are flagged with cell and difference — alongside the formula audit per sheet, cross-foot totals, structure health checks and recalc-fragility advisories. It is not a real Excel engine: formulas outside the supported subset are noted as unevaluable, not evaluated. The receipt fingerprint for a file audit is the SHA-256 of the file itself.

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
- Excel file audit (HTTP only, the MCP tool takes text): `POST https://agent-verifier.agent-verifier.workers.dev/v1/verify-file` with the raw .xlsx bytes as the body

## Pricing

Free: 20 checks per day per caller. After that, US$0.02 per check paid by the calling agent via the x402 protocol. Payments currently run on the Base Sepolia **test network** (test funds only — no real money yet); the free tier is fully functional.

## Privacy

Zero content retention. Submissions are processed in memory only; the tables themselves are never stored or logged. Only each receipt's fingerprint (verification ID, issue time, content SHA-256, verdict, counts) is kept, so receipts can be confirmed later. Usage counters are aggregate only: https://agent-verifier.agent-verifier.workers.dev/stats

## Listings

- Official MCP Registry: `io.github.zenoix888/agent-verifier`
- Live service and demo: https://agent-verifier.agent-verifier.workers.dev
