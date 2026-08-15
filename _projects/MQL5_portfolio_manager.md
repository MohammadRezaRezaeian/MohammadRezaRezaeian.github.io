---
layout: page
title: Portfolio Manager in MQL5
description: An OOP Designed Structure to Handle Portfolio Execution Complexity
img: assets/img/PortfolioManager.jpg
importance: 1
category: MQL
related_publications: true
medium_url: https://medium.com/@rezaeian.moh
---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Architecture](#architecture)
- [Architecture Flowchart](#architecture-flowchart)
- [How It Works](#how-it-works)
- [Issues and Challenges](#issues-and-challenges)
- [Tasks](#tasks)
- [Extensibility](#extensibility)
- [License](#license)

## Overview

The primary goal of this project is to minimize dependencies within the MQL5 Expert Advisor framework while maintaining flexibility and extensibility. MQL5 provides predefined event-based global functions, including OnInit, OnTick, OnTradeTransaction, OnChartEvent, and OnDeinit. Codes are designed to reduce code dependencies during these function calls by clearly defining their roles. This modular approach ensures the program efficiently handles portfolio management tasks, such as strategy initialization, trade execution, and user interface updates, within each event-driven function.

## Architecture

PortfolioEA's architecture is designed to be modular and extensible, centered around a StrategyFactory that creates user-defined components such as trading strategies, indicators, portfolio rules, open positions, orders, and interface settings. These components are initialized when the Expert Advisor's parameters are set, ensuring a clear starting point. To prevent unintended changes to variables during runtime, a singleton PortfolioManager module acts as a mediator, safeguarding variable values and preventing unnecessary reinitialization. The singleton pattern ensures that the PortfolioManager is instantiated only once, avoiding redundant initialization unless explicitly deleted.

To address the challenges of repetitive instructions, which impact both system accuracy and processing speed, PortfolioEA employs the Chain of Responsibility design pattern. By organizing strategy components into distinct tasks, the system filters which signals to process, optimizing efficiency. Additionally, a signal request object is used to carry all necessary data for trade execution. This object is passed through the task pipeline, where it is filtered and enriched with relevant data at each stage, ensuring precise and streamlined signal handling.

## Architecture Flowchart

<div class="text-center">
    <img src="{{ '/assets/img/flowcharts/PortfolioManager_MQL5.svg' | absolute_url }}" alt="Architecture Flowchart" class="img-fluid">
</div>

## How It Works

Expert begins by defining trading strategies as classes stored in the strategies folder. These strategies must be manually included in the StrategyFactory module due to MQL5's inability to create dynamic instances. Portfolio limitations and task settings are also defined, typically once per strategy set, with support for extensibility. Upon launching the Expert Advisor, it automatically initializes strategies, tasks, open positions, orders, and initial variables. During real-time execution, the system evaluates all strategy instances. Strategies that pass the initial task filter generate a ticket as an instance that encapsulates order requests, strategy settings, and necessary data for trade execution. Each task in the pipeline either filters out tickets by removing them or enriches their data. The final task validates and sends order requests to the broker. Whenever a trade transaction occurs, the Expert Advisor refreshes open positions and orders via the OnTradeTransaction function, updating the system to continue the cycle.

## Issues and Challenges

Designing a portfolio management system like this involves navigating several challenges, primarily due to the extensive dependencies inherent in coordinating multiple trading strategies and portfolio rules. These dependencies can lead to calculation errors and degrade runtime performance if not managed effectively. In programming such systems, numerous conditions must be evaluated continuously, and these conditions often interact with one another, increasing complexity. For instance, a change in one strategy's execution may impact portfolio-level constraints, requiring careful synchronization to maintain accuracy.

## Tasks

To address the challenges of managing dependencies and optimizing performance, This project employs a task-based approach using the Chain of Responsibility design pattern. Each task is designed to handle specific limitations that determine which trading signals should be executed, ordered to minimize processing overhead and enhance execution speed. By breaking down the process into distinct tasks, the system efficiently filters and processes signals, ensuring that only valid signals proceed through the pipeline. The tasks are structured as follows:

1. **Calendaer**:
   - Filters out signals during market times when trading is not desired, based on calendar events, preventing unnecessary processing. This task leverages the OnTimer function to periodically check market conditions, reducing the need to process every tick received in the OnTick function, thereby optimizing performance.
2. **Strategy/Symbol Calendaer**:
   - Validates strategies by checking their specific calendar constraints, symbol trading hours, and execution permissions. This task ensures that each strategy and its associated symbols comply with broker and account restrictions, as well as strategy-specific execution time rules, filtering out signals that do not meet these criteria early in the pipeline.
3. **PortfolioRulesTask**:
   - forces portfolio-level constraints, which include both inherent limitations derived from the portfolio's nature (e.g., risk thresholds, capital allocation) and user-defined rules (e.g., maximum exposure, position limits). This task also populates ticket data, such as trading volume, ensuring that valid signals are enriched with necessary parameters for execution.
4. **SignalObserverTask**:
   - Evaluates strategy logic to generate entry and exit signals based on predefined conditions. To handle complex or multiple strategies efficiently, it is strongly recommended to implement the Observer Design Pattern, allowing flexible monitoring of strategy signals. This task also enriches tickets with additional request data, such as signal-specific parameters, to prepare them for final execution.
5. **OrderHandleTask**:
   - Performs a final validation of order requests before sending them to the broker. This task ensures that all required request properties, populated during the pipeline process, are complete and contain valid data, guaranteeing accurate and reliable order execution.

Each task can potentially use the Observer Design Pattern to handle multiple rules or strategies, although this is not implemented in the current version for simplicity.

## Extensibility

The framework is designed with a strong emphasis on extensibility, prioritizing flexibility to allow seamless integration of new features over performance optimization. The modular architecture enables independent addition of capabilities such as user interfaces, statistical reports, or WebSocket communication for external data transmission. All necessary data is centralized within the PortfolioManager mediator, making it easily accessible for new modules to read and utilize. Additionally, advanced portfolio controls can be implemented by adding new tasks to the Chain of Responsibility pipeline. While this process is straightforward, it requires careful design since each task takes the output of the previous task as its input, ensuring proper data flow and compatibility.

## License

This project is licensed under the TradeYaar License - see the [LICENSE](https://github.com/MohammadRezaRezaeian/MohammadRezaRezaeian.github.io/blob/main/_projects/MQL5_portfolio_manager/LICENSE.md) file for details.
