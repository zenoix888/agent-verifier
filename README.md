# Agent Verifier

mcp-name: io.github.zenoix888/agent-verifier

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/zenoix888/agent-verifier)

[![M8ven Score](https://m8ven.ai/badge/mcp/zenoix888-agent-verifier-1h8879)](https://m8ven.ai/mcp/zenoix888-agent-verifier-1h8879?s=readme)

**Independent arithmetic verification for AI agents — before they deliver their work.**

An agent submits a finished table or breakdown. Agent Verifier recomputes every total two independent ways, lists each cell that does not add up (with the recomputed value and the difference), and returns a publicly checkable verification receipt: verification ID, UTC timestamp, a SHA-256 hash of what was checked, and a public verification URL that anyone can use to confirm the check happened.

- **Live service:** https://agent-verifier.agent-verifier.workers.dev/
- **Official MCP Registry:** `io.github.zenoix888/agent-verifier`
- **Status:** v0.4.3, live prototype; payments run on the Base Sepolia test network (test funds only, no real money yet)

## What it checks

- Tables and spreadsheets (markdown, CSV, TSV)
- Cost breakdowns, invoices, quotations, budgets
- Row totals, column totals, subtotals, and line items against a stated total
- Formulas inside a submitted table: supported formulas (`+ − * /`, `SUM`, `AVERAGE`, `MIN`, `MAX`) are evaluated, so formula totals are recomputed exactly like typed numbers
- Common international formats: accounting negatives in parentheses, currency symbols (HK$, US$, $, €, £, ¥, RMB, USD, HKD), semicolon-delimited tables with decimal commas, and percent values (checked when the stated total is itself a percent)
- Real .xlsx workbooks uploaded as files — the workbook itself is audited cell by cell (see "Audit an Excel file" below)

## Formula audit (new in v0.2.0)

Every formula in a submitted table is also audited, and the findings come back as advisory `formulaFindings` — suspicions to review, never a change to the arithmetic verdict in a pasted table. (In an uploaded workbook the line is stricter: error-level findings there are proven discrepancies and do count toward a MISMATCH — see "Audit an Excel file" below.)

- **Pattern deviations** — a formula whose shape differs from the dominant pattern of its row or column, e.g. a SUM that quietly covers a shorter range than its neighbours
- **Hardcoded values among formulas** — a typed, dead number sitting in a row or column where the other cells are formulas
- **Broken references** — formulas that reference themselves, form a circular chain, or point outside the table or at empty cells
- **Unevaluable formulas** — functions the service does not evaluate (e.g. VLOOKUP) are noted with the reason, never guessed

## Verify a receipt (new in v0.3.0)

Every receipt carries a public verification URL — `https://agent-verifier.agent-verifier.workers.dev/receipt/<verificationId>` — and anyone can open it, no account needed. The page shows the receipt's stored fingerprint: the verification ID, when the check was issued (UTC), the SHA-256 of the content that was checked, the verdict, the number of checks performed, and the issue and formula-finding counts. It proves that a check with that ID ran at that time and produced that verdict on content with that fingerprint.

To test a specific table against a receipt, POST it to the free `POST /v1/check-receipt` endpoint with `{ "verificationId": "...", "content": "<the exact table>" }`: the service hashes the content in memory and answers `hashMatches: true` only if it matches the receipt byte-for-byte.

What it does **not** prove: that the source data was true — the receipt covers arithmetic consistency only (see the scope wording below). And the table itself was never stored: only its fingerprint is kept, so the check can be confirmed but the content cannot be shown or recovered. Receipts exist only for checks run since v0.3.0; earlier receipts were never stored and cannot be looked up.

## Audit an Excel file (new in v0.4.0)

Instead of pasting a table, upload the real workbook: `POST https://agent-verifier.agent-verifier.workers.dev/v1/verify-file` with the raw .xlsx bytes as the request body. Limits: file ≤ 2 MB, ≤ 20,000 cells across all sheets (a larger input returns OUT_OF_SCOPE, with the exceeded limit named). Each sheet is audited:

- **Stored values recomputed** — every formula the service can evaluate is recomputed from the file's own inputs and compared with the value stored in the file; a stale or wrong stored value is flagged with its sheet, cell and the difference
- **Formula audit per sheet** — the same pattern-deviation, hardcoded-value, broken-reference and unevaluable findings as the table audit
- **Cross-foot totals** — row and column totals on each sheet's stored values are checked for adding up
- **Structure health** — duplicated cell entries, shared-string references out of range, formula cells with no stored value, sheets whose data part is missing
- **Recalc-fragility advisories** — formulas such as `DATEVALUE`/`VALUE` applied to a whole range, which may change value when Excel recalculates the file; the advisory says so if the workbook forces a full recalculation on load
- **Unverified regions refuse a verdict** — a workbook with nothing checkable returns UNABLE_TO_VERIFY, and if the totals agree but a numeric column was covered by no check, the workbook still returns UNABLE_TO_VERIFY naming the column, rather than a quiet MATCH

The receipt works the same way: its fingerprint is the SHA-256 of the file itself, checkable at the receipt's public verification URL, and the file bytes are processed in memory only, never stored.

One limit to be plain about: this is **not a real Excel engine**. Formulas inside the supported subset (`+ − * /`, parentheses, `%`, cell references, `SUM`/`AVERAGE`/`MIN`/`MAX` over ranges) are recomputed; anything richer (VLOOKUP, IF, cross-sheet references, defined names) is counted and noted as unevaluable — never evaluated by another engine and never guessed. Macros, charts, images and pivot caches are not inspected, and date-styled cells are compared by their stored serial numbers.

## Track record

As of 12 Nov 2026: 30 verifications run for outside agents since launch on 27 Oct 2026, with errors caught in 10 of those tables. Formula audit added 12 Nov 2026. Public receipt verification also went live on 12 Nov 2026 (v0.3.0) — every receipt since then carries a URL anyone can use to confirm the check happened. Excel file audit went live on 13 Nov 2026 (v0.4.0) — a real .xlsx upload is now audited cell by cell, stored values included. Live aggregate counters: https://agent-verifier.agent-verifier.workers.dev/stats

## What a verdict means

Every result states its own coverage: how many checks ran, the tolerance applied (0.005), and any numeric columns or formulas that could not be checked. A MATCH certifies only the totals that were located and recomputed — regions with no claimed total are reported as unverified, never silently passed. If a table's totals agree but a whole numeric column was covered by no check, the answer is UNABLE_TO_VERIFY naming that column; a submission with nothing checkable at all gets the same answer, never a quiet MATCH. The same discipline applies to uploaded workbooks.

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
- **Excel file audit:** `POST https://agent-verifier.agent-verifier.workers.dev/v1/verify-file` with the raw .xlsx bytes as the body (≤ 2 MB, ≤ 20,000 cells)

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

**Zero content retention.** Submissions are processed in memory only; the tables themselves are never stored or logged. The one thing kept is each receipt's fingerprint — its verification ID, issue time, the SHA-256 of what was checked, the verdict and the counts — so that a receipt can be looked up and confirmed later (see Verify a receipt above). Usage is counted in aggregate only — see the public counters at https://agent-verifier.agent-verifier.workers.dev/stats

## This repository

This repo is the public listing home for the hosted service: it holds the registry manifest (`server.json`) and this documentation. The service itself is hosted at the address above; there is nothing to install.
