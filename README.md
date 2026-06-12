# Re-running Arcade's Attio MCP Benchmark

An independent, end-to-end reproduction of [Arcade's attio-mcp-benchmark](https://github.com/ArcadeAI/attio-mcp-benchmark), which claimed Arcade's Attio toolkit uses **~100× fewer tokens** than Composio's.

**Read the full results:** [arcade-attio-benchmark-repro.vercel.app](https://arcade-attio-benchmark-repro.vercel.app) (or open [`index.html`](index.html) locally — no build needed).

## TLDR

- **Their raw data is genuine.** We re-counted their committed files (exact match: 7,426 vs 747,083 tokens) and re-ran every tool call against a freshly seeded workspace, landing within 1% per platform.
- **The 100× headline needed methodology help.** Their Composio responses were saved pretty-printed (+25% tokens), their own run gave Composio `limit: 500` while Arcade defaulted to 25 (192 vs 156 records), and Arcade's required schema-discovery calls were captured but excluded from totals.
- **Fair apples-to-apples: 50–64×.** Still a large, real gap.
- **Root cause:** Composio's Attio tool is a passthrough of Attio's raw v2 API (98.6% of the raw-API token count for identical queries). Arcade's tool post-processes: field projection, metadata stripping, flattening.

## The decomposition

| Step (one variable changed at a time) | Ratio |
|---|---|
| As published (their saved file formats) | 100.6× |
| Minify both sides | 81.7× |
| Matched record counts (192 = 192) | 63.6× |
| Count Arcade's schema-discovery calls | 50.0× |
| Counterfactual: Composio strips Attio's per-value metadata | 32.8× |
| Counterfactual: Composio adds field selection | 8.5× |

## How it was reproduced

1. Seeded a fresh Attio workspace with their `seed_workspace.py` (50 companies, 100 people, 50 deals, 26 custom attributes). One undocumented step: the script assumes 6 custom deal stages already exist; we created them via API.
2. Connected **both** Composio and Arcade to the same workspace via OAuth.
3. Replayed the exact 8 tool calls from their committed `metadata.json` on both platforms (deterministic run), plus an agent-driven run where two small-model subagents chose tools themselves — both runs converged on identical token totals.
4. Counted tokens with `tiktoken` `cl100k_base` (their encoder), then decomposed the headline one controlled variable at a time.

## Beyond the replication

The report also covers what the paper omitted:

- **Platform economics (§9):** published pricing side by side (Arcade Hobby/Growth vs Composio Free/$29/$229), all-in cost per run at three model price points (12.7× at $3/M, flipping below ≈$0.18/M), monthly TCO at the paper's own scale tiers, and what reproducing the study costs ($0 in platform fees).
- **Multi-toolkit eval pilot (§10):** tool-definition context overhead measured across Attio, Gmail, GitHub, and Slack, plus a registered protocol for extending the response benchmark beyond one CRM.

## Files

- `index.html` — the full report (live at the link above)
- `analysis.json` — per-query token counts across all variants (their committed data, our replication, matched limits, raw Attio API baseline)
- `decomposition.json` — the step-by-step waterfall totals and per-query breakdowns
- `agent-run-analysis.json` — token counts from the agent-driven (small model) runs
- `definition-tokens.json` — tool-definition token census across 4 toolkits on both platforms
- `diagrams/` — editable Excalidraw sources for the report figures

Security-related claims in Arcade's article were out of scope for this reproduction.
