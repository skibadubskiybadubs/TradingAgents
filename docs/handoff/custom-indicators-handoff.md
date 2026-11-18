# Handoff: Custom Indicators (pips, candles, BOS, trendlines)

> **For agents:** read all of this file before you write code. Every code claim
> below was checked against branch `ccr-2a00612b-uaymdk` (HEAD `555a2e3`,
> 2026-11-18). **Check the line numbers again before you edit.** They move.
> Status: **design only. No code is written yet.**

---

## 1. Goal

Give the Market Analyst agent new indicators:

1. **Custom formulas** (any math on OHLCV)
2. **Pips / candle-based**: body, wicks and range in pips, plus candle patterns
3. **BOS** (break of structure)
4. **Trendlines** (support and resistance lines through swing points)

---

## 2. Current flow (verified)

```
Market Analyst prompt (hardcoded list of allowed names)
  │  the LLM calls get_indicators("rsi")
  ▼
tradingagents/agents/tools.py:42            get_indicators  (@tool, splits "a,b" → 1 call each)
  ▼
tradingagents/dataflows/router.py:200       route_to_vendor("get_indicators", ...)
  │  vendor map at router.py:104; config key technical_indicators
  │  (default "yfinance", default_config.py:160)
  ├─ yfinance      → dataflows/vendors/yahoo/market.py:66   get_stock_stats_indicators_window
  │                    ├─ whitelist: best_ind_params dict (market.py:75)
  │                    ├─ window path:   _get_stock_stats_bulk (market.py:205)
  │                    └─ fallback path: get_stockstats_indicator (236) → get_stock_stats (273)
  │                    both paths: load_ohlcv → stockstats.wrap(df) → df[name]
  └─ alpha_vantage → dataflows/vendors/alpha_vantage/indicator.py:9  get_indicator
                       fixed list of indicators; custom names are NOT possible
```

**Rule:** an indicator name must match in **3 places**:

| # | Place | File |
|---|---|---|
| 1 | Prompt list (the names the LLM can choose) | `tradingagents/agents/analysts/market_analyst.py:22-46` |
| 2 | Whitelist plus description | `best_ind_params` in `yahoo/market.py:75` |
| 3 | Calculator | `stockstats` (`>=0.6.5`, pyproject.toml) inside `_get_stock_stats_bulk` and `get_stock_stats` |

**Found while checking:** `mfi` is in the whitelist (2) but not in the prompt (1), so the LLM never asks for it.

### Data source facts (verified)

- `load_ohlcv(symbol, as_of_date, fill_gaps=True)` is in `yahoo/ohlcv.py:161`.
- It downloads **5 years** of data (ohlcv.py:183) with `yf.Ticker(...).history(start, end, auto_adjust=True)` (ohlcv.py:212).
  It sets **no `interval`**, so the data is **1 candle per day** (the Yahoo default `1d`).
- It removes rows after `as_of_date`, so the agents cannot see future prices. `as_of_date` is a whole day (`YYYY-MM-DD`, normalized at ohlcv.py:179).
- It keeps one cache file per symbol, not per interval.
- `fill_gaps=True` carries prices forward over gaps, which **makes flat fake candles**.
- It converts symbols with `dataflows/symbols.py`: forex `EURUSD` → `EURUSD=X`, gold `XAUUSD` → `GC=F`.

---

## 3. Decisions made in this session

### 3.1 Custom formula: one registry

Make **one** dict that holds the formula and the description together. Both calculator paths use it, so the window table and the one-day value cannot drift apart.

```python
# NEW: tradingagents/dataflows/vendors/yahoo/custom_indicators.py
# name -> (function(df) -> pd.Series, description for the LLM)
CUSTOM_INDICATORS = {"sma50_gap_pct": (sma50_gap_pct, "SMA50 Gap %: ...")}

# Adds the custom column to df if the name is custom; returns df unchanged otherwise
def add_custom_indicator(df, name): ...
```

How to connect it:
- `market.py:66`: `best_ind_params.update({k: d for k, (_, d) in CUSTOM_INDICATORS.items()})`
- `market.py:205` and `market.py:273`: put `df = add_custom_indicator(df, indicator)` before the bare `df[indicator]` line.
- `market_analyst.py`: add the name to the prompt list.
- The registry needs the **symbol** for pip math. Pass it as an argument: `add_custom_indicator(df, name, symbol)`.

For a built-in stockstats indicator (`cci`, `kdjk`, `wr_14`, ...) you need only places 1 and 2. No registry is necessary.

### 3.2 Two kinds of indicator

**Rule:** if the answer is "a number for **each** candle", it is Type A. If it is "a **few facts** about the chart", it is Type B.

