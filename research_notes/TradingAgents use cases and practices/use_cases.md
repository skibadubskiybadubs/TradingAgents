# TradingAgents: Real-World Applications and Use Cases (as of Nov 2026)

Researched 2026-11-16. About 15 tool calls. Most third-party claims come from tool summaries of fetched pages and are flagged where they could not be checked against a primary source.

## What concrete applications do users report (research assistant, signal generator, auto-trading, portfolio review, etc.)?

### Takeaway
In practice, people mostly use TradingAgents as an LLM research desk for one ticker. It produces a multi-perspective analyst report and a BUY/SELL/HOLD decision. Around that core, users build:
- signal generators for paper-trading bots, mainly on Alpaca;
- backtests over ticker and date grids;
- crypto analysis using Yahoo-style tickers such as BTC-USD;
- tutorials and teaching material.

The upstream project itself executes only against a simulated exchange.

### Cited Findings
- **Core function.** Specialized agents analyze a ticker and debate toward a decision: fundamentals, sentiment, news and technical analysts, then bull/bear researchers, a trader, a risk team and a portfolio manager. The repo calls itself a "multi-agent trading framework." — [GitHub README (fetched Nov 2026)](https://github.com/TauricResearch/TradingAgents)
- **Built-in features (README, Nov 2026)**
  - `run_backtest` runs the pipeline over a grid of tickers and dates. It scores decisions by realized alpha against a regional benchmark, from Python or the CLI, and interrupted sweeps can resume.
  - Persistent memory log of past decisions, checkpoint resume, portfolio-aware runs, Docker support and HTML report output.
  - Approved orders go to a "simulated exchange." There is no brokerage integration upstream.
  - Source: [GitHub README](https://github.com/TauricResearch/TradingAgents)
- **Data sources (README)**
  - Yahoo Finance, covering most global markets and crypto.
  - SEC EDGAR filings, dated as filed.
  - FRED macro data and Alpha Vantage.
  - Polymarket, plus StockTwits and Reddit sentiment.
  - Source: [GitHub README](https://github.com/TauricResearch/TradingAgents)
- **LLM backends (README)**
  - Hosted providers: OpenAI, Gemini, Claude, Grok, DeepSeek, Qwen, GLM, MiniMax, Kimi, Mistral, Groq and NVIDIA NIM.
  - Routers and clouds: OpenRouter, Azure and Bedrock.
  - Local models through Ollama, vLLM, LM Studio or llama.cpp.
  - v0.7.0 (Oct 2026) added "subscription-based runs on ChatGPT Plus or Pro plans."
  - Taken together, this suggests hobbyists drive usage and want to keep cost down.
  - Source: [GitHub README](https://github.com/TauricResearch/TradingAgents)
- **Release pace.** v0.2.0 shipped in Feb 2026 and v0.7.1 in Oct 2026. v0.7.1 "scor[es] decisions on final closes" and fixes portfolio, price, SEC-statement and report bugs. — [GitHub README](https://github.com/TauricResearch/TradingAgents)
- **Popularity.** About 122.8k stars and 23.6k forks (Nov 2026). Older directory listings show 85k–92k stars, which is consistent with continued growth. — [GitHub](https://github.com/TauricResearch/TradingAgents); [search snippet listings via dev.co](https://dev.co/ai/frameworks/tradingagents)
- **Single-ticker sessions and cost (Pinggy blog, updated Aug 27, 2026)**
  - Typical use is single-ticker analysis sessions. The article frames it as a "research and learning tool."
  - One full ticker analysis costs about $0.30–$0.50 at GPT-5 rates. Daily runs across 20 stocks add up.
  - DeepSeek or local Ollama cuts cost by roughly 80–90%.
  - Alpha Vantage's free tier allows only 25 calls per day.
  - Source: [Pinggy blog](https://pinggy.io/blog/best_ai_trading_agents/)
- **Research depth and crypto tickers.** Tutorials describe shallow, medium and deep "research depth" options in the CLI. Crypto is entered with Yahoo symbols such as BTC-USD and ETH-USD. — [PickMyTrade tutorial "Build an AI Hedge Fund" (2026)](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/); [DigitalOcean tutorial](https://www.digitalocean.com/community/tutorials/tradingagents)
- **Education and hosting tutorials.**
  - DigitalOcean has a tutorial and a conceptual article. Its tutorial needs a free FinnHub account and a paid OpenAI key.
  - Union.ai has a "multi-agent trading simulation" tutorial.
  - Olares documents TradingAgents as a self-hosted use case and notes it does not provide certified financial advice.
  - Sources: [DigitalOcean](https://www.digitalocean.com/resources/articles/tradingagents-llm-framework); [Union.ai](https://www-new.union.ai/docs/v2/union/tutorials/trading-agents/page.md); [Olares docs](https://docs.olares.com/use-cases/tradingagents.html)
- **Signal generator for paper bots.**
  - A lablab.ai hackathon submission runs "unmodified" TradingAgents to produce a BUY/SELL/HOLD signal with a confidence score. That signal drives Alpaca paper trading.
  - An "Alpaca Agent" demo pairs TradingAgents signals with options structuring and deterministic risk gates. It trades paper options only.
  - Sources: [lablab.ai submission](https://lablab.ai/submissions/mwrrnsa0sl2eetxq516jhvew); [Alpaca Agent demo](https://16.112.188.138.sslip.io/) (code not verified)
- **Claude Code skill.** A "tradingagents-analysis" skill for A-shares (kylinmountain/tradingagents-ashare) is listed on a Claude skills marketplace. TradingAgents-style analysis is being packaged as an agent skill. — [claudemarketplaces.com](https://claudemarketplaces.com/skills/kylinmountain/tradingagents-ashare/tradingagents-analysis)

### Inferences
- The dominant pattern is "AI analyst report + decision" for discretionary review. Fully autonomous execution is rare. Most automation is paper trading.
- The cost per ticker ($0.30–$0.50) and data-API limits push people toward watchlist-sized daily runs and cheaper or local models. They do not run wide-universe screening.

### Gaps
- I found no surveys or usage statistics on what share of users run research, backtests or bots.
- I could not find substantive Reddit (r/algotrading, r/LocalLLaMA) or Hacker News threads through search. Community sentiment from those venues is under-sampled.
- Portfolio review: the README mentions "portfolio-aware runs," but I found no user reports describing how they use it.

## What notable forks, extensions, or products build on it?

### Takeaway
There is a large derivative ecosystem. The biggest is TradingAgents-CN, a Chinese-market fork with about 33k stars, a FastAPI+Vue web app, a paid Pro edition and a mixed open/proprietary license. Others include:
- Alpaca-integrated bot forks: AlpacaTradingAgent and TradingCrew;
- crypto forks;
- A-share forks;
- web UI packages;
- products "adapted from" its workflow, such as TradingGoose.

Tauric Research's own follow-up is Trading-R1, a reasoning LLM trained with reinforcement learning.

### Cited Findings
- **TradingAgents-CN (hsliuping)**
  - What it is: a Chinese-enhanced fork for learning and research. Features:
    - single-stock debate analysis;
    - natural-language market research;
    - stock screening and a stock-selection assistant;
    - an assistant with web search, tools and memory, plus a "skill center";
    - a learning center with courses and paper trading.
  - Data: Tushare, AKShare and BaoStock.
  - Stack: v3.0 uses FastAPI + Vue 3, MongoDB and Redis.
  - Size: about 33.0k stars and 6.9k forks.
  - Versions: v2.1 repositioned toward compliance-oriented "research assistance."
  - Pro edition: adds an agent workshop, a trading system, scheduled and batch analysis, and report export under a commercial license.
  - License: Apache-2.0 for `tradingagents/`, `cli/`, `docs/` and `tests/`. `app/`, `frontend/` and `core/` are source-available proprietary, allowing personal and educational use but no commercial use or redistribution. Contributors sign a CLA.
  - Disclaimer: no target prices, stop-loss levels or buy/sell timing.
  - Source: [GitHub hsliuping/TradingAgents-CN](https://github.com/hsliuping/TradingAgents-CN)
  - Market coverage conflict: secondary sources say it covers A-shares, Hong Kong and US stocks, while the fetched README summary mentioned only A-shares. See [ossinsight](https://ossinsight.io/analyze/hsliuping/TradingAgents-CN) and [aitoolly (Mar 24, 2026)](https://aitoolly.com/ai-news/article/2026-03-24-tradingagents-cn-a-specialized-multi-agent-large-language-model-framework-for-chinese-financial-trad).
- **Other Chinese-market derivatives**
  - cy-Yin/TradingAgents-CN-lite, a lightweight A-share fork.
  - birdxs/tradingagents-astock, a Docker image specialized for A-shares.
  - simonlin1212/TradingAgents-astock, an A-share fork with a Streamlit web UI.
  - TradingAgents-CN-Launcher, a one-click launcher on cnb.cool.
  - Gitee mirrors.
  - Sources: [search results incl. Docker Hub](https://hub.docker.com/r/birdxs/tradingagents-astock); [sourcepulse](https://www.sourcepulse.org/projects/29855634); [cnb.cool launcher](https://cnb.cool/maplemiao/TradingAgents-CN-Launcher); [Aliyun developer article](https://developer.aliyun.com/article/1733252)
- **AlpacaTradingAgent (huygiatrng)**
  - Relation: "inspired by" TradingAgents and re-engineered for Alpaca with an end-of-day focus.
  - Agents: five analysts, adding a FRED macro analyst; bull/bear researchers; three risk managers.
  - Interface: Plotly Dash UI on port 7860.
  - Crypto and scheduling: crypto uses BTC/USD format, with a loop mode for 24/7 crypto and market-hour triggers such as 10 AM and 3 PM ET.
  - Execution: a "Trade After Analyze" auto-execution toggle with a fixed dollar amount per symbol. Paper or live is set with `ALPACA_USE_PAPER`.
  - It still describes itself as "designed for research and educational purposes."
  - Source: [dev.to guide, Jul 4, 2025](https://dev.to/aarontrng/complete-guide-using-alpacatradingagent-web-ui-for-automated-trading-3k78)
- **TradingCrew (PyPI).** Extends TradingAgents with real-time Alpaca integration, crypto, options analysis and a "production-ready" Dash web interface. — [PyPI tradingcrew](https://pypi.org/project/tradingcrew/)
- **TradingGoose / Open-TradingGoose.** A multi-agent trading platform on Supabase edge functions whose workflow is "adapted from" TradingAgents. It is a separate product, not a direct fork. — [Open-TradingGoose README](https://github.gg22.workers.dev/TradingGoose/Open-TradingGoose/blob/main/README.md)
- **Crypto forks**
  - TradingAgents-CryptoA adds CCXT exchanges (Binance, OKX, Bybit) and market, on-chain and news analysts.
  - 0x0funky/tradingagents-crypto has a Flask web UI.
  - Sources: [TradingAgents-CryptoA](https://gh.r-mj.com/OK-computer008/TradingAgents-CryptoA); [tradingagents-crypto](https://github.com/0x0funky/tradingagents-crypto)
- **Web UIs.** `tradingagents-webui` on PyPI (FastAPI + WebSocket + Vue). — [PyPI](https://pypi.org/project/tradingagents-webui/)
- **Trading-R1 (Tauric Research, with UCLA, UW and Stanford; arXiv 2509.11420, Sep 2025)**
  - A financial-reasoning LLM trained with SFT and RL on about 100k curated samples spanning 18 months.
  - The README lists "Trading-R1 v0.1.0 early access" in Nov 2026. Tauric Research's site is tauric.ai.
  - Sources: [arXiv 2509.11420](https://arxiv.org/abs/2509.11420?context=cs); [HF papers](https://huggingface.co/papers/2509.11420); [GitHub README](https://github.com/TauricResearch/TradingAgents)

### Inferences
- The Chinese ecosystem is the largest single derivative community. TradingAgents-CN shows a commercialization path: an open core plus a proprietary UI and Pro tier. It also shows regulatory caution: the "research assistance" repositioning and the refusal to give price targets or timing.
- Broker integration and dashboards happen almost entirely in forks. Alpaca is the default broker because its paper-trading API is free.

### Gaps
- I did not verify the code or maintenance status of TradingCrew, TradingGoose or the crypto forks.
- I found no evidence of institutional or commercial deployment by a regulated firm.
- I did not confirm Trading-R1 Terminal availability.

## Is it used for live trading with real money, and with what results/caveats?

### Takeaway
I found no credible, documented real-money track record. Live trading is possible only through forks such as AlpacaTradingAgent (`ALPACA_USE_PAPER=False`). Upstream only simulates.

Independent evaluations are largely negative:
- A May 2026 one-year reproduction found TradingAgents beat buy-and-hold gross but fell below it after costs.
- Critics dispute the paper's baseline numbers and flag look-ahead and temporal-contamination risks.

### Cited Findings
- **Upstream and fork behavior.** Upstream sends approved orders to a simulated exchange. AlpacaTradingAgent supports live trading by setting `ALPACA_USE_PAPER=False`, and its guide recommends starting with paper. No performance results are given, and commenters asking for them got no answer. — [GitHub README](https://github.com/TauricResearch/TradingAgents); [dev.to guide](https://dev.to/aarontrng/complete-guide-using-alpacatradingagent-web-ui-for-automated-trading-3k78)
- **Pinggy anecdote and caveats (Aug 2026)**
  - It reports one community run of about 7% over 30 days, against 4.5% for the S&P 500, with 22% drawdowns. It says the result is not repeatable and that "the TradingAgents team explicitly recommends against using real money."
  - Caveats: costs and slippage, a cautious bias in LLM agents, and news that is already priced in.
  - The run is not independently verifiable.
  - Source: [Pinggy blog](https://pinggy.io/blog/best_ai_trading_agents/)
- **"The Alpha Illusion" (arXiv 2605.16895, May 16, 2026)**
  - Setup: a one-year backtest from 2025-01 to 2026-01 with $100K, equal-weighted across TSLA, NVDA, KO, XOM and MSTR.
  - Portfolio: TradingAgents ended at $106.4K gross but $102.3K net of commission, tokens, spread and impact. Buy-and-hold ended at $104.8K. Sharpe fell from 0.43 gross to 0.22 net.
  - Per ticker: net buy-and-hold beat TradingAgents on 4 of 5 tickers. On TSLA, TradingAgents was −10.17% net against +9.56% for buy-and-hold. On KO it was −17.17% against +12.77%. On MSTR it was +16.89% against −48.31%.
  - The paper argues reported LLM-agent alpha is not evidence of deployable ability and flags temporal contamination from model training data.
  - Source: [alphaXiv 2605.16895](https://www.alphaxiv.org/abs/2605.16895.md)
- **FINSABER (arXiv 2505.07078).** It re-tests LLM timing strategies over longer periods and broader universes and finds the reported advantages shrink substantially. It lists TradingAgents among short-horizon, few-symbol studies. — [arXiv 2505.07078](https://arxiv.org/html/2505.07078v2)
- **Look-Ahead-Bench (arXiv 2601.13770, Jan 2026).** A benchmark for look-ahead bias in point-in-time financial LLMs that cites TradingAgents. — [arXiv 2601.13770](https://arxiv.org/pdf/2601.13770)
- **dev.to critique (Jul 27, 2026; not peer-reviewed)**
  - Claim: the paper reports AAPL buy-and-hold at −5.23% for 2024-06-19 to 2024-11-19. The author's recomputation from dividend-adjusted Yahoo closes gives about +9.12%.
  - It also notes that transaction costs are not modeled, and that a Sharpe of 8.21 with a 0.91% max drawdown suggests an evaluation artifact.
  - It points to GitHub Issue #203 (a look-ahead bias report), which I did not verify.
  - Note: the date range cited differs from the paper's Jan–Mar 2024 window in the v7 table. It may refer to another version or an experiment, so this is unverified.
  - Source: [dev.to](https://dev.to/trow126/the-most-starred-llm-trading-paper-claims-buy-and-hold-lost-523-it-actually-gained-912-1jj6)
- **Real-capital question.** A Hugging Face paper-page commenter asked whether the results were verified with real capital. No confirmation appears. — [HF papers 2412.20138](https://huggingface.co/papers/2412.20138)
- **dev.co review.** Says there are no performance guarantees and recommends legal and compliance review before live use. It also warns that Reddit and StockTwits sentiment can be stale or manipulated. — [dev.co](https://dev.co/ai/frameworks/tradingagents)
- **Unverified live-trading claim.** A BestHub article claims the CLI has "live-trading modes." This contradicts the README and is unverified. — [BestHub](https://besthub.dev/articles/tradingagents-a-multi-agent-llm-framework-for-financial-trading-9a93fa4bbd7e)

### Inferences
- Any real-money use is individual and undocumented. The available evidence suggests that edge after costs is unproven, or negative, against buy-and-hold.
- Upstream v0.7.1 "scoring decisions on final closes" may be a response to the evaluation criticism. That is unconfirmed.

### Gaps
- I found no audited or verifiable live P&L from any user.
- I did not read the content of GitHub Issue #203 directly.
- I did not verify the "7% in 30 days" community run.

## What does the original paper (arXiv 2412.20138) and Tauric Research say it is intended for?

### Takeaway
The paper presents TradingAgents as a research prototype for studying multi-agent LLM collaboration that mimics a trading firm, validated only by a short backtest. The repo explicitly says it "is designed for research purposes" and "is not intended as financial, investment, or trading advice." The authors list live deployment as future work.

### Cited Findings
- **Paper metadata.** Authors are Yijia Xiao, Edward Sun, Di Luo and Wei Wang, with UCLA, MIT and Tauric Research affiliations according to secondary summaries. v1 is dated Dec 28, 2024 and v7 Jun 3, 2025. Comments note an "Oral @ Multi-Agent AI in the Real World" and there is an ICML 2025 listing. Categories are q-fin.TR, cs.AI, cs.CE and cs.LG. — [arXiv abs](https://arxiv.org/abs/2412.20138); [ICML virtual](https://icml.cc/virtual/2025/49302)
- **Design.** Agents with specialized roles:
  - fundamental, sentiment and technical analysts;
  - bull and bear researchers;
  - traders with varied risk profiles;
  - a risk management team.
  - Source: [arXiv abs](https://arxiv.org/abs/2412.20138)
- **Backtest setup (v7)**
  - Period: Jan 1 to Mar 29, 2024.
  - Models: gpt-4o-mini and gpt-4o for quick thinking, o1-preview for deep thinking.
  - Call volume: about 11 LLM calls and 20+ tool calls per prediction.
  - Baselines: buy-and-hold, MACD, KDJ+RSI, ZMR and SMA. There are no LLM baselines.
  - Source: [arXiv HTML v7](https://arxiv.org/html/2412.20138v7)
- **Reported results (cumulative return)**

  | Stock | TradingAgents CR | TradingAgents Sharpe | TradingAgents MDD | Buy & Hold CR |
  |---|---|---|---|---|
  | AAPL | 26.62% | 8.21 | 0.91% | — |
  | GOOGL | 24.36% | 6.39 | — | 7.78% |
  | AMZN | 23.21% | 5.60 | — | 17.10% |

  The authors call the Sharpe "exceptionally high," above their expected range, and attribute it to few pullbacks in the period. The window is short because of LLM cost. — [arXiv HTML v7](https://arxiv.org/html/2412.20138v7)
- **Future work.** Deployment in live trading, expanded agent roles and real-time data feeds. — [arXiv HTML v7](https://arxiv.org/html/2412.20138v7)
- **Repo disclaimer.** "Designed for research purposes," "not intended as financial, investment, or trading advice." Results depend on model, temperature, data quality and other non-deterministic factors. — [GitHub README](https://github.com/TauricResearch/TradingAgents)
- **Tauric Research.** Described as an open-source financial AI research community. Its site is tauric.ai. — [Trading-R1 paper / search summaries](https://huggingface.co/papers/2509.11420)

### Inferences
- There is a gap between intent and use. The authors frame it as research, but derivatives add live-broker execution and paid tiers. Every derivative studied still repeats a research or education disclaimer.

### Gaps
- The arXiv abstract page lists no affiliations, so affiliations come from secondary sources.
- I did not read the final ~17k characters of the paper HTML, which may include the appendix and limitations.
