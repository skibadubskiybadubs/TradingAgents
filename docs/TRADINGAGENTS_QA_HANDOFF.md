# TradingAgents — Q&A Handoff

Personal notes + context for future agent sessions. Repo: `skibadubskiybadubs/TradingAgents` (fork of TauricResearch/TradingAgents, v0.7.x).
Checked against the code on 2026-11-17. Research: `reports/TradingAgents use cases and practices.md` + `research_notes/TradingAgents use cases and practices/`.

**Legend:** ✅ = checked in code (file:line) · 📚 = research source · 💡 = suggestion, NOT built or tested yet

---

## 0. TL;DR

```
TradingAgents = AI "research desk" for ONE ticker → 5-tier rating (Buy/Overweight/Hold/Underweight/Sell)
  ✗ no broker, no real orders (pretend exchange only)
  ✗ no proven profit after costs (independent 1-year test lost to buy-and-hold)
  ✓ memory: scores past calls after N trading days → lessons → read by Portfolio Manager
  ✓ backtest over ticker × date grid
  ✗ does NOT know your trading style; holding_period_days only changes grading
  ✗ no cycle detection
```

**Corrections from earlier in the session:**
| Earlier claim | Correct (✅ code) |
|---|---|
| Cron: `tradingagents --ticker $T --save --no-show` is enough | Unattended runs also need `--date`, `--analysts` and 4 env vars (§10) |
| Only a run of the **same ticker** triggers scoring | **Every** run scores **all** tickers' due calls (`trading_graph.py:309-336`, `settle_all_pending(wait=False)`) |
| Repeat runs on the same day fill memory | Only the **first** run per ticker+date is stored (`memory/log.py:47-55`) |
| Ratings are BUY/HOLD/SELL | **5 ratings:** Buy, Overweight, Hold, Underweight, Sell (`agents/rating.py:22`) |

---

## 1. How one run works ✅

```
data (prices, news, filings, social)
  → 4 analysts (market · social · news · fundamentals)    ┐
  → Memory step: score due calls + read lessons            ┘ run side by side (trading_graph.py:309)
  → Bull vs Bear debate (max_debate_rounds=1)
  → Research Manager (deep model)
  → Trader → Risk team (risky · safe · neutral, max_risk_discuss_rounds=1)
  → Portfolio Manager (deep model, reads the memory lessons)
  → rating + text decision (+ optional Time Horizon, price target)
  → saved as PENDING in the memory log
```

- Returns: `final_state, rating = ta.propagate("NVDA", "2026-09-01")` ✅ (`trading_graph.py:405`)
- 📚 About 13–22 AI calls per run, about 5–10 min (blog estimate).
- 📚 Cost per run: ~$0.02–0.04 (DeepSeek), ~$0.30–0.50 (top OpenAI), up to ~$5 (long debates). Local model ≈ $0 but needs 27B+ with tool calling.

---

## 2. Ways to use it ✅

| Use | Command |
|---|---|
| Interactive | `tradingagents` or `python -m cli.main` |
| One ticker, no questions | `tradingagents --ticker NVDA --date 2026-11-14 --analysts market,news,fundamentals --save --no-show` |
| With your holdings | `tradingagents --portfolio my_book.json` |
| Backtest (many tickers) | `tradingagents backtest NVDA,AAPL --start 2026-06-01 --end 2026-08-01 --every 7` |
| Resume a crashed run | `--checkpoint` / `--clear-checkpoints` |
| ChatGPT plan instead of an API key | `tradingagents login chatgpt` |
| Docker | `docker compose run --rm tradingagents` |
| Local models | `docker compose --profile ollama run --rm tradingagents-ollama` |
| Python | `TradingAgentsGraph(config=cfg).propagate(ticker, date, portfolio=...)` |

- `--ticker` = **one** ticker (`cli/main.py:56`). Backtest takes a comma list (`cli/main.py:109`).
- Config from env: `TRADINGAGENTS_*` (`default_config.py:8-30`), e.g. `TRADINGAGENTS_MEMORY_LOG_PATH`, `TRADINGAGENTS_BENCHMARK_TICKER`, `TRADINGAGENTS_TEMPERATURE`.
- ⚠ No env var for `holding_period_days` or `global_news_lookback_days`. Set them in a Python config.