| | **Type A**: one value per candle | **Type B**: price structure |
|---|---|---|
| Output | a Series, shown as a date/value table | short text summary |
| Path | `CUSTOM_INDICATORS` → existing `get_indicators` | **new tool** `get_price_structure(symbol, curr_date)` |
| Why | it fits the existing table format | the LLM reads "last BOS up at 1.0912 on 05-06" better than 30 rows of 0/1 |

| Indicator | Type A | Type B |
|---|---|---|
| Pips per candle (`body_pips`, `upper_wick_pips`, `lower_wick_pips`, `range_pips`) | ✅ main | – |
| Candle patterns (`engulfing`, `pin_bar`, `inside_bar`: +1 / −1 / 0) | ✅ flags | optional "last pattern: bullish engulfing 05-07" |
| BOS | ✅ `bos` flag (+1 / 0 / −1) for history and backtests | ✅ current structure: last BOS date, level, direction |
| Trendline | ❌ (the same line repeats on each row = noise) | ✅ main |
| Swing highs/lows | an internal step | ✅ "last swing high/low" |

Type B text must contain:
- the last confirmed swing high and swing low (price and date)
- the last BOS (date, level, direction)
- the support and resistance lines: slope in pips/day, value today, number of touches
- the distance from price to each line in pips

---

## 4. Traps (must handle)

1. **Seeing the future (look-ahead).** A swing at bar *t* needs *k* bars after *t* to confirm it. You can know it only at bar *t+k*. Use `shift(k)` before you use any swing (BOS, trendline). A rolling window with `center=True` without the shift **leaks future data into backtests**.
2. **Fake candles.** For candle and pip math, call `load_ohlcv(..., fill_gaps=False)`.
3. **Pip size depends on the symbol:**
   - `*JPY=X` → 0.01
   - other `*=X` → 0.0001
   - all other symbols → 1.0 (price points)
   - Gold (`GC=F`) is not settled yet: many brokers use 0.1. **Open question.**
4. **Daily candles only.** See section 6.
5. **Alpha Vantage** cannot serve custom names. Register the new indicators for the `yfinance` vendor only.

---

## 5. Reference code (draft, not tested)

```python
# NEW: tradingagents/dataflows/vendors/yahoo/structure.py
import numpy as np
import pandas as pd

# Pip size per symbol: "USDJPY=X" -> 0.01, "EURUSD=X" -> 0.0001, other symbols -> 1.0
def pip_size(symbol: str) -> float:
    if symbol.endswith("=X"):
        return 0.01 if "JPY" in symbol else 0.0001
    return 1.0

# Per-candle sizes in pips: body (+ green / - red), upper wick, lower wick, range
def candle_pips(df: pd.DataFrame, pip: float) -> pd.DataFrame:
    top = df[["open", "close"]].max(axis=1)
    bottom = df[["open", "close"]].min(axis=1)
    return pd.DataFrame({
        "body_pips": (df["close"] - df["open"]) / pip,
        "upper_wick_pips": (df["high"] - top) / pip,
        "lower_wick_pips": (bottom - df["low"]) / pip,
        "range_pips": (df["high"] - df["low"]) / pip,
    })

# Last confirmed swing high/low on each day; a swing becomes known only k bars later
def confirmed_swings(df: pd.DataFrame, k: int = 3) -> pd.DataFrame:
    win = 2 * k + 1
    is_high = df["high"] == df["high"].rolling(win, center=True).max()
    is_low = df["low"] == df["low"].rolling(win, center=True).min()
    sh = df["high"].where(is_high)          # Step 1: swing price at the swing bar
    sl = df["low"].where(is_low)
    return pd.DataFrame({                   # Step 2: shift to the day it is known, carry forward
        "swing_high": sh.shift(k).ffill(),
        "swing_low": sl.shift(k).ffill(),
    })

# BOS per day: +1 close above the last swing high, -1 close below the last swing low, 0 none
def bos_signal(df: pd.DataFrame, k: int = 3) -> pd.Series:
    sw = confirmed_swings(df, k)
    up = df["close"] > sw["swing_high"].shift(1)
    down = df["close"] < sw["swing_low"].shift(1)
    return pd.Series(np.select([up, down], [1, -1], 0), index=df.index)

# Line through the last 2 raw swing points: (slope per bar, value on the last bar), or None
def trendline(points: pd.Series, n_bars: int) -> tuple[float, float] | None:
    pts = points.dropna()                   # raw swings at their own bars (RangeIndex)
    if len(pts) < 2:
        return None
    x = pts.index[-2:].to_numpy(dtype=float)
    y = pts.iloc[-2:].to_numpy()
    slope = (y[1] - y[0]) / (x[1] - x[0])
    return slope, y[1] + slope * (n_bars - 1 - x[1])
```

