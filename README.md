# Agent Verifier

mcp-name: io.github.zenoix888/agent-verifier

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/zenoix888/agent-verifier)

**Independent arithmetic verification for AI agents — before they deliver their work.**

An agent submits a finished table or breakdown. Agent Verifier recomputes every total two independent ways, lists each cell that does not add up (with the recomputed value and the difference), and returns a signed verification receipt: verification ID, UTC timestamp, and a SHA-256 hash of what was checked.

- **Live service:** https://agent-verifier.agent-verifier.workers.dev/
- **Official MCP Registry:** `io.github.zenoix888/agent-verifier`
- **Status:** live prototype; payments run on the Base Sepolia test network (test funds only, no real money yet)

## What it checks

- Tables and spreadsheets (markdown, CSV, TSV)
- Cost breakdowns, invoices, quotations, budgets
- Row totals, column totals, subtotals, and line items against a stated total
- Formulas inside a submitted table: supported formulas (`+ − * /`, `SUM`, `AVERAGE`, `MIN`, `MAX`) are evaluated, so formula totals are recomputed exactly like typed numbers

## Formula audit (new in v0.2.0)

Every formula in a submitted table is also audited, and the findings come back as advisory `formulaFindings` — suspicions to review, never a change to the arithmetic verdict:

- **Pattern deviations** — a formula whose shape differs from the dominant pattern of its row or column, e.g. a SUM that quietly covers a shorter range than its neighbours
- **Hardcoded values among formulas** — a typed, dead number sitting in a row or column where the other cells are formulas
- **Broken references** — formulas that reference themselves, form a circular chain, or point outside the table or at empty cells
- **Unevaluable formulas** — functions the service does not evaluate (e.g. VLOOKUP) are noted with the reason, never guessed

## Track record

As of 12 Nov 2026: 30 verifications run for outside agents since launch on 27 Oct 2026, with errors caught in 10 of those tables. Formula audit added 12 Nov 2026. Live aggregate counters: https://agent-verifier.agent-verifier.workers.dev/stats

## What it does not check

Arithmetic consistency only. It does **not** verify that the source data itself is true — and that wording is printed on every receipt:

> Arithmetic consistency checked by two independent recomputation paths. Source data accuracy is NOT verified.

It audits finished numbers — it does not build new calculations for you. Formulas already inside a submitted table are evaluated for the arithmetic check and audited (see Formula audit above); that is different from asking it to calculate something new.

## When to use it

Use Agent Verifier whenever numbers in a finished table have to add up before the table reaches a human:

- "Check the totals in this spreadsheet / CSV before I send it"
- "Do these line items add up to the total on the invoice / quotation?"
- "Cross-foot this table: do the row totals and column totals agree?"
- "Verify the numbers in this budget / cost breakdown"
- "Audit this markdown table for arithmetic errors"

If any number does not add up, the response names the exact cell, the value it should be, and the difference — so the agent can fix it before delivery instead of after a human spots it.

## Also listed on

- Agensi: https://www.agensi.io/skills/agent-verifier
- mcpservers.org: https://mcpservers.org/servers/zenoix888/agent-verifier
- LobeHub: https://lobehub.com/mcp/zenoix888-agent-verifier
- Glama: https://glama.ai/mcp/connectors/io.github.zenoix888/agent-verifier

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
