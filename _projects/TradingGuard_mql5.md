---
layout: page
title: TradeFilters Framework for MQL5
description: A Filtering Protective Layer between EAs and Broker
img: assets/img/TradingGuard_Anubis.jpg
importance: 2
category: MQL
related_publications: true
---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Architecture](#architecture)
- [Architecture Flowchart](#architecture-flowchart)
- [How It Works](#how-it-works)
- [Filter Modules](#filter-modules)
  - [GuardFilter – Account and Risk Protection](#guardfilter--account-and-risk-protection)
  - [SymbolFilter – Market and Symbol Quality](#symbolfilter--market-and-symbol-quality)
  - [CalendarFilter – Days, Hours, and News Control](#calendarfilter--days-hours-and-news-control)
  - [PriceAreaFilter – Price Zone Control](#priceareafilter--price-zone-control)
  - [TechnicalFilter – Indicator-Based Confluence](#technicalfilter--indicator-based-confluence)
- [Sample Expert Advisor](#sample-expert-advisor)
- [How to Use](#how-to-use)
- [Benefits](#benefits)
- [Extensibility](#extensibility)
- [License](#license)

## Overview

The TradeFilters Framework is a modular trading protection and validation system for MQL5 that acts as a universal gatekeeper between your Expert Advisor and the broker.[web:10][web:33] The primary goal is to separate trading logic from trading safety: your EA focuses on generating signals while the framework decides whether those signals are safe to execute. This design reduces catastrophic mistakes, makes risk rules reusable across multiple EAs, and allows you to toggle or tune filters without touching core strategy code.[web:38]

The framework extends the standard MQL5 `CTrade` class with a mediator pattern that orchestrates multiple independent filter modules.[web:33] Each trade request passes through a sequential validation chain covering account protection, market quality, time windows, price zones, and technical confluence before reaching the broker.

## Architecture

The architecture centers around the **TradeFilters class**, which inherits from MQL5's `CTrade` and acts as a single gateway for all trading operations.[web:33][web:38] The mediator manages five independent filter modules that are conditionally compiled based on preprocessor flags:

- **GuardFilter**: Account protection (risk limits, margin, position caps, environment checks)
- **SymbolFilter**: Market quality validation (spread, liquidity, session, volatility)
- **CalendarFilter**: Time-based trading rules (days, hours, news windows)
- **PriceAreaFilter**: Price zone control (high/low bands, fractals, round numbers)
- **TechnicalFilter**: Indicator-based confirmation (ADX, ATR, MA, RSI, MACD)

Each module is instantiated only when its corresponding `#define` flag is set, allowing lightweight configurations for specific needs.[web:10] The mediator overrides all major `CTrade` methods to inject the filter chain before broker execution.[web:33]

The framework uses a **fail-fast validation chain**: when any filter rejects a trade, the chain stops immediately with detailed logging, and control returns to the EA without sending an order. Only when all enabled filters approve does the underlying `CTrade` method execute.[web:10][web:33]

## Architecture Flowchart

<div class="text-center">
    <img src="{{ '/assets/img/flowcharts/TradeFilters_Architecture.svg' | relative_url }}" alt="TradeFilters Architecture Flowchart" class="img-fluid">
</div>

## How It Works

The Expert Advisor begins by defining which filter modules to activate using preprocessor directives at the top of the file. After including the main `tradingFilter.mqh` header, a single `TradeFilters` object is created that serves as the trading interface throughout the EA's lifetime.[web:10]

During initialization, the mediator instantiates only the enabled filter modules and configures expert context (symbol, timeframe, point value, digits). The EA accesses these via helper methods when creating indicators or calculating stop loss values.

At runtime, whenever the EA's signal logic detects an entry or exit condition, it calls a mediator method such as `Buy()` or `Sell()` instead of using `CTrade` directly. The mediator immediately invokes its internal filter chain with the full trade context.[web:33]

Inside the filter chain, each enabled filter receives the trade parameters and performs its validation checks in strict sequence. If any filter returns a rejection, the mediator logs the specific reason and returns control to the EA without executing the trade.[web:10] If all filters pass, the original `CTrade` method is invoked and the order request is sent to the broker.[web:33]

The framework also provides optional housekeeping functionality that automatically closes the EA's positions and cancels pending orders near day-end based on the magic number.

## Filter Modules

### GuardFilter – Account and Risk Protection

GuardFilter provides account-level protection by validating trading environment, connectivity, and risk parameters before allowing any trade.[web:33] It checks terminal and broker permissions, margin adequacy as a percentage of equity, maximum position counts per EA, per-trade volume limits, and stop-loss-based risk calculations in both percentage and absolute terms. All failures trigger detailed log messages for troubleshooting.

### SymbolFilter – Market and Symbol Quality

SymbolFilter ensures the chosen symbol and current market conditions meet minimum quality standards.[web:38] It validates broker tradability, spread thresholds, trading session status, liquidity through session volume, volatility ranges to avoid flat markets, and execution constraints like freeze levels and minimum stop distances. This module acts as a quality gate that prevents trading during adverse market microstructure conditions.

### CalendarFilter – Days, Hours, and News Control

CalendarFilter makes strategies time-aware by enforcing day-of-week and hour-of-day restrictions.[web:8][web:41] It provides individual toggles for each weekday, supports up to three independent time windows per day for session-based strategies, includes placeholder integration for economic news blackout periods, and allows symbol-specific volume restrictions during volatile hours. This eliminates weekend gaps and avoids low-liquidity periods.

### PriceAreaFilter – Price Zone Control

PriceAreaFilter controls where in the price ladder trades are permitted by defining forbidden zones around key technical levels. It creates deviation bands around recent highs and lows over configurable lookback periods, identifies Bill Williams Fractal turning points as support and resistance zones, and marks round number levels as psychological barriers. Trades are blocked when price enters any enabled zone, helping avoid false breakouts and congestion areas while encouraging entries in momentum regions.

### TechnicalFilter – Indicator-Based Confluence

TechnicalFilter adds technical confirmation layers on top of raw strategy signals.[web:10][web:42] It supports optional requirements for trend strength via ADX thresholds, volatility gates using ATR minimums, trend direction alignment with moving averages, momentum extreme filtering through RSI overbought/oversold levels, volatility expansion comparisons between timeframes, and momentum direction confirmation via MACD crossovers. Each sub-filter is independently toggled, allowing flexible single-indicator checks or full multi-indicator confluence systems.

## Sample Expert Advisor

The framework includes a minimal Moving Average crossover EA that demonstrates integration.[web:11] The EA generates signals based on fast and slow MA crosses and routes all trades through the mediator object instead of using `CTrade` directly.[web:33] This automatically subjects every trade to the full filter chain while keeping strategy code focused purely on signal detection. The sample shows proper use of expert context helpers, stop loss calculations, and optional end-of-day position cleanup.

## How to Use

To integrate the framework into a new Expert Advisor:

1. Define desired filter modules at the top of your EA using preprocessor directives
2. Include the library header and create a single mediator object
3. Replace all direct `CTrade` calls with mediator methods for orders and positions
4. Use expert context helpers when creating indicators and calculating prices
5. Configure filter input parameters to tune behavior for your strategy

The mediator automatically applies all enabled filters to every trade request, with failures logged clearly for troubleshooting.[web:10][web:33]

## Benefits

**Robust Safety**: Multi-layered account protection prevents dangerous trading conditions and excessive risk exposure.[web:33]

**Quality Focus**: Filters across time, market structure, price location, and technical indicators ensure only high-probability setups execute.

**Clean Code**: Complete separation between signal generation and safety validation reduces EA complexity and cognitive load.[web:10]

**Reusability**: Standardized risk management layer works across multiple strategies and trading styles.[web:38]

**Flexibility**: Independent filter modules can be toggled, tuned, or extended without modifying strategy logic.[web:10]

## Extensibility

The framework supports straightforward extension through its modular architecture. New filters can be added by creating classes with standard validation interfaces and integrating them into the mediator under new preprocessor flags.[web:10] Existing filters can be enhanced independently without affecting other modules, such as connecting real economic calendar data to CalendarFilter.[web:41] The mediator itself can be extended with additional context methods for consistency across strategies.[web:33] The clear separation of concerns and well-defined interfaces make the framework adaptable to diverse trading styles, risk management philosophies, and portfolio management systems.[web:10][web:38]

## License

This project is licensed under the TradeYaar License - see the [LICENSE](/projects/MQL5_tradefilters/LICENSE.md) file for details.
