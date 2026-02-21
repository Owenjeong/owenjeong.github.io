---
layout: single
title:  "Deep dive into analysis"
categories: Analysis
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---



# 📊 Stock Analysis Workflow

> **System:** ML Analysis Pipeline · AI Valuation Engine · Schwab GEX Module · Agentic AI

---

## Table of Contents

1. [System Architecture Overview](#1-system-architecture-overview)
2. [Step 1 — Ticker Selection](#2-step-1--ticker-selection)
3. [Step 2 — Fundamental Analysis](#3-step-2--fundamental-analysis)
4. [Step 3 — DCF Valuation (AI)](#4-step-3--dcf-valuation-ai)
5. [Step 4 — Machine Learning Forecast](#5-step-4--machine-learning-forecast)
6. [Step 5 — Backtesting (MA Strategy)](#6-step-5--backtesting-ma-strategy)
7. [Step 6 — Sentiment Analysis](#7-step-6--sentiment-analysis)
8. [Step 7 — Quantitative Risk Analysis](#8-step-7--quantitative-risk-analysis)
9. [Step 8 — Macroeconomic Analysis](#9-step-8--macroeconomic-analysis)
10. [Step 9 — GEX (Gamma Exposure)](#10-step-9--gex-gamma-exposure)
11. [Step 10 — Weighted Score & Signal](#11-step-10--weighted-score--signal)
12. [Step 11 — AI Final Summary](#12-step-11--ai-final-summary)
13. [Step 12 — Agentic AI (Alternative Pipeline)](#13-step-12--agentic-ai-alternative-pipeline)
14. [Output — Email Report](#14-output--email-report)
15. [Configuration Reference](#15-configuration-reference)

---

## 1. System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                     fetch_ML_v2.py / ML_report_v2.py                │
│                                                                       │
│  [Ticker Selection]                                                   │
│       │                                                               │
│       ▼                                                               │
│  ┌─────────────┐  ┌───────────┐  ┌──────────┐  ┌────────────────┐   │
│  │ Fundamental │  │    DCF    │  │    ML    │  │  Backtesting   │   │
│  │  Analysis   │  │  (AI LLM) │  │ Forecast │  │  (MA Strategy) │   │
│  └──────┬──────┘  └─────┬─────┘  └────┬─────┘  └───────┬────────┘   │
│         │               │             │                 │             │
│  ┌──────▼──────┐  ┌──────▼──────┐  ┌──▼──────────┐  ┌──▼──────────┐ │
│  │  Sentiment  │  │    Quant    │  │    Macro    │  │    GEX      │ │
│  │  (FinBERT)  │  │ (QuantStats)│  │   (FRED)   │  │  (Schwab)   │ │
│  └──────┬──────┘  └─────┬───────┘  └────┬────────┘  └─────┬───────┘ │
│         │               │               │                  │          │
│         └───────────────┴───────────────┴──────────────────┘          │
│                                    │                                   │
│                          [Weighted Score]                             │
│                                    │                                   │
│                          [AI Summary Report]                          │
│                                    │                                   │
│                    [Email + Database Insert]                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Step 1 — Ticker Selection

**Source:** Stock Screener Utility — Sector & momentum-ranked stock picker

### How It Works

The screener finds **the single highest-momentum, highest-liquidity stock** each day using Finviz.

```
1. Sector finder
   └─ Overview → all sectors sorted by daily % change
   └─ Pick the TOP sector (best-performing sector today)

2. Ticker finder
   └─ Custom screener with filters:
       • Index:        Any
       • Country:      USA
       • Performance:  Today Up
       • Gap:          Up  (opening gap up)
   └─ Order by: % Change (descending)
   └─ Take top 25 results
   └─ Calculate Trade Value = Price × Volume
   └─ Sort by Trade Value (descending)
   └─ Return #1 ticker  ← highest momentum + highest liquidity
```

### Screener Columns Pulled

| Column | Description |
|--------|-------------|
| Ticker | Symbol |
| Price | Current price |
| Change | Daily % change |
| Volume | Share volume |
| P/E | Price/Earnings |
| EPS this Y | EPS growth this year |
| EPS next Y | EPS growth next year |
| Sales Q/Q | Revenue growth QoQ |
| P/FCF | Price/Free Cash Flow |
| P/B | Price/Book |
| ROI | Return on Investment |
| ROE | Return on Equity |



## 3. Step 2 — Fundamental Analysis

**Source:** Fundamental Data Fetcher — Multi-source stock fundamentals aggregator (Finviz + yfinance)

### Data Sources

| Source | Data |
|--------|------|
| **Finviz** (`finvizfinance`) | P/E, P/B, P/S, ROA, ROE, gross margin, operating margin, EPS growth, short float, insider own% |
| **yfinance** | Company info, valuation, earnings, balance sheet, analyst targets, insider transactions, earnings dates |

### Key Metrics Collected

**Valuation**
- Market Cap, Enterprise Value
- Trailing P/E, Forward P/E, PEG Ratio
- Price/Book, Price/Sales, EV/EBITDA, EV/Revenue

**Profitability**
- EPS (trailing & forward)
- Revenue & Revenue Growth
- Gross Margins, Operating Margins, Profit Margins
- ROE (Return on Equity), ROA (Return on Assets)

**Balance Sheet Health**
- Total Cash, Total Debt, Debt/Equity Ratio
- Current Ratio
- Free Cash Flow, Operating Cash Flow

**Price & Targets**
- Current Price
- Analyst Target (Low / Mean / Median / High)
- 52-Week High/Low
- 50-Day / 200-Day Moving Averages
- Dividend Yield, Payout Ratio

**Qualitative**
- Recent analyst recommendations (last 5)
- Insider transactions (last 5)
- Upcoming earnings dates

---

## 4. Step 3 — DCF Valuation (AI)

**Source:** AI-Powered DCF Valuation Engine — 7-step discounted cash flow analysis

### Inputs (from yfinance)

```python
ticker_yf.cashflow        # Cash flow statement
ticker_yf.balance_sheet   # Balance sheet
ticker_yf.income_stmt     # Income statement (P&L)
company_info              # Key stats dict
```

### AI DCF Methodology

The AI model is instructed to follow this exact 7-step process:

```
Step 1 ─ Forecast 5 years of Free Cash Flow (FCF)
          • Based on historical FCF trend
          • Max 20% annual growth unless justified
          • No negative forecasted FCF

Step 2 ─ Terminal Value (Gordon Growth Model)
          TV = Final Year FCF × (1 + g) / (WACC - g)

Step 3 ─ WACC Calculation
          • Cost of Equity = CAPM: rf + β × (rm - rf)
          • Cost of Debt = avg interest rate × (1 - tax rate)
          • Capital structure = Debt / (Debt + Equity)
          • WACC = wE × rE + wD × rD

Step 4 ─ Discount all FCFs + Terminal Value to PV
          PV = FCF_t / (1 + WACC)^t

Step 5 ─ Subtract Net Debt
          Equity Value = Σ PV(FCF) + PV(TV) - Net Debt
          Net Debt = Total Debt - Cash

Step 6 ─ Intrinsic Share Price
          Price = Equity Value / Shares Outstanding

Step 7 ─ Output: Intrinsic value (USD total) + per-share price
```

### Output

> **Intrinsic Share Price: $XXX.XX**
> Comparison to current market price → overvalued / fair / undervalued

---

## 5. Step 4 — Machine Learning Forecast

**Source:** ML Forecasting Engine — Bidirectional LSTM + XGBoost ensemble price predictor

### Data Preparation

- **Price history:** 3 years from yfinance
- **Lookback window:** 60 trading days
- **Forecast horizon:** 20 trading days
- **Train/Test split:** 80% / 20%

### Feature Engineering

```python
# Price features
returns          = Close.pct_change()
log_returns      = log(Close / Close.shift(1))

# Moving averages (ratios — normalized)
sma_5/10/20/50   = Close.rolling(window).mean()
sma_ratio        = Close / sma

# Volatility
volatility_20    = returns.rolling(20).std()

# RSI (14-period)
rsi = 100 - (100 / (1 + avg_gain / avg_loss))

# MACD
macd             = EMA(12) - EMA(26)
macd_signal      = EMA(macd, 9)

# Bollinger Bands
bb_mid           = SMA(20)
bb_upper         = SMA(20) + 2 × std(20)
bb_lower         = SMA(20) - 2 × std(20)
bb_position      = (Close - bb_lower) / (bb_upper - bb_lower)
```

### Model Architecture

#### LSTM (Primary)

```
Input: (60 days × 1 feature — scaled Close price)
  ↓
Bidirectional LSTM (128 units) → BatchNorm → Dropout(0.3)
  ↓
LSTM (64 units) → BatchNorm → Dropout(0.3)
  ↓
LSTM (32 units) → BatchNorm → Dropout(0.2)
  ↓
Dense(16, ReLU) → Dense(1)

Optimizer:  Adam (lr=0.001)
Loss:       Huber (robust to outliers)
Callbacks:  EarlyStopping (patience=5), restore_best_weights=True
Epochs:     10 max
```

#### XGBoost (Ensemble Pair)

```python
XGBRegressor(
    n_estimators=100,
    max_depth=6,
    learning_rate=0.1,
    subsample=0.8,
    colsample_bytree=0.8
)
```

#### Ensemble

```
Final Prediction = (LSTM prediction + XGBoost prediction) / 2
```

### Forecast Output

```
RMSE              : Root Mean Squared Error
MAE               : Mean Absolute Error
MAPE              : Mean Absolute Percentage Error
Model Accuracy    : max(0, 100 - MAPE) %
Forecast Trend    : "Up" or "Down"
Expected Return   : (last_forecast - first_forecast) / first_forecast
forecast_df       : DataFrame of 20-day price forecast
```

---

## 6. Step 5 — Backtesting (MA Strategy)

**Source:** MA Strategy Backtester — Vectorized moving average signal simulator

### Strategy Logic

```
For each MA window in [5, 7, 9, 12, 15, 20, 50]:
  ┌──────────────────────────────────────┐
  │ Entry Signal                         │
  │   |Price - MA| / MA < 1%            │
  │   (price touching the moving average)│
  └──────────────────────────────────────┘
            │
            ▼
  ┌──────────────────────────────────────┐
  │ Exit Signal                          │
  │   Take Profit:  PnL ≥ +5%           │
  │   Stop Loss:    PnL ≤ -5%           │
  └──────────────────────────────────────┘
```

- **Data:** yfinance from `2022-01-01` to yesterday
- **Starting capital:** $10,000
- **Benchmark:** Buy-and-Hold return over same period

### Output Per MA Window

| Metric | Description |
|--------|-------------|
| `total_trades` | Number of trades executed |
| `win_rate` | % of profitable trades |
| `return_pct` | Total strategy return % |
| `final_capital` | End capital from $10,000 |
| `avg_holding_days` | Average days held per trade |

### Selection Criteria

**Best MA Window** = highest `return_pct` across all windows
Feeds into: `best_sma` stored in database, `best_holding` days

---

## 7. Step 6 — Sentiment Analysis

**Source:** Sentiment Analysis Pipeline — FinBERT NLP engine for news & Reddit crowd signals

### Sentiment Models

| Model | Role |
|-------|------|
| **FinBERT** (`ProsusAI/finbert`) | Primary — fine-tuned on financial text |
| **VADER** + custom finance lexicon | Fallback if FinBERT unavailable |

#### Custom Finance Lexicon (VADER additions)

```python
'bullish': +3.7   'bearish': -3.7
'buy': +4.0       'sell': -4.0
'upgrade': +3.0   'downgrade': -3.0
'beat': +2.5      'miss': -2.5
'moon': +4.0      'crash': -3.5
'rally': +2.5     'squeeze': +2.0
```

### News Analysis

**Sources:**
- **Finviz** — recent news headlines for the ticker
- **Yahoo Finance** — `yf.Ticker(ticker).news`

**Process:**
```
Fetch headlines → Deduplicate → FinBERT scoring →
Average compound score → Return {score, pos%, neg%, neu%, headline_count}
```

### Reddit Analysis

**Subreddits:** `wallstreetbets`, `stocks`, `investing`, `stockmarket`

**Filters:**
- Upvote ratio ≥ 0.70
- Upvotes ≥ 20
- Checks post title + body + top comments

**Process:**
```
Parallel fetch (4 workers) across subreddits →
Match ticker symbol or company name →
FinBERT scoring on matching posts/comments →
Average compound score → Return {score, mention_count}
```

---

## 8. Step 7 — Quantitative Risk Analysis

**Source:** Quantitative Risk Analyzer — QuantStats performance metrics + Risk/Return vs S&P 500 charting

### QuantStats Metrics (from `2022-01-01`)

| Metric | Meaning |
|--------|---------|
| **Sharpe Ratio** | Risk-adjusted return (rf = 4% annual) |
| **Sortino Ratio** | Downside-adjusted return |
| **Calmar Ratio** | CAGR / Max Drawdown |
| **Max Drawdown** | Worst peak-to-trough loss |
| **Volatility** | Annualized standard deviation of returns |
| **Win Rate** | % of positive daily returns |
| **Profit Factor** | Gross profit / Gross loss |
| **CAGR** | Compound Annual Growth Rate |
| **Value at Risk (VaR)** | 95% worst-case daily loss |
| **Expected Return** | Mean daily return |

### Risk vs Return Chart

`MarketVsticker.generate_vsticker()` produces a **scatter plot**:
- X-axis: Risk (daily return std dev)
- Y-axis: Average return × 10
- Plots ticker vs S&P 500
- Red dashed line = S&P 500 slope (Sharpe reference line)

> Tickers **above** the S&P line offer better risk-adjusted returns

---

## 9. Step 8 — Macroeconomic Analysis

**Source:** Macroeconomic Analysis Engine — FRED API data tracker with VIX and BofA Bear/Bull indicator scoring

### Data Sources

| Source | Series | Description |
|--------|--------|-------------|
| **FRED API** | `GDP` | Gross Domestic Product |
| | `FEDFUNDS` | Federal Funds Rate |
| | `CPIAUCSL` | CPI — inflation |
| | `PPIACO` | Producer Price Index |
| | `DGS10` | 10-Year Treasury Yield |
| | `DGS2` | 2-Year Treasury Yield |
| | `UNRATE` | Unemployment Rate |
| | `USSLIND` | Leading Economic Index |
| | `HOUST` | Housing Starts |
| | `PERMIT` | Building Permits |
| | `UMCSENT` | Consumer Confidence |
| **Yahoo Finance** | `^VIX` | Volatility Index |
| | `SPY`, `QQQ`, `IWM` | Market breadth |

### VIX Classification

| VIX Level | Classification |
|-----------|---------------|
| < 15 | Low (complacent) |
| 15–20 | Normal |
| 20–25 | Elevated |
| 25–30 | High |
| > 30 | Extreme (fear) |

### Yield Curve Analysis

```
Spread = 10Y Treasury Yield - 2Y Treasury Yield

Spread < 0  →  ⚠️ INVERTED (Recession warning)
Spread > 0  →  ✅ Normal
```

### Market Breadth (SPY / QQQ / IWM)

For each index: 1D, 1W, 1M, 3M, YTD returns + trend classification:
- strong uptrend / uptrend / sideways / downtrend / strong downtrend

### BofA Bear Indicators (10 signals — Market Peak Warnings)

| # | Signal | Threshold |
|---|--------|-----------|
| 1 | Consumer Confidence | > 110 (prior 60mo) |
| 2 | Net % expecting stocks higher | > 20% (prior 6mo) |
| 3 | Sell-side indicator | "Sell" signal |
| 4 | S&P 500 LT growth 5yr Z-score | > 1 |
| 5 | M&A deals 10yr Z-score | > 1 |
| 6 | (Trailing PE + YoY CPI) Z-score | > 1 |
| 7 | Low PE underperforms High PE | by 2.5ppt (6mo) |
| 8 | **Inverted yield curve** | Spread < 0 |
| 9 | Credit stress indicator | < 0.25 |
| 10 | Tightening credit (SLOOS) | Triggered |

### BofA Bull Indicators (10 signals — Market Bottom Signals)

| # | Signal | Threshold |
|---|--------|-----------|
| 1 | Fed cutting rates | Prior 12mo |
| 2 | Unemployment rising | vs 12mo low |
| 3 | More bears than bulls | AAII survey |
| 4 | ERP increase | > 75bps vs 12mo low |
| 5 | 2Y yield decline | > 50bps vs 6mo high |
| 6 | Sell-side indicator | "Buy" signal |
| 7 | Yield curve steepening | vs 6mo low |
| 8 | 5% bear market rally | Prior 3mo |
| 9 | Rule of 20 triggered | — |
| 10 | PMI improves | vs 12mo YoY low |

### Macro Sentiment Classification

```
Bear signals - Bull signals > 2  →  BEARISH
Bull signals - Bear signals > 2  →  BULLISH
Otherwise                        →  NEUTRAL
```

### Macro Score Adjustment

```python
if macro_sentiment == 'bearish' or bear_count >= 5:
    macro_adjustment = -0.05   # Reduce buy conviction
elif macro_sentiment == 'bullish' or bull_count >= 5:
    macro_adjustment = +0.02   # Slightly increase conviction

if VIX > 30:
    macro_adjustment -= 0.03   # Additional caution in fear spike
```

---

## 10. Step 9 — GEX (Gamma Exposure)

**Source:** Schwab GEX Calculator — Dealer gamma exposure derived from live Schwab option chain feed  
**API:** Charles Schwab Live Market Data — Real-time option chain endpoint (`/marketdata/v1/chains`)

### What Is GEX?

GEX measures how much gamma market makers (dealers) hold, which predicts near-term market behavior:

```
GEX Formula:
  Call GEX =  Gamma × Open Interest × 100 × Spot Price  (positive)
  Put  GEX = -Gamma × Open Interest × 100 × Spot Price  (negative)

Total GEX = Σ(Call GEX) + Σ(Put GEX) across all strikes
```

### Market Condition Interpretation

| Condition | Meaning | Market Behavior |
|-----------|---------|----------------|
| **Positive GEX** (Total > 0) | Dealers are **long gamma** | Mean-reverting, lower volatility, price pinned to large strikes |
| **Negative GEX** (Total < 0) | Dealers are **short gamma** | Trending/accelerating, higher volatility, moves amplified |

### Key GEX Levels

```
GEX Flip Level  — Strike where cumulative GEX crosses zero
                  Acts as a market pivot / support-resistance

Support Level   — Highest |GEX| strike below current price
                  Large positive GEX below = dealers hedge by buying → support

Resistance Level — Highest |GEX| strike above current price
                   Large positive GEX above = dealers hedge by selling → resistance
```

### 0DTE GEX

Special calculation for **same-day expiration** options only (highest gamma impact near expiry). Useful for intraday trading levels.

### GEX Report Output

```
═══════════════════════════════════
         GEX ANALYSIS: SPY
═══════════════════════════════════
📊 Current Price: $565.43
💰 Total GEX: +$2.14B

📈 Market Condition: POSITIVE_GAMMA
   Dealers long gamma → Market likely to mean-revert

🎯 Key Levels:
   • GEX Flip:    $558.00
   • Support:     $560.00
   • Resistance:  $570.00

📍 Top Positive GEX Strikes (Resistance):
   • $570.00: $420.5M
   • $575.00: $380.2M
   ...
```

### Covered Call Screener (bonus)

**Source:** Covered Call Screener — High-IV option income opportunity finder using Schwab option chains

Screens for high-IV covered call opportunities:

| Metric | Formula |
|--------|---------|
| Premium Yield | bid / spot price |
| Annualized Return | premium_yield × 365/DTE |
| Downside Protection | bid / spot price |
| Theta/Gamma Ratio | θ / Γ (higher = better risk/reward) |

**Filters:** DTE 14–60, OTM 1%–15%, min OI 10, configurable IV floor

---

## 11. Step 10 — Weighted Score & Signal

**Source:** ML Analysis Orchestrator — Weighted signal aggregator that combines all module outputs with macro adjustment

### Score Weights

```
                    ┌─────────────────────────────────────────────────┐
                    │           WEIGHTED SCORE CALCULATION            │
                    │                                                  │
   News Sentiment   │  score × 0.30  (FinBERT compound, range -1 to 1)│
   Reddit Sentiment │  score × 0.20  (FinBERT compound, range -1 to 1)│
   ML Forecast      │  return × 0.30 (expected 20-day return decimal) │
   Buy-and-Hold     │  return × 0.20 (historical BnH return decimal)  │
                    │  ─────────────────────────────────────────────── │
                    │  raw_score  = sum of above                       │
                    │  + macro_adjustment  (-0.08 to +0.02)            │
                    │  ─────────────────────────────────────────────── │
                    │  weighted_score = raw_score + macro_adjustment   │
                    └─────────────────────────────────────────────────┘
```

### Signal Decision

```python
if weighted_score > 0.5:
    signal = "Buy"
else:
    signal = "Hold"
```

### Database Record (`tk_selected` table)

```sql
INSERT INTO tk_selected (date, ticker, score, news, reddit, lstm, holding_days, sma, result)
VALUES (today, ticker, weighted_score, news_score, reddit_score, forecast_trend, best_holding, best_ma, signal)
```

---

## 12. Step 11 — AI Final Summary

**Source:** AI Investment Report Generator — Multi-factor synthesis engine that produces a buy-side equity memo

### Inputs Fed to AI Model

```
═══ FUNDAMENTALS ═══        (Finviz + yfinance table)
═══ TECHNICAL CHART ═══     (Matplotlib chart: price + MA + forecast + sentiment + backtest)
═══ ML ANALYSIS REPORT ═══  (Full text report from generate_report())
═══ QUANTITATIVE METRICS ══ (QuantStats summary)
═══ MACROECONOMIC ANALYSIS ═(FRED + VIX + BofA signals)
═══ GEX ANALYSIS ═══        (Schwab GEX report)
═══ DCF VALUATION ═══       (AI DCF output)
```

### Output Structure

```
1. Executive Summary (1-2 paragraphs)
   └─ Valuation, market position, technicals, macro, sentiment

2. Macroeconomic Context
   └─ VIX level, yield curve status, market breadth (SPY/QQQ/IWM)
   └─ BofA Bear/Bull signals
   └─ Risk-on / Risk-off assessment

3. GEX Analysis
   └─ Positive vs Negative gamma interpretation
   └─ Key GEX support/resistance/flip levels
   └─ Position sizing guidance from GEX

4. Fundamental Strengths & Risks (bullet points)

5. Sentiment Assessment
   └─ News vs Reddit divergence analysis

6. Risk/Return Positioning
   └─ Volatility, drawdowns, win rate

7. Model Alignment
   └─ MA backtest vs LSTM forecast alignment

8. DCF Comparison
   └─ Intrinsic value vs current price

9. ─────────────────────────────────────────────────────
   🟡 Final Recommendation (as of YYYY-MM-DD)
   ▶ Position: BUY / HOLD / SELL
   ▶ Macro Alignment: FAVORABLE / NEUTRAL / UNFAVORABLE
   ▶ GEX Condition: POSITIVE GAMMA / NEGATIVE GAMMA

   Key Notes:
   - Macro Context: [VIX, yield curve, market trend]
   - GEX Levels: [Support $X, Resistance $Y, Flip $Z]
   - Tactical entry/exit ranges
   - Position sizing guidance
   - Position management (trailing stop, profit target)
   - 1-line Summary Statement
   ─────────────────────────────────────────────────────
```

---

## 13. Step 12 — Agentic AI (Alternative Pipeline)

**Source:** Agentic AI Orchestrator & Specialist Agents — LLM-first sequential analysis pipeline with four domain-expert personas

This is an **LLM-first** alternative to the quantitative pipeline, useful for deeper qualitative analysis.

### Agent Execution Flow

```
User provides ticker(s)
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 1: Macro Analyst (runs ONCE, shared across all tickers)   │
│          Skill: macroeconomic (FRED + VIX + market data)         │
│          Output: RISK-ON / RISK-OFF / TRANSITIONAL              │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 2: Fundamental Analyst (per ticker)                        │
│          Skills: microeconomic + qualitative                     │
│          Data: yfinance financials, DCF, sector comps, news      │
│          Output: OVERVALUED / FAIR / UNDERVALUED + catalysts     │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 3: Quant Analyst (per ticker)                              │
│          Skill: quantitative                                     │
│          Data: Price history, MA structure, RSI/MACD, Sharpe,   │
│                max drawdown, beta, LSTM forecasts, GEX           │
│          Output: Technical signal + risk metrics + entry/exit    │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 4: Portfolio Strategist (synthesis)                        │
│          Receives: all 3 agent analyses                          │
│          Output: BUY/HOLD/SELL + conviction (1-10) +            │
│                  position size + risk parameters                 │
└─────────────────────────────────────────────────────────────────┘
```

### Agent Prompting Philosophy

Each agent has a specific **system prompt persona**:

| Agent | Persona |
|-------|---------|
| Macro Analyst | Senior macro strategist at a global macro hedge fund |
| Fundamental Analyst | Buy-side equity research analyst at a top-tier investment firm |
| Quant Analyst | Quantitative portfolio analyst at a systematic trading firm |
| Portfolio Strategist | Chief Investment Officer at a multi-strategy fund |

### Running the Agentic Pipeline

```python
from agentic_ai.orchestrator import Orchestrator

orchestrator = Orchestrator()

# Single ticker
result = orchestrator.analyze_ticker("AAPL")

# Multiple tickers (macro cached and shared)
results = orchestrator.analyze_portfolio(["AAPL", "TSLA", "NVDA"])
```

---

## 14. Output — Email Report

**Source:** Email Report Dispatcher — HTML report builder and Gmail SMTP sender

The HTML email contains all sections assembled:

```
Subject: "Daily Stock Report: {TICKER}, {Company Name}"

HTML Email Sections:
┌──────────────────────────────────────────────────────────┐
│ 1. ML Analysis Text Report (pre-formatted text)           │
│ 2. Analysis Dashboard Chart (2×2 matplotlib grid):        │
│    ├─ [Top-Left]  Price history (6mo) + MA20/MA50         │
│    ├─ [Top-Right] LSTM 20-day price forecast               │
│    ├─ [Bot-Left]  Sentiment bar chart (News/Reddit/Score) │
│    └─ [Bot-Right] MA backtest returns vs win rates        │
│ 3. Risk vs Return Chart (vs S&P 500 scatter plot)         │
│ 4. Fundamental Data Table (Finviz HTML table)             │
│ 5. QuantStats Summary                                     │
│ 6. AI DCF Valuation Analysis                              │
│ 7. AI Final Summary (with GEX & Macro)                   │
│ 8. Macro Analysis Report                                  │
│ 9. GEX Report (Schwab option chain analysis)              │
│ 10. Covered Call Screener (optional)                      │
└──────────────────────────────────────────────────────────┘
```

---

## 15. Configuration Reference

**Source:** Analysis Configuration Dataclass — All tunable parameters for the ML pipeline


## Quick Reference Summary

```
TICKER SELECTION  →  Finviz: top sector + highest Trade Value stock
FUNDAMENTAL       →  Finviz + yfinance: P/E, margins, targets, insider
DCF               →  AI model: 5yr FCF forecast → WACC → intrinsic price
ML FORECAST       →  LSTM + XGBoost: 60-day lookback → 20-day price forecast
BACKTEST          →  MA strategy (5–50): touch MA → 5% TP / -5% SL
SENTIMENT         →  FinBERT on Finviz news + Reddit (4 subreddits)
QUANT RISK        →  QuantStats: Sharpe, Sortino, VaR, CAGR + Risk/Return plot
MACRO             →  FRED: yields, CPI, unemployment + VIX + BofA signals
GEX               →  Schwab API: dealer gamma → support/resistance/flip levels
WEIGHTED SCORE    →  0.3(news) + 0.2(reddit) + 0.3(ML) + 0.2(BnH) ± macro adj
SIGNAL            →  Score > 0.5 → BUY, else HOLD
AI SUMMARY        →  All data → AI model → final recommendation with entry/exit
```

## 16. Conclusion

That's the entire workflow from ticker selection all the way through to a final buy & hold decision with covered calls on top. Every step feeds into the next: the screener narrows the universe, fundamentals and DCF set a fair-value anchor, ML and backtesting add a forward-looking edge, sentiment captures the crowd's mood, macro and GEX keep me aware of the bigger picture, and the weighted score ties it all together into a single actionable signal.

Is it perfect? No. Markets will always surprise you. But having a structured, repeatable process removes most of the emotion from the decision and lets the data do the heavy lifting. The agentic AI pipeline is a newer addition that gives me a second opinion, a qualitative lens on top of the quantitative one and I've found the two complement each other well.

Building this system has been one of the most rewarding projects I've worked on. If you made it this far, I hope it gave you some ideas for your own workflow. Happy investing.