# TradingAgents works as a research desk, not a money-maker

**Bottom line:** People mostly use TradingAgents as an AI "research desk" for one stock at a time. It writes a report from several angles and ends with BUY, SELL or HOLD. Some people feed that answer into practice-money trading bots. No one has published a checked record of making real money with it. Independent tests over longer periods found it does about as well as simply buying and holding the stock, and worse once fees and AI costs are counted. The community's advice: run it cheaply, test it honestly, and keep a human making the final call.

## What people actually use it for

- **Single-stock research reports.** This is the main use. Several AI "analysts" (company finances, news, social mood, price charts) feed a bull-vs-bear debate, then a trader, a risk team and a manager give a final call ([GitHub](https://github.com/TauricResearch/TradingAgents)). Reviewers call it a "research and learning tool" ([Pinggy](https://pinggy.io/blog/best_ai_trading_agents/)).
- **Signals for practice-money bots.** Hackathon and demo projects turn its BUY/SELL/HOLD into trades on Alpaca's free paper (fake-money) account ([lablab.ai](https://lablab.ai/submissions/mwrrnsa0sl2eetxq516jhvew)).
- **Testing on past dates.** A built-in tool replays the full process over a list of stocks and dates. It scores each call against the market, and a sweep that stops halfway can pick up where it left off ([GitHub](https://github.com/TauricResearch/TradingAgents)).
- **Crypto.** You enter coins as Yahoo symbols such as BTC-USD ([DigitalOcean](https://www.digitalocean.com/community/tutorials/tradingagents)). Separate forks connect it to crypto exchanges ([tradingagents-crypto](https://github.com/0x0funky/tradingagents-crypto)).
- **Teaching.** Tutorials on DigitalOcean, Union.ai and Olares use it as a teaching example ([DigitalOcean](https://www.digitalocean.com/resources/articles/tradingagents-llm-framework); [Union.ai](https://www-new.union.ai/docs/v2/union/tutorials/trading-agents/page.md); [Olares](https://docs.olares.com/use-cases/tradingagents.html)).
- **The original project never places real trades.** Approved orders go only to a pretend exchange ([GitHub](https://github.com/TauricResearch/TradingAgents)).
- **Its creators call it research only.** It is "not intended as financial, investment, or trading advice." The paper lists live trading as future work ([GitHub](https://github.com/TauricResearch/TradingAgents); [arXiv](https://arxiv.org/html/2412.20138v7)).

## Spin-off projects add what the original leaves out

- **TradingAgents-CN** is a version for Chinese stocks with about 33k stars. It adds a web app, stock screening, a learning center and a paid Pro tier. It refuses to give price targets or timing ([GitHub](https://github.com/hsliuping/TradingAgents-CN)).
- **AlpacaTradingAgent** adds a dashboard, scheduled runs and auto-trading. Setting `ALPACA_USE_PAPER=False` switches it to real money. It still calls itself research and education only ([dev.to](https://dev.to/aarontrng/complete-guide-using-alpacatradingagent-web-ui-for-automated-trading-3k78)).
- **TradingCrew** adds real-time Alpaca trading, crypto and options ([PyPI](https://pypi.org/project/tradingcrew/)). **TradingGoose** is a separate platform built on the same workflow ([Open-TradingGoose](https://github.gg22.workers.dev/TradingGoose/Open-TradingGoose/blob/main/README.md)).
- **The creators' next project** is Trading-R1, an AI model trained specifically for financial reasoning ([arXiv](https://arxiv.org/abs/2509.11420?context=cs)).
- **Gap:** no regulated firm is known to use it.

## Does it make money? The evidence says no clear edge

- **The paper's headline results are thin.** They cover 3 months (Jan–Mar 2024) and 3 large stocks (AAPL, GOOGL, AMZN). They count no trading fees, and the risk-adjusted scores are so high that the authors themselves call them "exceptionally high" ([arXiv](https://arxiv.org/html/2412.20138v7)).
- **A one-year independent test (May 2026):** $100K grew to $106.4K before costs and $102.3K after. Simply holding the same 5 stocks gave $104.8K, and holding beat it on 4 of the 5 stocks ([Alpha Illusion](https://www.alphaxiv.org/abs/2605.16895.md)).
- **Similar AI trading agents lose their reported edge** when tested over 20 years and wider stock lists ([FINSABER](https://arxiv.org/html/2505.07078v4)).
- **One community run reportedly made 7% in 30 days,** against 4.5% for the market, but with 22% drops along the way. It could not be verified or repeated ([Pinggy](https://pinggy.io/blog/best_ai_trading_agents/)).
- **"Hindsight cheating" is the core problem.** The AI may already "remember" what happened to a stock from its training data. One user found that 10 of 12 AI models recalled past earnings results. The maintainers say code "can't fully eliminate" this ([Issue #805](https://github.com/TauricResearch/TradingAgents/issues/805)).
- **Results vary run to run.** The default reasoning models "vary the most" and largely ignore the setting meant to make answers repeatable ([GitHub](https://github.com/TauricResearch/TradingAgents)).

## Setup: models and cost

- **Use two tiers of AI model.** A cheap, fast model does most of the work (analysts, debaters, trader). A strong model handles only the two manager roles. Each tier can come from a different provider ([GitHub](https://github.com/TauricResearch/TradingAgents#readme)).
- **Cost per stock** runs from a few cents (DeepSeek Flash, about 13 AI calls) to about $0.30–$0.50 on top-tier OpenAI models, and up to $5 with long debates ([Issue #750](https://github.com/TauricResearch/TradingAgents/issues/750); [Pinggy](https://pinggy.io/blog/best_ai_trading_agents/); [Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/)).
- **Cheap options:** DeepSeek or a model running on your own computer (Ollama) cut cost by about 80–90% ([Pinggy](https://pinggy.io/blog/best_ai_trading_agents/)). A ChatGPT Plus or Pro subscription also works, and its use counts against the plan's limits ([GitHub](https://github.com/TauricResearch/TradingAgents#readme)).
- **The debate is the expensive part,** at 3–5x the cost of the analysts. Each extra debate round roughly doubles the cost. Keep it at 1 round, which is the default and the "Shallow" setting ([Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/); [default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)).
- **Other ways to save:** run fewer analysts (`--analysts market,news`), lower the news-article limits, and cap output length with `max_tokens` ([default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)).
- **Models on your own computer:**
  - The model must support tool use ([PR #268](https://backiee.wasmer.app/https_git_codeproxy_net/TauricResearch/TradingAgents/pull/268)).
  - Very small models (7–9B) "loop without resolving" the debate ([Apidog](https://apidog.com/blog/tradingagents-multi-agent-llm-trading/)).
  - A 24 GB graphics card running a ~27–32B model is a working setup that users report ([Issue #1492](https://github.com/TauricResearch/TradingAgents/issues/1492)).
- **Time:** about 5–10 minutes per stock, though only one source gives a number ([PickMyTrade](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/)).
- **Ignore old blog advice.** Posts that say Finnhub is required, recommend GPT-4o, or mention a `--rounds` flag describe the 2024–early 2025 code ([PickMyTrade](https://blog.pickmytrade.io/build-ai-hedge-fund-tradingagents/); [cli/main.py](https://github.com/TauricResearch/TradingAgents/blob/main/cli/main.py)).

## Running it reliably

- **Data limits break runs.**
  - Yahoo blocks heavy use and often blocks cloud servers ([Issue #1425](https://github.com/TauricResearch/TradingAgents/issues/1425)).
  - Alpha Vantage's free tier allows 25 calls a day ([Issue #305](https://github.com/TauricResearch/TradingAgents/issues/305)).
  - Fix: list backup data sources yourself, for example `"yfinance,alpha_vantage"`. The tool does not fall back on its own ([default_config.py](https://github.com/TauricResearch/TradingAgents/blob/main/tradingagents/default_config.py)).
- **Lock library versions.** yfinance 1.7.0 silently returned no company news. Version 1.8.0 fixed it ([Issue #1663](https://github.com/TauricResearch/TradingAgents/issues/1663)).
- **Don't trust "Analysis Complete!"** One run said it finished even though 72 of its 80 data lookups had failed ([Issue #1672](https://github.com/TauricResearch/TradingAgents/issues/1672)). Check reports for "DATA_UNAVAILABLE."
- **Set retries and limits:** `TRADINGAGENTS_LLM_MAX_RETRIES` for AI rate limits, and `max_tokens` to stop DeepSeek hangs ([Issue #1204](https://github.com/TauricResearch/TradingAgents/issues/1204)).
- **Nightly runs:**
  - Set everything through `TRADINGAGENTS_*` settings in a `.env` file, and pass `--ticker`/`--date` so it never stops to ask questions.
  - Turn on `--checkpoint` so a crashed run can resume.
  - Call `settle_all_pending()` daily to score past calls ([GitHub](https://github.com/TauricResearch/TradingAgents#readme)).
- **Many stocks:** there is no watchlist mode, so run one stock at a time. One user with 26 stocks screens them first to decide which get the full, costly analysis ([Issue #1665](https://github.com/TauricResearch/TradingAgents/issues/1665)).
- **SEC data:** set `SEC_EDGAR_USER_AGENT` to your own name and email ([GitHub](https://github.com/TauricResearch/TradingAgents#readme)).

## Safeguards before any real money

- **Test only on dates after the AI model's training ended,** or hide the stock name and dates from the model ([Issue #805](https://github.com/TauricResearch/TradingAgents/issues/805)).
- **Count every cost:** fees, the gap between buy and sell prices, price slippage and AI costs ([Alpha Illusion](https://www.alphaxiv.org/abs/2605.16895.md)).
- **Compare with simple choices:** buying and holding, an index fund, and random picks ([FINSABER](https://arxiv.org/html/2505.07078v4)).
- **Test long periods** that include up, down and flat markets, and stocks that later failed, not just today's winners ([FINSABER](https://arxiv.org/html/2505.07078v4)).
- **Run each test several times** and look at the spread, because answers change from run to run. For steadier results, use temperature 0 with non-reasoning models ([GitHub](https://github.com/TauricResearch/TradingAgents#readme)).
- **Paper trade first.** Fork guides say to start in practice mode ([dev.to](https://dev.to/aarontrng/complete-guide-using-alpacatradingagent-web-ui-for-automated-trading-3k78)).
- **Treat the output as advice to a human, not an order.** Let separate, rule-based tools handle position size and risk ([Alpha Illusion](https://www.alphaxiv.org/abs/2605.16895.md)).
- **Get a legal or compliance check** before using it live, and expect Reddit and StockTwits mood data to be stale or manipulated ([dev.co](https://dev.co/ai/frameworks/tradingagents)).

## What's still unknown

- No checked record of real-money results exists.
- There is no usage data showing how many people use it for research, testing or bots.
- Very little discussion turned up on Reddit or Hacker News.
- No test checks whether the multi-agent debate beats a single AI once costs are counted.

## Conclusion

- **Its real value is the explanation, not the signal.** It costs cents and takes minutes to get a structured, multi-angle write-up on a stock. Nobody has shown its BUY/SELL calls beat holding the stock.
- **The biggest danger is a test that looks good for the wrong reason.** The code has fixed most of its data mix-ups. But the AI model's own memory of past events is a trap only the user can avoid, by testing on recent dates or in live practice trading.
