# TradingAgents: Community Best Practices for Setup and Operation (as of 2026-11-16)

Source note: Primary sources are the local checkout of TauricResearch/TradingAgents (README, CHANGELOG through v0.7.1 dated 2026-10-27, `tradingagents/default_config.py`, `cli/`), plus GitHub issues read via the web. Reddit/YouTube searches surfaced almost no first-hand TradingAgents threads; third-party blogs exist but several contain outdated or inaccurate details (flagged below). Local-repo facts are cited to the GitHub repo URL, since the checkout mirrors it.

## Q1. Which LLM providers/models do users recommend, and what does a run typically cost and take in time?

### Takeaway
Use a two-tier setup: a cheap/fast "quick" model for the analysts, researchers, debaters and trader, and a strong reasoning model only for the Research Manager and Portfolio Manager (since v0.6.0 each tier can be on its own provider). A run is about 13 to 22 LLM calls. Third-party estimates run from roughly $0.10 to $5 per ticker, and from a few cents on DeepSeek Flash. Runtime is roughly 5 to 10 minutes per run, but only one blog gives that figure and nobody has measured it rigorously.

### Cited Findings
**Model tiers and defaults**
- Default config (v0.7.x): `llm_provider="openai"`, `deep_think_llm="gpt-6-sol"`, `quick_think_llm="gpt-6-luna"`, `max_debate_rounds=1`, `max_risk_discuss_rounds=1`, `max_recur_limit=100`, `max_tool_rounds=20` — [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)
- "The quick model serves the analysts, researchers, debaters and trader; the deep model serves the research and portfolio managers." Each tier can use its own provider, e.g. `deep_think_provider="anthropic"`, `deep_think_llm="claude-opus-5-5"` with the rest on OpenAI (#1440, v0.6.0) — [README](https://github.com/TauricResearch/TradingAgents#readme)
- Supported providers (Nov 2026): OpenAI, Google, Anthropic, xAI, DeepSeek, Qwen (intl + CN), GLM, MiniMax, OpenRouter, Mistral, Kimi, Groq, NVIDIA NIM, Ollama, Azure OpenAI, AWS Bedrock, any `openai_compatible` endpoint (vLLM, LM Studio, llama.cpp), and a ChatGPT Plus/Pro subscription (`llm_provider="chatgpt"`, v0.7.0, 2026-10-16). The subscription "does not accept `temperature` or `max_tokens`" and counts toward plan usage limits — [README](https://github.com/TauricResearch/TradingAgents#readme); [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)
- Reasoning knobs per provider: `openai_reasoning_effort`, `google_thinking_level`, `anthropic_effort`. Each is settable by env var for non-interactive runs — [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)
- Blog (Apidog, 2026-05-07) on the minimum viable models for debate roles: "DeepSeek V4 Flash, Qwen 3.6 32B, GPT-5.5, and Claude 4.5". It says small models "loop without resolving", that DeepSeek V4 Flash in thinking mode matches GPT-5.5 debate quality at lower cost, and that thinking mode roughly doubles DeepSeek cost — [Apidog blog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/)
- Blog (PickMyTrade, 2026-05-13) recommends a cheap model for analysts and a premium model for the trader, claiming 60-80% savings (unverified). For local use it suggests Qwen 2.5 14B over Llama 3.1 8B. OUTDATED: the article also recommends GPT-4o / Claude 3.5 Sonnet and says Finnhub is required, which no longer matches the codebase — [PickMyTrade](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/)

**Local models (Ollama / llama.cpp / vLLM)**
- Ollama setup: `llm_provider="ollama"`, default endpoint `http://localhost:11434/v1`, `OLLAMA_BASE_URL` for a remote server (added v0.2.5). There is a Docker profile: `docker compose --profile ollama run --rm tradingagents-ollama`. For other local servers use `openai_compatible` plus `backend_url` — [README](https://github.com/TauricResearch/TradingAgents#readme)
- Local models need tool-calling support. A reviewer on an early Ollama PR said `llama3` "lacks support for the tool-calling features essential for TradingAgents" and recommended llama3.2, llama3.1, mistral-nemo or qwen2.5 (2025-era advice) — [PR #268 mirror](https://backiee.wasmer.app/https_git_codeproxy_net/TauricResearch/TradingAgents/pull/268)
- Hardware guidance from blogs: 16 GB VRAM for 7-13B models and 24 GB+ for 70B ([PickMyTrade](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/)). A 24 GB GPU can run DeepSeek V4 Flash or Qwen 3.6 32B, while 8 GB (Llama 5.1 8B) "quality drops" ([Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/)). Maintainers give no official hardware spec: a question about minimum requirements (#1094, 2026-06-26) got only "see the Ollama section of the README" — [Issue #1094](https://github.com/TauricResearch/TradingAgents/issues/1094)
- One real local config reported in issues: llama.cpp on a 24 GB GPU, quick = Qwen3.5-9B (no-think), deep = Qwen3.8-27B, ctx 262,144 — [Issue #1492](https://github.com/TauricResearch/TradingAgents/issues/1492); [Issue #1625](https://github.com/TauricResearch/TradingAgents/issues/1625)

**Cost and calls per run**
- Issue #750 counts 16-22 LLM calls per full pipeline run and an approximately 0% prompt-cache hit rate, because of dynamic system messages and f-string prompts. Full reports are re-sent "6-8 times per run". The author estimates 30-40% unnecessary overhead on providers that discount cached tokens. One v0.7.1 run (DeepSeek V4.1 Flash, 1 analyst, 1 debate round, 1 risk round) made about 13 calls and cost ¥0.15-0.28 (about $0.02-0.04). The issue is still open, with no maintainer reply — [Issue #750](https://github.com/TauricResearch/TradingAgents/issues/750)
- A run with 4 analysts and full debates on deepseek-v4-pro (deep) and v4-flash (quick) made about 21 LLM calls — [Issue #1390](https://github.com/TauricResearch/TradingAgents/issues/1390)
- Third-party per-ticker estimates: $0.10-$5 depending on model and rounds; the Bull/Bear debate "usually 3-5x more expensive than the analyst stage" ([Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/)). A search summary of other blogs also gives $0.20-0.80 (gpt-4o/4o-mini, 1 round), $0.50-2.00 (GPT-5.5), and says doubling rounds roughly doubles cost. These are secondary and unverified — [toolbrain review](https://toolbrain-astro.pages.dev/blog/tauric-tradingagents-review-2026/); [dev.to](https://dev.to/hassann/tradingagentsopen-source-llm-trading-framework-394e)
- Monthly budget estimates: $50-200 premium, $5-20 budget cloud, $15-50 hybrid, $0 local — [PickMyTrade](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/)
- Cost levers in config: `news_article_limit` (20), `global_news_article_limit` (10) and `global_news_lookback_days` (7) are documented as "decrease to reduce token usage". `max_tokens` caps output — [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)
- OpenAI's Flex service tier can be chosen from `.env` for cheaper, slower batch-style calls (issue #1487, closed 2026-10-05; I saw only the title) — [Issue #1487](https://github.com/TauricResearch/TradingAgents/issues/1487)
- A 2025 issue reported that raw news inflated usage to 6-7k tokens even with the top 5 articles (#291, closed) — [Issue #291](https://github.com/TauricResearch/TradingAgents/issues/291)

**Time per run**
- "Analysis: about 5-10 minutes per run, depending on the LLM" — [PickMyTrade](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/) (the only explicit figure found)
- Analysts originally ran sequentially, and a request to parallelize them with asyncio (#433, 2026-03-23) was closed. In Oct 2025 a web UI in "Deep" mode returned empty results because of long processing time (#213) — [Issue #433](https://github.com/TauricResearch/TradingAgents/issues/433); [Issue #213](https://github.com/TauricResearch/TradingAgents/issues/213)

### Inferences
- The best value is a cheap, strong quick model (DeepSeek V4 Flash, GPT-6-luna, Gemini Flash-class) with a frontier model only on the deep tier. Most calls and tokens go to the quick tier: analysts, debaters and trader.
- Budget roughly linearly with debate rounds and number of analysts. Measure one ticker on one date before a sweep (advice repeated across blogs).
- Treat local 7-9B models as quick-tier only. Use at least about 27-32B for anything that must produce structured output or resolve debates.

### Gaps
- No official or rigorously measured per-run token counts or wall-clock times exist for v0.7.x. The blog numbers predate GPT-6 pricing.
- No first-hand Reddit (r/algotrading, r/LocalLLaMA) or YouTube benchmarks were found.

## Q2. Which common errors and pitfalls appear in GitHub issues, and what are the fixes?

### Takeaway
The recurring failure classes are: data-vendor rate limits and breakage (Yahoo 429s, yfinance news feed breaks, the Alpha Vantage 25/day free cap, Reddit RSS 429s); context blow-ups from unbounded tool output on local models; structured-output and tool-call failures on thinking models and on the ChatGPT-subscription provider; and look-ahead bias at both the data level (largely fixed in v0.4.0-v0.7.1) and the model-weights level (unfixable in code). Most of these are fixed by upgrading, pinning dependency versions, and setting `llm_max_retries`, `max_tokens` and vendor fallback chains.

### Cited Findings
**Rate limits and data vendors**
- Yahoo 429 (`YFRateLimitError`) used to end the run or read as a delisted symbol, and the indicator path never retried. It was fixed 2026-09-24 (PR #1387), so a 429 is now retried and then reported as unavailable — [Issue #1386](https://github.com/TauricResearch/TradingAgents/issues/1386). An earlier backoff landed in #426 — [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)
- Yahoo rate limiting blocks all market data from data-center and non-US networks, and a keyless akshare (Sina) vendor was proposed (#1425, closed). Avoid relying on yfinance alone from cloud VMs — [Issue #1425](https://github.com/TauricResearch/TradingAgents/issues/1425)
- yfinance 1.7.0 returned company news as DATA_UNAVAILABLE on every run from about 2026-11-03. yfinance 1.8.0 reads the feed again (#1663, open 2026-11-06), so pin or upgrade yfinance — [Issue #1663](https://github.com/TauricResearch/TradingAgents/issues/1663)
- Global news can return an unrelated ticker's company news instead of macro news (#1678, open 2026-11-10) — [Issue #1678](https://github.com/TauricResearch/TradingAgents/issues/1678)
- Alpha Vantage's free tier allows 25 requests/day, and sleeps do not help. The maintainer (2026-04-25) said there is "no formal partnership with Alpha Vantage", so limits depend on your own tier. OUTDATED: older README text implied a TradingAgents-specific quota — [Issue #305](https://github.com/TauricResearch/TradingAgents/issues/305)
- Turning on Alpha Vantage news produced a 329,068-token request against a 262,144-token context (v0.7.1, llama.cpp, 2026-10-30, open) — [Issue #1625](https://github.com/TauricResearch/TradingAgents/issues/1625)
- Reddit RSS 429s: backoff, then OAuth2 (#1132), then a cross-process coordination request (#1286) and a fix for reddit.py under multiple processes and threads (#1459, closed 2026-09-30). This matters for parallel batch runs — [Issue #1286](https://github.com/TauricResearch/TradingAgents/issues/1286); [Issue #1459](https://github.com/TauricResearch/TradingAgents/issues/1459)
- Vendor config is an exact chain ("requests are NOT silently routed to vendors you didn't choose"), so list fallbacks yourself, e.g. `"yfinance,alpha_vantage"`. The defaults are: prices, indicators and news on yfinance; fundamentals `sec_edgar,yfinance`; macro `fred` (needs a free `FRED_API_KEY`); prediction markets on `polymarket` (keyless). Tool-level overrides go in `tool_vendors` — [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)
- SEC EDGAR needs no key but expects contact details. Set `SEC_EDGAR_USER_AGENT="Name email"` so SEC does not attribute your traffic to the project's default — [README](https://github.com/TauricResearch/TradingAgents#readme)
- OUTDATED vendor advice: blogs citing Finnhub, Polygon or OpenBB as the data sources ([Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/), [PickMyTrade](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/)) describe the 2024-early 2025 codebase. Current vendors are yfinance, Alpha Vantage, SEC EDGAR, FRED, Polymarket, StockTwits and Reddit — [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)

**LLM errors and limits**
- LLM 429 bursts: set `llm_max_retries` / `TRADINGAGENTS_LLM_MAX_RETRIES` (v0.3.1, #1091) — [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)
- Hangs with deepseek-v4-flash after 2026-07-31: "long reasoning chains exhaust token budget / trip gateway idle timeout" (#1204). The fix is the `max_tokens` config — [Issue #1204](https://github.com/TauricResearch/TradingAgents/issues/1204); [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)
- Context overflow from prompt accumulation (#1492): Fundamentals Analyst prompts reached 515k-627k tokens on llama.cpp with Qwen 9B/27B, because small models emit many parallel tool calls per round and statement CSVs are unbounded. Proposed fix: PR #1507 `max_tool_context_chars` (default 200k, env `TRADINGAGENTS_MAX_TOOL_CONTEXT_CHARS`), merge unconfirmed. The maintainer said a clean v0.7.1 checkout "should stay well below that size". Closed 2026-10-27. Workaround: lower `max_tool_rounds` — [Issue #1492](https://github.com/TauricResearch/TradingAgents/issues/1492)
- Structured output falls back to free text about once per run on DeepSeek thinking models: 11 fallbacks in 10 runs, mostly in the Trader and Research Manager. Ratings are recoverable but numeric entry and stop-loss fields are not. A retry-once fix is in PR #1658 (open as of 2026-11-05), which cut fallbacks from 3 to 1 across three tickers — [Issue #1390](https://github.com/TauricResearch/TradingAgents/issues/1390)
- ChatGPT-subscription provider (v0.7.1, langchain-openai 1.7.3/1.7.4): parallel tool calls arrive with `{}` arguments. 72 of 80 tool calls failed, yet the run still printed "Analysis Complete!". Workarounds are `parallel_tool_calls=False` (PR #1673) or a delta-replay shim. Open since 2026-11-09 — [Issue #1672](https://github.com/TauricResearch/TradingAgents/issues/1672)
- Early Ollama + Alpha Vantage `get_news` tool-call failure (#271, 2025-11, closed) — [Issue #271](https://github.com/TauricResearch/TradingAgents/issues/271)

**Look-ahead bias**
- Data-level fixes shipped across releases:
  - yfinance news no longer leaks future-dated or undated articles (#1007).
  - FRED returned unpublished or revised data in historical runs (#1275, fixed in v0.4.0, 2026-08).
  - The fundamentals snapshot was stamped with wall-clock `now()` (#1300).
  - Backtests were scored from close(d) although the run reads news after the close of d; they are now scored from the next close (#1518, v0.7.0).
  - Zero-unpadded dates let EDGAR serve later filings (#1488, v0.7.1).
  - Sources: [Issue #805](https://github.com/TauricResearch/TradingAgents/issues/805); [Issue #1275](https://github.com/TauricResearch/TradingAgents/issues/1275); [Issue #1518](https://github.com/TauricResearch/TradingAgents/issues/1518); [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)
- Point-in-time behaviour: SEC EDGAR serves statements as filed. For non-US filers, Yahoo statements and insider trades are withheld for past-dated runs. Social and news sources "still reflect now" — [README](https://github.com/TauricResearch/TradingAgents#readme)
- Model-level leakage (knowledge in the weights) is "a separate, fundamental limitation a code change can't fully eliminate" (maintainer, 2026-06-13). One user found 10 of 12 models showed significant recall on 3,708 earnings events (2026-09-25). Community recommendations:
  - backtest after the model's knowledge cutoff;
  - label cells that fall before the cutoff;
  - blind tickers and dates (as ai-hedge-fund did);
  - forward-test live.
  - The issue is still open. Source: [Issue #805](https://github.com/TauricResearch/TradingAgents/issues/805)
- Historical runs have no valuation (market cap, multiples) since #1300 (#1374, open) — [Issue #1374](https://github.com/TauricResearch/TradingAgents/issues/1374)

### Inferences
- Pin `yfinance` and `langchain-openai` versions in unattended deployments, and re-test after upgrades. Several November 2026 breakages came from upstream library versions, not from TradingAgents itself.
- Watch for silent degradation: a run can "complete" with most tools failing (#1672) or with structured fields missing (#1390). Check the reports for DATA_UNAVAILABLE and the fallback counts rather than trusting the exit status.
- Any backtest that predates the model's knowledge cutoff is suspect, however clean the data pipeline is.

### Gaps
- PR #1507 (tool-context budget), PR #1658 (structured retry) and PR #1673 (serial tool calls) were not confirmed merged as of 2026-11-16.
- No systematic list exists of which local Ollama models reliably pass the full pipeline in v0.7.x.

## Q3. How do users configure it for batch or scheduled runs across many tickers? (Also: debate rounds, analyst choice, reproducibility, checkpointing, memory log)

### Takeaway
For unattended use: configure entirely through `TRADINGAGENTS_*` env vars and `.env`, and pass `--ticker/--date/--analysts` to skip the prompts. Use `run_backtest` / `tradingagents backtest` for ticker-by-date grids, with built-in resume via `run_id`. Enable `--checkpoint` for crash-resume, and call `ta.settle_all_pending()` from a scheduled job to score past decisions. Keep debate rounds at 1 for sweeps, and cut analysts or pre-screen the watchlist to control cost.

### Cited Findings
**Unattended configuration**
- Every key config has an env override: `TRADINGAGENTS_LLM_PROVIDER`, `_DEEP_THINK_LLM`, `_QUICK_THINK_LLM`, `_MAX_DEBATE_ROUNDS`, `_MAX_RISK_ROUNDS`, `_MAX_TOOL_ROUNDS`, `_CHECKPOINT_ENABLED`, `_TEMPERATURE`, `_LLM_MAX_RETRIES`, `_MAX_TOKENS`, plus reasoning-effort vars. Invalid values "fail loudly at startup, not quietly misconfigure an unattended run" — [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)
- CLI flags skip the prompts: `--ticker`, `--date`, `--analysts market,news`, `--portfolio file.json`, `--checkpoint`, `--clear-checkpoints`, plus save/show/html options — [cli/main.py](https://github.com/TauricResearch/TradingAgents/blob/main/cli/main.py)
- "Research depth" in the CLI maps to Shallow=1, Medium=3, Deep=5 for both `max_debate_rounds` and `max_risk_discuss_rounds`. Env vars override the interactive choice (#977) — [cli/prompts.py](https://github.com/TauricResearch/TradingAgents/blob/main/cli/prompts.py); [cli/run.py](https://github.com/TauricResearch/TradingAgents/blob/main/cli/run.py)
- Apidog's advice to "start with `--rounds 2`" is INACCURATE for the current CLI, which has no `--rounds` flag. Use `TRADINGAGENTS_MAX_DEBATE_ROUNDS` or the config instead — [Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/) vs [cli/main.py](https://github.com/TauricResearch/TradingAgents/blob/main/cli/main.py)
- A run with no terminal (scheduled job or CI) skips the update/announcement check (v0.7.0). `TRADINGAGENTS_NO_UPDATE_CHECK=1` disables the check explicitly — [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)
- Docker: `docker compose run --rm tradingagents`, with data in the `tradingagents_data` volume or under `TRADINGAGENTS_DATA_DIR`. Keys go in `.env`, because images include no `.env` files (v0.7.1) — [README](https://github.com/TauricResearch/TradingAgents#readme)
- ChatGPT-subscription runs need a one-time `tradingagents login chatgpt`, which works in Docker. The token is stored in `~/.tradingagents/chatgpt-auth.json` — [README](https://github.com/TauricResearch/TradingAgents#readme)

**Batch runs and backtests**
- Batch grid: `run_backtest(["NVDA","AAPL"], iter_grid("2026-06-01","2026-08-01",every_n_days=7), config, selected_analysts=["market","news"])` or `tradingagents backtest NVDA,AAPL --start ... --end ... --every 7`. It writes its own memory log, never your main one. Re-running with `run_id` skips completed cells, so an interrupted sweep resumes. Each cell is scored on alpha against the regional benchmark — [README](https://github.com/TauricResearch/TradingAgents#readme)
- Two backtests started in the same second used to share a run (fixed in v0.7.1, #1549) — [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)

**Memory log and checkpointing**
- The memory log is always on: `~/.tradingagents/memory/trading_memory.md`, or `TRADINGAGENTS_MEMORY_LOG_PATH`. Decisions settle after `holding_period_days` (default 5) with a reflection. `memory_log_max_entries` caps growth. `ta.settle_all_pending()` settles decisions "for a scheduled job" without running an analysis. Settlement now visits every ticker (#1445) — [README](https://github.com/TauricResearch/TradingAgents#readme); [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py); [Issue #1445](https://github.com/TauricResearch/TradingAgents/issues/1445)
- Since v0.7.1 (#1603), backfilling older dates no longer displaces the most recent lessons. With older versions, avoid backfilling into the live memory log — [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)
- Checkpointing: per-ticker SQLite at `~/.tradingagents/cache/checkpoints/<TICKER>.db`, cleared on success. Use `--clear-checkpoints` to reset. Checkpoint resume first shipped in v0.2.4, was made graph-shape-aware in v0.3.1 and was "working CLI checkpoint resume" in v0.4.0, so earlier versions were unreliable — [README](https://github.com/TauricResearch/TradingAgents#readme)

**Scaling to many tickers**
- How users run many tickers: there is no built-in watchlist mode for live runs, so they run one ticker at a time. One user runs daily on 26 NSE large caps and proposed a deterministic pre-screen to choose which tickers get the full graph (#1665, open 2026-11-07). The author's own score had about zero rank correlation with forward returns, so it is a cost-triage tool only — [Issue #1665](https://github.com/TauricResearch/TradingAgents/issues/1665)
- Blog patterns: a 10-ticker nightly watchlist with decisions stored in Postgres; about 50 tickers per provider to compare providers; log per-run token spend because "a runaway loop in the debate phase can get expensive within minutes"; cache vendor data (4-8 vendor requests per run) — [Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/)
- Cache: `data_cache_dir` (`~/.tradingagents/cache`), one OHLCV cache file per symbol (#1330) — [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py); [CHANGELOG](https://github.com/TauricResearch/TradingAgents/blob/main/CHANGELOG.md)
- Analysts: market, sentiment (formerly "social"), news and fundamentals, selectable via `--analysts` or `selected_analysts` — [cli/main.py](https://github.com/TauricResearch/TradingAgents/blob/main/cli/main.py)

**Reproducibility**
- Runs are non-deterministic. Reasoning models (the default GPT-6 family) "vary the most" and "largely ignore temperature". For tighter reproducibility, set `temperature=0.0` and name non-reasoning models. Pinning the analysis date fixes prices and indicators, but news and social sources still reflect "now" — [README](https://github.com/TauricResearch/TradingAgents#readme)
- Pin `TYPESAFE_DEFAULT_MODEL` to a versioned ID (e.g. `jev-1.13.0`) rather than `jev-latest` when sentiment screening is on — [README](https://github.com/TauricResearch/TradingAgents#readme)
- "Treat the framework as a research scaffold ... not as a strategy with a fixed, replicable return" — [README](https://github.com/TauricResearch/TradingAgents#readme)

### Inferences
- A practical nightly recipe:
  - cron or a scheduler runs `docker compose run --rm tradingagents --ticker X --date $(date +%F) --analysts market,news,fundamentals` (or a Python loop over `propagate`), with `TRADINGAGENTS_MAX_DEBATE_ROUNDS=1`, `TRADINGAGENTS_CHECKPOINT_ENABLED=true` and `TRADINGAGENTS_LLM_MAX_RETRIES=5`;
  - stagger tickers to respect Yahoo and Reddit limits;
  - call `settle_all_pending()` once daily.
- When running in parallel, give each process its own ticker. Checkpoints are per ticker, and Reddit RSS rate limits are shared across processes.
- Dropping the sentiment and news analysts saves the most tokens and avoids the most rate-limited sources (Reddit, StockTwits, news), at the cost of coverage.

### Gaps
- The issues I read show no official cron or Airflow example and no documented safe concurrency level against Yahoo.
- The `--analysts` naming changed from "social" to "sentiment" at some point; I did not confirm the exact version.
