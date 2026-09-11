---
layout: page
title: The Psychology of Price
description: Quantization of market regimes!
img: assets/img/regimes.jpg
importance: 3
category: Quant Analysis
related_publications: true
---

# The Psychology of Price: A Quantitative Framework for Market Regimes

When analyzing a single asset, we must consider that it is a human system, not a random walk drawn from a specific statistical distribution. An asset falls into different regimes driven by the behavior of its human buyers and sellers. From a quantitative point of view, we can evaluate this variety using three primary metrics: Trend (Future Value Mean), Volatility (Variance), and Irregularity (Stationarity vs. Non-Stationarity).

To see how these metrics apply, let's first discuss various market regimes, their underlying causes, and how they are represented in asset price action:

## 1- Soft Trend
*   **Candlestick Properties:** Small/medium bodies, almost consistent color, small shadows, mild overlapping.
*   **Price Property:** Steady, persistent directional movement with shallow pullbacks.
*   **Economic Cause:** Consistent capital, stable interest rates, steady institutional accumulation.
*   **Human Cause:** Rational valuation, gradual accumulation, and stable confidence.

<img src="{{ '/assets/img/The_Psychology_of_Price/Soft Trend.jpg' | absolute_url }}" alt="Soft Trend" width="100%">

## 2- Ranging Market
*   **Candlestick Properties:** Alternating red/green colors, frequent Dojis, long shadows near boundaries.
*   **Price Property:** Price bounces horizontally between defined support and resistance.
*   **Economic Cause:** Equilibrium in valuation; waiting for new data or catalysts.
*   **Human Cause:** Split beliefs and a divided consensus among market participants regarding the asset's true value.

<img src="{{ '/assets/img/The_Psychology_of_Price/Ranging Market.jpg' | absolute_url }}" alt="Ranging Market" width="100%">

## 3- Volatility Compression
*   **Candlestick Properties:** Progressively shrinking bodies, more inside bars, and drying up volume.
*   **Price Property:** Progressively tighter price range with Progressively shrinking trading volume.
*   **Economic Cause:** Awaiting a major binary economic catalyst or macro decisions.
*   **Human Cause:** Risk aversion, hesitation to commit massive capital before news.

<img src="{{ '/assets/img/The_Psychology_of_Price/Volatility Compression.jpg' | absolute_url }}" alt="Volatility Compression" width="100%">

## 4- Volatile Trend
*   **Candlestick Properties:** Large bodies, long shadows, frequent engulfing patterns continuing a direction.
*   **Price Property:** General upward/downward direction but with wide, aggressive swings.
*   **Economic Cause:** Conflicting macro data; shifting liquidity causing value reassessment.
*   **Human Cause:** Uncertainty and fear clashing with FOMO.

<img src="{{ '/assets/img/The_Psychology_of_Price/Volatile Trend.jpg' | absolute_url }}" alt="Volatile Trend" width="100%">

## 5- Volatile Range
*   **Candlestick Properties:** Erratic, massive alternating candles failing to establish any direction.
*   **Price Property:** Horizontal movement characterized by aggressive, wide swings.
*   **Economic Cause:** High uncertainty and rapid reactions to conflicting news headlines.
*   **Human Cause:** Confusion, lack of consensus.

<img src="{{ '/assets/img/The_Psychology_of_Price/Volatile Range.jpg' | absolute_url }}" alt="Volatile Range" width="100%">

## 6- Mean Reversion
*   **Candlestick Properties:** Extreme reversal candles followed by Morning/Evening Star reversals.
*   **Price Property:** Extreme price move quickly snaps back toward a historical moving average.
*   **Economic Cause:** Institutional profit-taking and fundamental reassessment of the asset's baseline valuation.
*   **Human Cause:** Profit-taking by momentum traders and a collective psychological anchoring to the asset's historical average price.

<img src="{{ '/assets/img/The_Psychology_of_Price/Soft Trend.jpg' | absolute_url }}" alt="Soft Trend" width="100%">

## 7- Impulses & Breakouts
*   **Candlestick Properties:** Massive Marubozu candles (long bodies, short wicks) closing near highs/lows.
*   **Price Property:** Sudden, high-momentum spikes or drops that shatter previous ranges.
*   **Economic Cause:** Earnings surprises or unexpected macro data resetting baselines.
*   **Human Cause:** Sudden urgency, panic buying/selling, algorithmic momentum.

<img src="{{ '/assets/img/The_Psychology_of_Price/Mean Reversion.jpg' | absolute_url }}" alt="Mean Reversion" width="100%">

## 8- Gap and Go
*   **Candlestick Properties:** Blank space on the chart followed by a strong continuing candle.
*   **Price Property:** Price opens significantly higher/lower than prior close and keeps running.
*   **Economic Cause:** Overnight fundamental shocks (buyouts, surprise earnings).
*   **Human Cause:** Trapped traders panicking to cover bad positions to survive.

<img src="{{ '/assets/img/The_Psychology_of_Price/Gap and Go.jpg' | absolute_url }}" alt="Gap and Go" width="100%">

## 9- Liquidity Sweeps
*   **Candlestick Properties:** Pin Bars / Hammers; long wicks piercing boundaries but bodies closing inside.
*   **Price Property:** Brief break of support/resistance followed by a violent reversal.
*   **Economic Cause:** Institutions forcing price into stop-losses to fill block orders.
*   **Human Cause:** Automated stop-losses to trap premature breakout traders.

<img src="{{ '/assets/img/The_Psychology_of_Price/Liquidity Sweeps.jpg' | absolute_url }}" alt="Liquidity Sweeps" width="100%">

## 10- Blow-Off / Capitulation
*   **Candlestick Properties:** Massive climax full body candles followed instantly by huge opposite-color engulfing candles.
*   **Price Property:** Vertical acceleration on massive volume ending in a permanent reversal.
*   **Economic Cause:** Large institutions push the price trigger wave of orders, creating liquidity to exit.
*   **Human Cause:** Smart money trap, Latecomers are lured into the trap by extreme momentum.

<img src="{{ '/assets/img/The_Psychology_of_Price/Blow-Off Capitulation.jpg' | absolute_url }}" alt="Blow-Off Capitulation" width="100%">

***

## Conclusion

Based on the underlying drivers of these market regimes, asset prices fundamentally change because macroeconomic events reshape their true value, prompting buyers and sellers to adjust their positioning. This continuous recalibration of underlying value dictates the asset's future **Trend** (the moving mean).

Simultaneously, market participants constantly make estimation errors, react to institutional traps, and rapidly shift their attention based on breaking news. These sudden behavioral shocks are the event-driven elements of the market that break historical patterns, causing the price series to become unpredictable and **Non-Stationary**.

The third factor is the degree to which these events divide the consensus of the crowd. When market participants are sharply split between hope and fear, or when fundamental understandings diverge, this psychological separation manifests directly as **Volatility** (variance).

By actively measuring Trend, Volatility, and Stationarity together, we can mathematically detect these human and economic shifts in real-time. This framework allows us to identify structural price-altering events as they happen, dynamically adapting to new market regimes without losing the broader context of market history.
