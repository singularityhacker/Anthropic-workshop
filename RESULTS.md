# Build agents that last: lab results (Tinplate Merchant Systems)

Run on 2 Nov 2026 (ET) on Justice's Claude Max subscription via Claude Code 2.1.301 OAuth, Agent SDK 0.2.176. No API key.
Cost is the lab's own token-price estimate (metrics.py prices), not money billed: on a subscription it only draws down usage limits.
Score = dashboard formula: 100 x (0.5 quality + 0.3 cheapest/cost + 0.2 fastest/time), relative to the other runs in the race.

| Step | Variant | Quality | Cost (est.) | Time | Score (race 1-5) | Score (race incl. 6) |
|---|---|---|---|---|---|---|
| 1 | old-style prompt | 7/8 (1,986 words) | $0.1548 | 49.6s | 53 | 48 |
| 2 | better prompt | 8/8 | $0.1147 | 24.0s | 67 | 57 |
| 3 | research subagent | 8/8 | $0.0498 | 34.6s | 72 | 56 |
| 4 | MCP tools | 7/8 | $0.0374 | 38.1s | 70 | 50 |
| 4 | CLI tools | 5/8 | $0.0601 | 56.6s | 48 | 35 |
| 5 | run 1: research + save notes | 8/8 | $0.0338 | 50.8s | 77 | 55 |
| 5 | run 2: write from notes | 8/8 | $0.0250 | 12.1s | 100 | 66 |
| 6a | step 5 from notes, lead effort low | 8/8 | $0.0178 | 8.4s | - | 73 |
| 6b | step 5 from notes, Haiku lead, effort low | 8/8 | $0.0019 | 10.2s | - | **96** |

## What each step changed
- **1 → 2**: dropping the shouting/template prompt for context and a goal cut the briefing from ~2,000 words to 247, halved the time, and fixed the one-screen habit check.
- **3**: moving the reading into a Haiku subagent cut cost by ~55%; the lead's context fell from ~23k to ~4k tokens. A little slower (two agents).
- **4**: small MCP tools trimmed cost again but the search tool returned only 5 passages and missed one buried fact. The CLI variant was worst: Bash tool overhead, refused commands and a 5/8 briefing.
- **5**: notes saved on run 1 made run 2 four times faster and the cheapest so far, with full quality — the guide's "memory pays off on the second run" point holds.
- **6**: once notes exist the lead only rewrites from them, so a cheap fast lead (Haiku, low effort) kept 8/8 at ~1/13th the cost.

## Adaptations for subscription auth (not in original lab)
- No `.env`; venv built by hand (`python3 -m venv .venv && pip install -r requirements.txt`). setup.py/check.py (API-key + direct anthropic calls) not used.
- `lab.py`: drops an empty/placeholder `ANTHROPIC_API_KEY` so it can't override the OAuth login; sets `ENABLE_CLAUDEAI_MCP_SERVERS=false` in the SDK env (otherwise the account's claude.ai connectors add ~19k tokens of MCP tools to every run).
- Step 5: `remember.py` uses the raw Messages API (memory tool + context-editing beta), which needs a key. When no key is set, `lab.py 5` now runs the same step through the Agent SDK: same prompts, models, effort, turn budgets, memory folder and "researcher stays home when notes exist" logic. New `memory_mcp_server.py` exposes the same three tools (memory with `safe_path` sandbox, get_company_section, read_document) over stdio MCP. Not reproducible on this path: the context-editing beta (`clear_tool_uses`) and explicit `cache_control` (Claude Code caches automatically).
- dashboard.py's `has_key` check would show "no key"; not used (CLI only).