---

## 3. Real-world use cases 📚

| Use | Fit | Notes |
|---|---|---|
| Single-stock research report | ✅ good | Main use; a second opinion, not an order |
| Portfolio review | ✅ good | `--portfolio` |
| Nightly watchlist | ✅ good | Loop + cron (§10) |
| Backtest | ⚠ care | Only dates after the model's cutoff (§5) |
| Paper-trading bot signals | ⚠ care | Through forks (§11) |
| Crypto | ⚠ care | `BTC-USD`; fundamentals are thin |
| Chinese markets | ✅ | Fork TradingAgents-CN (~33k stars, paid Pro tier) |
| Learn multi-agent AI | ✅ | `python main.py` |
| Academic baseline | ✅ | Run each case several times |
| Live real-money auto trading | ❌ risky | Forks only; no checked profit record |

---

## 4. Does it make money? 📚

"The Alpha Illusion" (arXiv 2605.16895): $100K, Jan 2025 → Jan 2026, TSLA/NVDA/KO/XOM/MSTR.

```
              $98K   $100K   $102K   $104K   $106K
Agents gross   ───────────────────────────────■ 106.4
Buy & hold     ─────────────────────────■ 104.8
Agents net     ───────────────■ 102.3          ← after fees and slippage
```

- Before costs: agents +$1.6K better. **After costs: agents −$2.5K worse.** Costs ≈ $4.1K, because the agents trade often.
- Holding won on 4 of 5 stocks; MSTR was the only win for the agents. Sharpe 0.43 → 0.22 after costs.
- The paper's claim (AAPL +26.6%, Sharpe 8.21) covers 3 months and 3 mega-caps, with no fees.
- FINSABER (arXiv 2505.07078): over longer periods, the edge of similar LLM agents shrinks or goes away.
- No checked real-money track record found anywhere.
- Limit: one study, 5 stocks, 1 year. It does not prove the agents always lose.

---

## 5. Training cutoff + hindsight trap ✅📚

```
model knowledge ────────────────────┤ cutoff (e.g. Jun 2025)
backtest Mar 2025 ──► model may REMEMBER what happened → fake "skill"
backtest Sep 2025 ──────────────────────────► model cannot know → real test
```