Notes:
- `trendline` needs the **raw** swing points (`df["low"].where(is_low)`). Use only swings older than `k` bars. Do not use the carried-forward column.
- The column names must be lowercase. `stockstats.wrap()` makes them lowercase. Check what `load_ohlcv` returns before `wrap`.
- The `bos` flag shows **every** close above the old swing high, not only the first break. Decide if you want only the first break; to get it, compare with the previous bar.

---

## 6. Intraday (5m / 1h) is a separate project

The current pipeline is daily. Intraday BOS and pip scalping need these changes:

| Change | Where |
|---|---|
| A data source with long intraday history (broker API such as OANDA, or a paid vendor). Add it as a new vendor | `dataflows/router.py` vendor map |
| An `interval` argument plus a cache key per interval | `yahoo/ohlcv.py:161-212` |
| `as_of` as a **time**, not a day (look-ahead cutoff) | `ohlcv.py:179`, the tools and the graph trade date |
| Agents that run per bar or session, not per trade date | graph / CLI |

Yahoo intraday history limits (set by Yahoo, can change):

| Interval | Max history |
|---|---|
| 1m | 7 days per request, only within the last 30 days |
| 2m – 90m | 60 days |
| 60m / 1h | 730 days |

These limits make 5 years of intraday history from Yahoo impossible.

---

## 7. Implementation plan (checklist)

- [ ] **Step 1:** add `mfi` to the prompt in `market_analyst.py` (quick fix)
- [ ] **Step 2:** create `yahoo/custom_indicators.py` (registry plus `add_custom_indicator(df, name, symbol)`)
- [ ] **Step 3:** connect the registry in `market.py` (whitelist update plus both calculator paths)
- [ ] **Step 4:** create `yahoo/structure.py` (section 5 functions)
- [ ] **Step 5:** add Type A entries: `body_pips`, `upper_wick_pips`, `lower_wick_pips`, `range_pips`, `bos`, `engulfing`
- [ ] **Step 6:** write `get_price_structure(symbol, curr_date) -> str` with `fill_gaps=False`
- [ ] **Step 7:** register `get_price_structure` in `router.py` (new method plus category, yfinance only)
- [ ] **Step 8:** write the `@tool` wrapper in `agents/tools.py` (copy `get_indicators` and keep the `as_of(curr_date, trade_date)` handling)
- [ ] **Step 9:** add the tool to the Market Analyst tool list and explain in the prompt when to use it
- [ ] **Step 10:** write the tests (section 8)
- [ ] **Step 11:** run `/understand` to refresh `.ua/knowledge-graph.json`

## 8. Tests to write

Copy the style of `tests/test_undated_tools_as_of.py` and `tests/test_yahoo_rate_limit.py`.

1. **Pip size:** `EURUSD=X` → 0.0001, `USDJPY=X` → 0.01, `AAPL` → 1.0.
2. **Candle pips:** for one fake candle O=1.1000, C=1.1050, H=1.1080, L=1.0990, check body 50, upper wick 30, lower wick 10, range 90.
3. **No look-ahead:** make a fake series with a known swing at bar 10. With k=3, `confirmed_swings` must be NaN or the old value at bars 10–12 and the new value from bar 13.
4. **BOS:** when the close goes above the last confirmed swing high on bar *n*, `bos[n] == 1`, and `bos` is 0 before bar *n*.
5. **Trendline:** for 2 swing lows at (bar 0, 1.00) and (bar 10, 1.10), slope = 0.01 and the value at bar 20 = 1.20.
6. **Integration:** `get_stock_stats_indicators_window("AAPL", "<custom>", date, 30)` returns numbers, not `N/A`. Mock `load_ohlcv`; do not use the network.

## 9. Open questions (user to decide)

- Pip size for gold, indices and crypto?
- Swing strength `k` (default 3): fixed, or a parameter for each call?
- BOS: every close above the level, or the first break only? Does a wick break count, or only a close break?
- Trendline: the line through the last 2 swings (simple), or a fit through 3 or more touches?
- Intraday: is it needed? If it is, which data vendor?

## 10. Sources

- Code: the files and lines above (verified at HEAD `555a2e3`)
- Yahoo intraday limits:
  - [yfinance wiki: Ticker](https://github.com/ranaroussi/yfinance/wiki/Ticker)
  - [yfinance issue #1916 (2m/5m within 60 days)](https://github.com/ranaroussi/yfinance/issues/1916)
  - [yfinance issue #2451 (1h within 730 days)](https://github.com/ranaroussi/yfinance/issues/2451)
- stockstats: <https://github.com/jealous/stockstats>
