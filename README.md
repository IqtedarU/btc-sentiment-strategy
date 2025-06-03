# Sentiment-Driven BTC Trading Strategy

This project implements a rule-based trading system for Bitcoin using sentiment extracted from daily crypto news. The strategy integrates semantic filtering, transformer-based sentiment scoring, and risk-controlled trade execution to assess whether news tone can systematically guide BTC exposure.

## Project Overview

The strategy simulates daily trades based on smoothed sentiment signals extracted from crypto news articles. Articles are filtered by semantic relevance to Bitcoin before being aggregated. Each day’s average sentiment is translated into a trading signal, with trade execution subject to real-world constraints like slippage, transaction costs, and stop-loss logic. The strategy’s performance is compared to a benchmark buy-and-hold BTC approach over the same period.

## Data and Setup

- **News Source**: Crypto news articles collected using the query  
  `"Bitcoin cryptocurrency blockchain BTC halving"`  
  This query aims to target major macro and technical themes relevant to BTC.

- **Sentiment Scoring**: Sentiment is computed using a transformer-based SentenceTransformer model. Scores represent overall article tone and are continuous-valued, not just positive/negative classification.

- **Semantic Relevance**: Each article is scored for Bitcoin-specific relevance using a semantic similarity model. Only articles exceeding a given relevance threshold (e.g. 0.2) are considered in signal generation.

## Backtest Design

- **Training Period**: 2018-05-01 to 2022-05-01  
  A grid search is performed on this period to optimize parameters.

- **Testing Period**: 2022-05-02 to 2025-01-01  
  The best-performing strategy from training is applied to this period without modification.

## Strategy Mechanics

- **Signal Construction**:
  - Articles are grouped by date.
  - Daily average sentiment is computed from relevant articles.
  - Sentiment is smoothed using a rolling window and scaled to determine position size.
  - Trades are entered **at the open of the next day** following signal generation.

- **Execution Constraints**:
  - **Transaction Cost**: 0.1% per trade leg.
  - **Slippage**: 0.15% per trade leg.
  - **Stop-loss**: Triggered at 3% adverse move from entry price.
  - All returns are calculated in log-return space for realism and stability.

## Grid Search (Train Period)

A grid search was conducted over the following parameters:

- **Relevance Thresholds**: [0.2, 0.3, 0.4]  
- **Sentiment Thresholds**: [0.2, 0.3, 0.4, 0.5]  
- **Smoothing Windows**: [3, 5, 7]  
- **Stop Loss**: [0.03, 0.05, 0.08]  

The best configuration based on Sharpe ratio was:

- Relevance: **0.2**  
- Sentiment Threshold: **0.2**  
- Smoothing Window: **7 days**  
- Stop-loss: **3%**  

## Strategy Results (Test Set)

| Metric               | Value       |
|----------------------|-------------|
| Sharpe Ratio         | 2.92        |
| Sharpe CI (95%)      | [2.90, 2.93]|
| Annualized Return    | 93.81%      |
| Max Drawdown         | -18.02%     |
| Number of Trades     | 575         |
| Win Rate             | 39.07%      |
| Profit Factor        | 1.66        |
| Sortino Ratio        | 8.40        |
| Calmar Ratio         | 5.2068      |

The strategy was consistently profitable, with very strong risk-adjusted metrics and relatively low drawdown. The low win rate is offset by strong average trade payoff and solid downside protection.

## Benchmark: Buy-and-Hold BTC (Test Set)

| Metric               | Value       |
|----------------------|-------------|
| Total Return         | 532.06%     |
| Annualized Return    | 144.20%     |
| Max Drawdown         | -26.15%     |

Although the benchmark achieved higher raw return due to the tail end of a bull run, it experienced larger drawdowns and far lower risk-adjusted returns than the sentiment-based strategy.

## Sensitivity Analysis

To test the robustness of the strategy, I look at how performance varies with each parameter:

### Relevance Threshold

| Relevance | Avg. Sharpe |
|-----------|-------------|
| 0.2       | 2.82        |
| 0.3       | 2.82        |
| 0.4       | 1.36        |

> Lower thresholds yield better Sharpe, suggesting that broader article inclusion improves performance.

### Sentiment Threshold

| Sentiment | Avg. Sharpe |
|-----------|-------------|
| 0.2       | 2.54        |
| 0.3       | 2.50        |
| 0.4       | 2.23        |
| 0.5       | 2.06        |

> Lower sentiment thresholds lead to stronger signals. Overly strict sentiment cutoffs reduce performance.

### Smoothing Window

| Window | Avg. Sharpe |
|--------|-------------|
| 3      | 2.26        |
| 5      | 2.31        |
| 7      | 2.43        |

> Longer smoothing helps reduce noise and improve signal quality.

### Stop Loss

| Stop Loss | Avg. Sharpe |
|-----------|-------------|
| 0.03      | 3.62        |
| 0.05      | 2.24        |
| 0.08      | 1.14        |

> Tight stop-losses are critical in reducing drawdowns and enhancing Sharpe.

### Interaction: Relevance × Sentiment (Sharpe)

| Relevance ↓ / Sentiment → | 0.2  | 0.3  | 0.4  | 0.5  |
|---------------------------|------|------|------|------|
| 0.2                       | 3.05 | 2.98 | 2.72 | 2.53 |
| 0.3                       | 3.05 | 2.98 | 2.72 | 2.53 |
| 0.4                       | 1.54 | 1.52 | 1.26 | 1.13 |

> Best performance is observed at low Relevance and Sentiment thresholds, affirming the benefit of a broad filter and early signal capture.

## Improvements & Future Work

Several extensions and enhancements can further improve the robustness, accuracy, and generalizability of this strategy:

- **Advanced Sentiment Models**: Replace SentenceTransformer scoring with finance-specific models like FinBERT, which may offer more accurate sentiment extraction in economic and crypto contexts.

- **Query Diversification**: Expand or test alternative queries (e.g., `"bitcoin regulation"`, `"crypto ETF"`, `"BTC liquidity mining"`) to capture emerging narratives or regime shifts.

- **Dynamic Semantic Relevance Models**: Fine-tune relevance scoring by experimenting with newer sentence embedding models (e.g., OpenAI, Cohere, or LLM-based embedding APIs) or by learning relevance directly from market impact labels.

- **Adaptive Thresholds**: Allow relevance or sentiment thresholds to change over time using volatility scaling or regime-switching logic.