- The code hides future **data**: point-in-time fixes v0.4–v0.7.1; entry at the first close after the decision date.
- The code cannot remove what the **model** remembers from training (GitHub issue #805, open).
- Rule: trust backtests only after the model's cutoff. The provider's model page lists the cutoff.

---

## 6. Why runs differ (non-determinism) ✅📚

```
same ticker + same date
  ├─ AI answers are sampled randomly (reasoning models ignore temperature)
  ├─ news/social show "now", even for a past date
  └─ debate is chained → small changes grow into a different rating
```

- README "Reproducibility": expected, "not a defect".
- Fixed per run: price/indicator window for the date; company identity.
- Less variation: `temperature: 0` **plus** a non-reasoning model in `deep_think_llm` / `quick_think_llm`.
- One run = one sample, not "the answer".

---

## 7. Memory + scoring (grading) ✅

File: `~/.tradingagents/memory/trading_memory.md` (`default_config.py:86`). Code: `memory/settlement.py`, `memory/log.py`, `memory/reflection.py`.

```
DAY 0  run NVDA 2026-11-16 → "Buy"
       └─ saved: [2026-11-16 | NVDA | Buy | pending]          (log.py:57)
DAY 1  entry price = first close AFTER the decision date      (settlement.py:68)
+N     wait holding_period_days TRADING days (default 5)      (default_config.py:180)
LATER  ANY run (any ticker) or settle_all_pending():
       ├─ window traded?  no → stay pending, retry next time
       ├─ prices: ticker + benchmark (SPY for US; benchmark_map by suffix)
       │    raw   = (exit − entry) / entry
       │    alpha = raw − benchmark return
       ├─ quick model writes a 2–4 sentence lesson             (reflection.py)
       └─ entry → SCORED (+raw, alpha, lesson, resolved date)
NEXT RUN of NVDA → Portfolio Manager gets:                      (portfolio_manager.py:35)
       • 5 newest SCORED NVDA entries (full)    n_same=5        (log.py:94)
       • 3 newest lessons from other tickers    n_cross=3
       • pending entries are SKIPPED                            (log.py:105)
       • past-date run: only lessons resolved by that date      (trading_graph.py:130)
```

**Scoring needs:** (1) N trading days passed · (2) Yahoo prices for ticker **and** benchmark · (3) the lesson AI call succeeds · (4) a trigger: any run, or `ta.settle_all_pending()`.

**On by default:** yes, the memory log is always on.

**"Learning" = notes in the prompt, not training:**
```
memory file ──► put in Portfolio Manager prompt ──► model reads it ──► decision
               (every run)                           (weights unchanged — Claude, GPT, any)
```
- Works the same with Claude (a supported provider). Delete the file → the learning is gone.
- Only the Portfolio Manager reads memory; analysts and debaters do not.
- 📚 No study shows this memory improves results over time.

---

## 8. Multiple runs per day → majority vote ✅💡

```
Run 1 NVDA 11-16 → Buy  ──► memory ✅ stored (first one wins)
Run 2 NVDA 11-16 → Sell ──► ❌ blocked (same ticker+date)     (log.py:47-55)
Run 3 NVDA 11-16 → Hold ──► ❌ blocked
--save reports are still written as files, but they are NOT memory
```

💡 Better: vote first, then store one call.

```
3–5 runs ──► each to a THROWAWAY memory file ──► majority vote ──► store 1 call in REAL memory
```

```python
from collections import Counter
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.memory.log import TradingMemoryLog

# Runs one ticker N times, each with its own empty memory; returns the ratings
"""['Buy', 'Buy', 'Hold']"""
def sample_ratings(ticker, date, config, n=3, tmp_dir="/tmp"):
    ratings = []
    for i in range(n):
        cfg = dict(config, memory_log_path=f"{tmp_dir}/ta_sample_{i}.md")  # keep samples out of real memory
        _, rating = TradingAgentsGraph(config=cfg).propagate(ticker, date)  # returns (final_state, rating)
        ratings.append(rating)
    return ratings

# Majority rating; no clear majority → "Hold"
"""['Buy','Buy','Hold'] -> 'Buy'   ['Buy','Sell','Hold'] -> 'Hold'"""
def majority_rating(ratings):
    rating, count = Counter(ratings).most_common(1)[0]
    return rating if count > len(ratings) / 2 else "Hold"

# Stores the single consensus call in the real memory log (scored later as usual)
def save_consensus(ticker, date, ratings, config):
    rating = majority_rating(ratings)
    text = f"Consensus of {len(ratings)} runs: {ratings} -> {rating}"
    TradingMemoryLog(config).store_decision(ticker, date, text, rating=rating)
```

- ⚠ Delete the throwaway files between days, or the samples read each other's old entries. To give the samples your real lessons, copy the real log into each one first.
- ⚠ The consensus entry has no full analysis text, so the lesson will be thin. You can append the majority run's `final_state["final_trade_decision"]` to `text`.
- Cost × N. Keep the split ("3/3" vs "2/3") as a confidence signal and check later whether 3/3 scores better.

---

## 9. Trading styles, holding_period_days, multiple memories ✅💡

### 9.1 What `holding_period_days` changes ✅

```
holding_period_days ──► GRADING only (how many TRADING days later the call is scored)
                    ✗   not sent to analysts, debaters, trader or Portfolio Manager
                    ✗   does not change data or indicators
```
- Only use: `settlement.py:157`. The reflection prompt also sees it (`reflection.py:23`).
- The agents choose their own optional "Time Horizon" per decision (`agents/schemas.py:257`). Grading **ignores** it.

### 9.2 The mismatch → wrong lessons

```
Run on Nov 1: agents say "Buy, horizon 2–3 weeks"   (they chose this)
config:       holding_period_days = 60              (you chose this)

Nov 1 ──── Nov 20: price +8% ✅ (agents' trade worked) ──── ~late Jan: price −5% ❌
                                                              │
                                          grading looks HERE, not at Nov 20
                                                              ▼
              lesson: "Buy was wrong, −5% vs market"  ← wrong lesson
```

**It happens when:**
- You set 60 days but the agents still think in weeks. Nothing tells them your timeframe.
- The default 5 days grades a "6–12 month" positional call after 1 week. ✅ This case is partly softened: the lesson prompt says the window "may be shorter than the horizon" and asks to say so "if the window is too short" (`reflection.py:23-28`).
- The long-window case (60d vs 2 weeks) is **not** softened. Grading sees only the last price, not the peak in between.
- Each run picks its own horizon ("2 weeks", then "6 months"), but all are graded on one window.

### 9.3 Fit by style ✅

| Style | Fit | Why |
|---|---|---|
| Swing (days–weeks) | ✅ good | 10 EMA, RSI, MACD, Bollinger, ATR (`market_analyst.py:25-44`); fresh news |
| Positional (months) | ⚠ partial | 50/200 SMA, fundamentals, SEC filings help; macro news only 7 days by default |
| Day trading | ❌ | Daily prices; 5–10 min per run |

### 9.4 Settings per style ✅

```python
config["holding_period_days"] = 10        # swing: 5–15 | positional: 40–60 (trading days)
config["global_news_lookback_days"] = 30  # positional: wider macro view (default 7)
config["memory_log_path"] = "~/.tradingagents/memory/swing.md"  # one file per style (~ is expanded, log.py:25)
```

### 9.5 Multiple memories (one per style) ✅

```
                ┌─ swing config ───────────────────┐   ┌─ positional config ───────────────┐
                │ memory: swing.md                 │   │ memory: positional.md             │
                │ holding_period_days = 10         │   │ holding_period_days = 60          │
                └──────────────┬───────────────────┘   └──────────────┬────────────────────┘
NVDA, Nov 1  ── run (swing) ───┤                        run (positional)┤
                               ▼                                        ▼
               [Nov 1 | NVDA | Buy | pending]           [Nov 1 | NVDA | Hold | pending]
                               │ 10 trading days                        │ 60 trading days
                               ▼                                        ▼
               graded vs SPY over 10 days              graded vs SPY over 60 days
               lesson → swing.md only                  lesson → positional.md only
                               │                                        │
next swing NVDA run reads ◄────┘          next positional NVDA run reads◄┘
```

- Memory: each file is read and written on its own. Swing lessons never reach positional runs.
- Same ticker, same day: allowed **once per file**.
- Grading: each file uses its own window. Run `settle_all_pending()` once per config. Any run also scores its own file's due calls.

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph

# One config per trading style: own memory file and own grading window
"""{'swing': {...10 days...}, 'positional': {...60 days...}}"""
def style_configs(base):
    return {
        "swing":      dict(base, memory_log_path="~/.tradingagents/memory/swing.md",      holding_period_days=10),
        "positional": dict(base, memory_log_path="~/.tradingagents/memory/positional.md", holding_period_days=60),
    }

# Scores every pending call in every style's memory, each on its own window
def settle_every_style(base):
    for name, cfg in style_configs(base).items():
        TradingAgentsGraph(config=cfg).settle_all_pending()
```

### 9.6 What separate files do NOT fix → 💡 code change

The agents still don't know the style; a "positional" run can still say "2 weeks". Separate files fix **grading**, not **thinking**.

💡 Robust fix:
1. **`trading_style` setting in the prompts**, e.g. "You are a positional trader. Horizon: 2–3 months." The analysts, trader and Portfolio Manager then reason on your timeframe.
2. **Grade on the call's own horizon**: parse `time_horizon` (or use the style window). Optionally score at several horizons:
```
call ──► @5d: −1%   @20d: +6%   @60d: +2%
                    ▲ "works on a swing horizon" → better lesson
```
Files to touch: `default_config.py`, the analyst/manager prompts, `memory/settlement.py`, `memory/log.py` (store more than one result).

---

## 10. Nightly watchlist ✅💡

"Nightly watchlist" = run your fixed list of tickers automatically each night after the close, so reports are ready by morning. There is no built-in mode; you write a loop.

```
weekdays 22:00 → cron
  ├─ for T in watchlist: tradingagents --ticker $T --save --no-show
  └─ settle_all_pending()   (optional: each run already scores due calls)
morning → fresh reports + memory updated
```

```bash
# run_watchlist.sh — analyzes each ticker, then scores old calls
# Without a terminal (cron) the CLI needs ALL of these, or it lists what is missing (cli/selections.py:58)
export TRADINGAGENTS_LLM_PROVIDER=openai
export TRADINGAGENTS_QUICK_THINK_LLM=<model>  TRADINGAGENTS_DEEP_THINK_LLM=<model>
export TRADINGAGENTS_MAX_DEBATE_ROUNDS=1      TRADINGAGENTS_MAX_RISK_ROUNDS=1
export TRADINGAGENTS_OUTPUT_LANGUAGE=English
D=$(date +%F)                                   # analysis date = today, local time
for T in NVDA AAPL MSFT BTC-USD; do
  tradingagents --ticker "$T" --date "$D" --analysts market,social,news,fundamentals --save --no-show
done
python -c "from tradingagents.graph.trading_graph import TradingAgentsGraph as G; \
from tradingagents.default_config import DEFAULT_CONFIG as C; G(config=C).settle_all_pending()"
```
```
# crontab -e
0 22 * * 1-5  /path/to/run_watchlist.sh >> ~/ta_watchlist.log 2>&1
```
- ✅ Unattended runs need `--ticker --date --analysts --save/--no-save --show/--no-show` plus the env vars above (`cli/selections.py:58-70`). API keys go in `.env`.
- Without `--date`, the interactive default is today (`cli/selections.py:192`). In cron, always pass it.
- 📚 Watch Yahoo rate limits (429s; fixed 2026-09-24, #1386). Keep 5–20 tickers.

---

## 11. Paper trading, bots, forks 📚

- **Paper trading** = pretend money on real live prices.
- **Alpaca** = US broker with an API and a free paper account.
- **AlpacaTradingAgent** (huygiatrng): "inspired by" TradingAgents; Dash UI, FRED macro analyst, crypto, scheduled runs, auto-trade with fixed $ per symbol, `ALPACA_USE_PAPER=True/False`.
- **TradingCrew** (PyPI): TradingAgents + Alpaca, crypto, options, web UI.
- I did not check the code or upkeep of either fork.

```
[1] scheduler           ← fork / your code
[2] BRAIN: analysis → rating             ← TradingAgents
[3] risk rules (size, max loss, skip if runs disagree)   ← fork / your code
[4] orders → Alpaca (paper or live)      ← fork / your code
[5] tracking, dashboard                  ← fork / your code
     └─► memory scoring                  ← TradingAgents
```

Safe path: research → backtest after cutoff → compare vs buy-and-hold/index/random → paper trade for months → small live, a person approves each trade.

---

## 12. Using your own historic trades ✅💡

```
your trades (date, ticker, rating, why)
  ├─ A. TEST: run the agents on YOUR dates → compare agents vs you   (no code change)
  ├─ B. TEACH: write your trades into memory → lessons for the PM    (small script)
  └─ C. Your cycle analysis as a standing input                      (code change, §13)
```

**A. Test** ✅
```python
from tradingagents.backtest import run_backtest, summarize
result = run_backtest(["NVDA"], ["2025-03-10", "2025-06-02", "2025-09-15"], config)  # own memory log
print(summarize(result).render())
```

**B. Teach** ✅ (`store_decision` accepts any text, `log.py:32`)
```python
from tradingagents.memory.log import TradingMemoryLog
from tradingagents.graph.trading_graph import TradingAgentsGraph

# Writes one of your past trades into memory as pending, to be scored later
"""[2025-03-10 | NVDA | Buy | pending]  + your reasoning text"""
def add_my_trade(log, ticker, date, rating, why):
    text = f"MY TRADE ({rating}). Reason: {why}"
    log.store_decision(ticker, date, text, rating=rating)   # rating: Buy/Overweight/Hold/Underweight/Sell

cfg = dict(config, memory_log_path="~/.tradingagents/memory/my_swing.md", holding_period_days=10)
log = TradingMemoryLog(cfg)
add_my_trade(log, "NVDA", "2025-03-10", "Buy", "cycle low + weekly RSI divergence")
TradingAgentsGraph(config=cfg).settle_all_pending()   # score them → AI writes a lesson for each
```

| Limit | Effect | Fix |
|---|---|---|
| One window per file | Your real exit is ignored | One file per style (§9.5) |
| Only the 5 newest same-ticker entries are read | A long history adds little | Best examples last, or one summary entry |
| Only the PM reads memory | Analysts don't see your trades | Code change |
| Past-date runs use only lessons resolved by that date ✅ | No leaks from the future | — |
| Benchmark is SPY for US stocks | Fine for US | `benchmark_ticker` for others |
| Hindsight trap (§5) | Old dates look too good | Test after the cutoff |

---

## 13. Cycles ✅💡

- ✅ No cycle detection in the code (no cycle/Fourier/Hurst logic). Indicators only: SMA 50/200, EMA 10, MACD, RSI, Bollinger, ATR, VWMA.
- The AI may *describe* patterns in text; that is a guess, not a measurement.

💡 Robust: a **Cycles analyst** node in `tradingagents/agents/analysts/`. Code measures the cycle; the AI only reads the numbers, so they are the same on every run.

```
prices ──► detrend ──► measure dominant cycle length (e.g. 38 days)
                  ──► current phase (e.g. 70% → near top)
                  ──► cycle strength (is it real or noise?)
                          │
                          ▼
             "Cycles report" ──► debate ──► Trader ──► Portfolio Manager
```
Open decision: method (fixed length, math-detected dominant cycle, or cycle lows from your own notes).

---

## 14. Best-practice checklist 📚

- [ ] Test only after the model's training cutoff
- [ ] Count all costs: fees, slippage, AI tokens
- [ ] Compare with buy-and-hold, an index and random picks
- [ ] Run each case several times; use the majority
- [ ] Paper trade before real money; a person makes the final call
- [ ] Cheap model for analysts, strong model for the 2 managers
- [ ] Debate at 1 round (it costs the most)
- [ ] Pin library versions (yfinance 1.7.0 lost company news, #1663)
- [ ] Don't trust "Analysis Complete!"; check tool errors (72/80 failed in #1672, ChatGPT-subscription provider)
- [ ] Backup data source for Yahoo / Alpha Vantage limits (AV free: 25 requests/day)
- [ ] Local models: 27B+ with tool calling
- [ ] One memory file per style; grading window = your real holding time

---

## 15. Open tasks (for a future agent) 💡

1. `trading_style` prompt setting + grade on `time_horizon` / several horizons (§9.6).
2. Cycles analyst node (§13). Needs the user's method first.
3. Majority-vote wrapper with throwaway memory (§8).
4. CSV importer for the user's past trades → A (test) + B (teach) (§12).
5. Watchlist script + cron with a correct `--date` (§10).
6. Optional: Alpaca **paper** bot around the vote + fixed risk rules (§11).

## Sources

- Code (this repo): `cli/main.py`, `tradingagents/default_config.py`, `tradingagents/graph/trading_graph.py`, `tradingagents/memory/{log,settlement,reflection}.py`, `tradingagents/agents/{rating,schemas}.py`, `tradingagents/agents/managers/portfolio_manager.py`, `tradingagents/agents/analysts/market_analyst.py`, `tradingagents/backtest.py`, `README.md`
- Paper: https://arxiv.org/abs/2412.20138 · The Alpha Illusion: https://arxiv.org/abs/2605.16895 · FINSABER: https://arxiv.org/abs/2505.07078
- Repo issues: #805 (model leakage), #1386 (Yahoo 429), #1663 (yfinance news), #1672 (ChatGPT tool calls) — https://github.com/TauricResearch/TradingAgents
- Forks: AlpacaTradingAgent guide https://dev.to/aarontrng/complete-guide-using-alpacatradingagent-web-ui-for-automated-trading-3k78 · TradingCrew https://pypi.org/project/tradingcrew/
- Full research with all links: `reports/TradingAgents use cases and practices.md`
