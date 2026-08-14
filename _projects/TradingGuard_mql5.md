---
layout: page
title: GuardTrades Library for MQL5
description: A Guarding Protective Layer between EAs and Broker
img: assets/img/TradingGuard_Anubis.jpg
importance: 2
category: MQL
related_publications: true
---

1. **Copy the `GuardTrades` folder** into your EA's directory inside the MetaTrader 5 `Experts` folder. Your file structure should look like this:

   ```
   MQL5/
   └── Experts/
      └── YourEAFolder/
         ├── YourEA.mq5
         └── GuardTrades/
            ├── GuardTrades.mqh
            ├── G_Positions.mqh
            ├── G_Symbols.mqh
            ├── G_Calendar.mqh
            ├── G_PriceArea.mqh
            ├── G_Technical.mqh
            └── Indicators/
               └── Fractal.mq5
   ```

2. **Compile the custom indicator** `GuardTrades/Indicators/Fractal.mq5` if you plan to use the Price Area Filter's Fractals feature. Open it in MetaEditor and press **F7** to compile.

3. **Open your EA file** (e.g., `YourEA.mq5`) and add the `#define` macros for the modules you want, then include the library:

   ```mql5
   // Step 1: Define the modules you need (BEFORE the #include)
   #define GUARD_MODULE
   #define SYMBOL_MODULE
   #define CALENDAR_MODULE
   #define PRICE_AREA_MODULE
   #define TECHNICAL_MODULE

   // Step 2: Include the library (path is relative to your EA file)
   #include "./GuardTrades/GuardTrades.mqh"
   ```

4. **Replace `CTrade` with `CGuardTrades`** in your EA. Wherever you previously declared `CTrade trade;`, change it to:

   ```mql5
   CGuardTrades trade;
   ```

5. **Compile your EA** in MetaEditor (F7). All filter input parameters will automatically appear in the EA's settings panel when you attach it to a chart.

<h3 id="step-by-step">Step-by-Step</h3>

<ol class="stepper">
  <li>
    <p><strong>Copy the <code>GuardTrades</code> folder</strong> into your EA's directory inside the MetaTrader 5 <code>Experts</code> folder. Your file structure should look like this:</p>

    <pre><code>MQL5/
└── Experts/
   └── YourEAFolder/
      ├── YourEA.mq5
      └── GuardTrades/
         ├── GuardTrades.mqh
         ├── G_Positions.mqh
         ├── G_Symbols.mqh
         ├── G_Calendar.mqh
         ├── G_PriceArea.mqh
         ├── G_Technical.mqh
         └── Indicators/
            └── Fractal.mq5
    </code></pre>
  </li>

  <li>
    <p><strong>Compile the custom indicator</strong> <code>GuardTrades/Indicators/Fractal.mq5</code> if you plan to use the Price Area Filter's Fractals feature. Open it in MetaEditor and press <strong>F7</strong> to compile.</p>
  </li>

  <li>
    <p><strong>Open your EA file</strong> (e.g., <code>YourEA.mq5</code>) and add the <code>#define</code> macros for the modules you want, then include the library:</p>

    <pre><code>// Step 1: Define the modules you need (BEFORE the #include)
#define GUARD_MODULE
#define SYMBOL_MODULE
#define CALENDAR_MODULE
#define PRICE_AREA_MODULE
#define TECHNICAL_MODULE

// Step 2: Include the library (path is relative to your EA file)
#include "./GuardTrades/GuardTrades.mqh"
    </code></pre>
  </li>

  <li>
    <p><strong>Replace <code>CTrade</code> with <code>CGuardTrades</code></strong> in your EA. Wherever you previously declared <code>CTrade trade;</code>, change it to:</p>

    <pre><code>CGuardTrades trade;
    </code></pre>
  </li>

  <li>
    <p><strong>Compile your EA</strong> in MetaEditor (F7). All filter input parameters will automatically appear in the EA's settings panel when you attach it to a chart.</p>
  </li>
</ol>

## How to Activate Modules
To save performance and memory, filters are modular. You can enable or disable them using preprocessor macros (`#define`) at the very top of your EA, **before** including the library.

```mql5
// Enable desired modules
#define GUARD_MODULE
#define SYMBOL_MODULE
#define CALENDAR_MODULE
#define PRICE_AREA_MODULE
#define TECHNICAL_MODULE

// Include the wrapper after definitions
#include "./GuardTrades/GuardTrades.mqh"
```

If a macro is not defined, its corresponding filter is entirely bypassed and excluded from compilation. For example, if you only need risk management and time filtering:

```mql5
#define GUARD_MODULE
#define CALENDAR_MODULE

#include "./GuardTrades/GuardTrades.mqh"
```

This approach keeps the compiled EA lightweight—only the filters you actually use consume memory and CPU.

## Using the MA Cross Example
The included `MAcross.mq5` file demonstrates a complete working EA that uses all five GuardTrades modules.

### How the example is structured

1. **Module Activation**: At the top of the file, all five modules are enabled via `#define`.
2. **EA Inputs**: Standard strategy inputs are declared (`FastMAPeriod`, `SlowMAPeriod`, `LotSize`, `StopLoss`, `TakeProfit`).
3. **Global Instance**: A single `CGuardTrades trade;` object is created globally.
4. **Initialization (`OnInit`)**: MA indicator handles are created using `trade.ExpertSymbol()` and `trade.ExpertTimeFrame()` — these helper methods return the chart's symbol and timeframe automatically.
5. **Signal Generation**: `buySignal()` and `sellSignal()` detect fast/slow MA crossovers.
6. **Execution (`OnTick`)**: When a signal fires, the EA calls `trade.Buy()` or `trade.Sell()`. **You do not need to manually check risk, spread, time, or any other condition** — the `trade` object runs every enabled filter automatically before executing.
7. **Day-End Cleanup**: `trade.CloseAllPositionsAtDayEnd()` is called every tick to close all expert positions and cancel pending orders at 23:55 server time.

```mql5
#define GUARD_MODULE
#define SYMBOL_MODULE
#define CALENDAR_MODULE
#define PRICE_AREA_MODULE
#define TECHNICAL_MODULE

input int FastMAPeriod = 10;
input int SlowMAPeriod = 30;
input double LotSize = 0.1;
input double StopLoss = 200;
input double TakeProfit = 400;

#include "./filters/GuardTrades.mqh"

// Global instance — replaces CTrade
CGuardTrades trade;

void OnTick()
{
   MqlTick tick;
   if(!SymbolInfoTick(trade.ExpertSymbol(),tick)) return;

   trade.CloseAllPositionsAtDayEnd();

   // Check buy signal
   if(buySignal())
   {
      // Close opposite position first
      if(PositionSelect(trade.ExpertSymbol()) && PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL)
         trade.PositionClose(trade.ExpertSymbol());

      // Open buy — filters are checked automatically inside trade.Buy()
      if(!PositionSelect(trade.ExpertSymbol()))
         trade.Buy(LotSize, trade.ExpertSymbol(), tick.bid,
                   tick.bid - StopLoss * trade.ExpertPoint(),
                   tick.bid + TakeProfit * trade.ExpertPoint());
   }

   // Check sell signal
   if(sellSignal())
   {
      if(PositionSelect(trade.ExpertSymbol()) && PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY)
         trade.PositionClose(trade.ExpertSymbol());

      if(!PositionSelect(trade.ExpertSymbol()))
         trade.Sell(LotSize, trade.ExpertSymbol(), tick.ask,
                    tick.ask + StopLoss * trade.ExpertPoint(),
                    tick.ask - TakeProfit * trade.ExpertPoint());
   }
}
```

## Module Features

### 1. Guard Filter (`G_Positions.mqh`)
This module manages account-level and trade-level risk constraints. It is the most critical safety layer.

**Input Parameters:**
| Input | Default | Description |
|---|---|---|
| `InpMaxPositions` | 5 | Maximum total open positions on the entire account |
| `InpMaxExpertPositions` | 5 | Maximum open positions created by this specific EA |
| `InpMaxLot` | 1.0 | Maximum lot size allowed per single trade |
| `InpMaxTradeRiskPercent` | 2.0% | Maximum risk as a percentage of account balance per trade |
| `InpMaxTradeRiskMoney` | $1000 | Maximum risk in dollars per trade |
| `InpMinFreeMarginPercent` | 100% | Minimum free margin percentage (free margin / equity) |
| `InpMinMarginLevel` | 100% | Minimum margin level before new trades are blocked |
| `InpAllowTradingReal` | true | If false, blocks trading on real (non-demo) accounts |

**Checks Performed:**
* **Environment Safety**: Verifies that trading is enabled in the terminal, the broker allows trading on the account, and the terminal is connected to the server. If the account is a real account and `InpAllowTradingReal` is `false`, trading is blocked.
* **Max Positions**: Counts all open positions on the account and compares against `InpMaxPositions`. Then counts only positions opened by this EA (matched by Magic Number) and compares against `InpMaxExpertPositions`.
* **Lot & Risk Constraints**: Blocks the trade if the requested volume exceeds `InpMaxLot`. For risk calculation, it uses the Stop Loss distance, tick value, and tick size to compute the dollar risk. It then checks both the percentage risk (`riskAmount / balance * 100`) and absolute dollar risk against their respective limits.
* **Margin Safety**: Calculates `(FreeMargin / Equity) * 100` and blocks if below `InpMinFreeMarginPercent`. Also checks the broker-reported Margin Level and blocks if below `InpMinMarginLevel` (only when margin is actively used).
* **Spam Prevention**: Implements a hardcoded 2-second cooldown. If the EA attempts the exact same order type on the same symbol within 2 seconds, the trade is blocked. This protects against EA logic bugs that fire repeated orders on every tick.

---

### 2. Symbol Filter (`G_Symbols.mqh`)
This module validates environmental and broker-level conditions for the specific trading pair.

**Input Parameters:**
| Input | Default | Description |
|---|---|---|
| `InpMaxSpreadPoints` | 30 | Maximum allowed spread in points |
| `InpMaxSymbolPositions` | 3 | Maximum open positions for a single symbol |
| `InpMaxSymbolLot` | 1.0 | Maximum total lot exposure on a single symbol |
| `InpMaxSymbolRiskPercents` | 2.0% | Maximum total risk for a single symbol (% of balance) |
| `InpMaxSymbolRiskMoney` | $1000 | Maximum total risk for a single symbol in dollars |

**Checks Performed:**
* **Symbol Tradability**: Confirms the broker allows orders on the symbol (`SYMBOL_ORDER_MODE`) and that the trade mode is `SYMBOL_TRADE_MODE_FULL`. Symbols in close-only or disabled mode are blocked.
* **Fresh Quotes**: Retrieves the latest tick and verifies it is less than 1 second old. Stale tick data (e.g., during connection issues) will block the trade.
* **Spread Safety**: Reads the current spread in points and blocks the trade if it exceeds `InpMaxSpreadPoints`. Invalid (zero or negative) spreads also trigger a block.
* **Trade Session**: Queries `SymbolInfoSessionTrade()` to check if the market session is currently open. Trades outside session hours are blocked.
* **Liquidity**: Checks the session tick volume (`SYMBOL_SESSION_VOLUME`) against a minimum threshold to avoid trading in illiquid conditions.
* **Volatility**: Calculates intraday volatility as `(High - Low) / Point` and blocks if it falls below a minimum threshold.
* **Execution Safety**: Warns about freeze levels and blocks if the stops level is excessively high (> 50 points).
* **Symbol Exposure**: Scans all open positions for the given symbol and calculates current position count, total lots, and total monetary risk. It then checks whether adding the new trade would exceed `InpMaxSymbolPositions`, `InpMaxSymbolLot`, `InpMaxSymbolRiskMoney`, or `InpMaxSymbolRiskPercents`.

---

### 3. Calendar Filter (`G_Calendar.mqh`)
Controls *when* the EA is permitted to trade based on time and days.

**Input Parameters:**
| Input | Default | Description |
|---|---|---|
| `TradeMonday` | true | Allow trading on Monday |
| `TradeTuesday` | true | Allow trading on Tuesday |
| `TradeWednesday` | true | Allow trading on Wednesday |
| `TradeThursday` | true | Allow trading on Thursday |
| `TradeFriday` | true | Allow trading on Friday |
| `TradeSaturday` | true | Allow trading on Saturday |
| `TradeSunday` | true | Allow trading on Sunday |
| `TimeWindow1Start` | 00:00 | Time Window 1 start (HH:MM format) |
| `TimeWindow1End` | 23:50 | Time Window 1 end (HH:MM format) |
| `TimeWindow2Start` | 00:00 | Time Window 2 start (HH:MM format) |
| `TimeWindow2End` | 00:00 | Time Window 2 end (disabled by default) |
| `TimeWindow3Start` | 00:00 | Time Window 3 start (HH:MM format) |
| `TimeWindow3End` | 00:00 | Time Window 3 end (disabled by default) |

**Checks Performed:**
* **Allowed Days**: Uses the server's current day of the week and checks against the boolean inputs. For example, setting `TradeFriday = false` will block all trades on Fridays.
* **Time Windows**: Converts the start/end strings to minutes since midnight and checks if the current server time falls within any of the 3 defined windows. A window is considered disabled if its start and end are both `00:00`. The EA will only trade if the current time is inside **at least one** active window.
* **Volatile Hours Protection**: As an additional hardcoded rule, high-volume trades (> 1.0 lots) on EURUSD are blocked during the 13:00–15:00 server time window to avoid volatile news-driven hours.

---

### 4. Price Area Filter (`G_PriceArea.mqh`)
Ensures trades only happen in specific structural market zones based on deviation bands. Each sub-filter can be independently enabled or disabled. When enabled, the price **must** be inside the zone for the trade to be allowed.

**Input Parameters:**
| Input | Default | Description |
|---|---|---|
| `EnableHighLow` | false | Enable High/Low zone filter |
| `HighLowTimeframe` | D1 | Timeframe to calculate High/Low |
| `HighLowShift` | 1 | Starting candle to look back |
| `HighLowDays` | 1 | Number of candles to scan for High/Low |
| `HighLowDeviationPct` | 0.1% | Deviation band width around High/Low levels |
| `EnableFractals` | false | Enable Fractals zone filter |
| `FractalTimeframe` | D1 | Timeframe for fractal calculation |
| `FractalDepth` | 5 | Number of recent fractals to consider |
| `FractalDeviationPct` | 0.05% | Deviation band width around fractal levels |
| `EnableRound` | false | Enable Round price zone filter |
| `RoundDigits` | 2 | Rounding precision (e.g., 2 → 1.10, 1.20) |
| `RoundDeviationPct` | 0.02% | Deviation band width around round levels |

**Checks Performed:**
* **High/Low Area**: Copies recent High and Low candle data for the configured timeframe and lookback period. Finds the highest high and lowest low, then checks if the current price falls within a symmetric deviation band (`level ± level * deviation%`). Visual horizontal lines are drawn on the chart showing the band boundaries.
* **Fractals Area**: Uses a custom Bill Williams Fractals indicator (`filters/Indicators/Fractal.mq5`) to identify recent upper and lower fractal levels. The filter checks if the current price is within the deviation band of any recent fractal. The custom indicator must be compiled separately.
* **Round Prices**: Calculates the nearest psychological round number based on `RoundDigits` (e.g., with `RoundDigits=2`, levels are 1.10, 1.20, 1.30, etc.). The trade is only allowed if the price is within the deviation band of that round level.
* **Visual Feedback**: The filter draws horizontal lines on the chart for the upper band, lower band, and the center level, making it easy to visually confirm the active zone.

> **Note:** If multiple sub-filters are enabled (e.g., both High/Low and Round), the price must be inside **all** enabled zones for the trade to pass.

---

### 5. Technical Filter (`G_Technical.mqh`)
Blocks trades if current market momentum or trend contradicts standard technical indicators. Each indicator filter can be independently enabled or disabled.

**Input Parameters:**
| Input | Default | Description |
|---|---|---|
| `EnableADX` | false | Enable ADX trend strength filter |
| `ADXTimeframe` | Current | Timeframe for ADX calculation |
| `ADXPeriod` | 14 | ADX indicator period |
| `ADXThreshold` | 25.0 | Minimum ADX value to allow trading |
| `EnableATR` | false | Enable ATR volatility filter |
| `ATRTimeframe` | Current | Timeframe for ATR calculation |
| `ATRPeriod` | 14 | ATR indicator period |
| `ATRThreshold` | 0.01 | Minimum ATR value to allow trading |
| `EnableMA` | false | Enable Moving Average alignment filter |
| `MATimeframe` | Current | Timeframe for MA calculation |
| `MAPeriod` | 50 | MA period |
| `MAMethod` | SMA | MA calculation method (SMA, EMA, etc.) |
| `MAPrice` | Close | Applied price for MA |
| `EnableRSI` | false | Enable RSI overbought/oversold filter |
| `RSITimeframe` | Current | Timeframe for RSI calculation |
| `RSIPeriod` | 14 | RSI period |
| `RSIOverbought` | 70.0 | RSI level above which Buys are blocked |
| `RSIOversold` | 30.0 | RSI level below which Sells are blocked |
| `EnableATRCompare` | false | Enable ATR(x) > ATR(y) comparison filter |
| `ATRPeriodX` / `ATRPeriodY` | 14 / 14 | Periods for the two ATR values |
| `ATRTimeframeX` / `ATRTimeframeY` | Current / H1 | Timeframes for the two ATR values |
| `EnableMACD` | false | Enable MACD momentum filter |
| `MACDFast` / `MACDSlow` / `MACDSignal` | 12 / 26 / 9 | MACD EMA periods |
| `MACDPrice` | Close | Applied price for MACD |

**Checks Performed:**
* **ADX Filter**: Reads the ADX indicator value. If `ADX <= ADXThreshold`, the trade is blocked. This ensures the market is trending strongly enough to justify entry.
* **ATR Filter**: Reads the ATR value. If `ATR <= ATRThreshold`, the trade is blocked. This prevents entries in extremely low-volatility environments where price movement may not reach the Take Profit.
* **MA Alignment Filter**: For buy-type orders, the current price must be **above** the Moving Average. For sell-type orders, the price must be **below** the MA. This ensures trades are aligned with the broader trend direction.
* **RSI Filter**: For buy-type orders, blocks the trade if `RSI >= RSIOverbought` (e.g., 70). For sell-type orders, blocks if `RSI <= RSIOversold` (e.g., 30). This prevents entering against extreme momentum.
* **ATR Compare Filter**: Compares two ATR values calculated on different timeframes/periods. The trade is only allowed if `ATR(x) > ATR(y)`. This is useful for confirming that short-term volatility exceeds longer-term norms.
* **MACD Filter**: For buy-type orders, the MACD line must be **above** the Signal line. For sell-type orders, the MACD line must be **below** the Signal line. This confirms momentum direction before entry.

---

## Utility Methods

The `CGuardTrades` class also provides several helper methods beyond filtering:

| Method | Description |
|---|---|
| `ExpertSymbol()` / `ExpertSymbol(string)` | Get or set the EA's working symbol (defaults to `_Symbol`) |
| `ExpertTimeFrame()` / `ExpertTimeFrame(ENUM_TIMEFRAMES)` | Get or set the EA's working timeframe (defaults to `_Period`) |
| `ExpertPoint()` / `ExpertDigits()` | Get the chart's point size and digit count |
| `ExpertPositionsTotal()` | Count open positions created by this EA (matched by Magic Number) |
| `ExpertOrdersTotal()` | Count pending orders created by this EA |
| `CloseAllPositionsAtDayEnd()` | At 23:55 server time, closes all EA positions and cancels pending orders |

---

## Contributing & Roadmap

`GuardTrades` is an **open-source project** and we warmly invite the trading community to contribute, improve, and extend it. Whether you are an experienced MQL5 developer or just getting started, there are many ways to help:

### Ideas for New Modules
- **News Filter**: A real-time economic calendar filter using MQL5's built-in `Calendar*` functions to block trades around high-impact news events (NFP, CPI, FOMC, etc.).
- **Correlation Filter**: Prevent opening correlated positions (e.g., long EURUSD and long GBPUSD simultaneously) to reduce portfolio risk.
- **Drawdown Filter**: Block new trades when the account is experiencing a drawdown beyond a defined threshold.
- **Session Filter**: Advanced session-based filtering (Asian, London, New York sessions) with customizable overlap handling.
- **Equity Curve Filter**: Only allow trades when the EA's equity curve is above its own moving average (meta-strategy filtering).

### How to Contribute
1. **Fork** the repository
2. **Create a new filter** following the existing module pattern (create a class with an `ApplyFilter()` method)
3. **Add a `#define` macro** and conditional includes in `GuardTrades.mqh`
4. **Submit a Pull Request** with a clear description of what your filter does

### Reporting Issues
If you find a bug, have a suggestion, or want to request a feature, please open an issue on the repository. All feedback is valuable.

---

> **Every EA deserves a guardian.** Help us build the most comprehensive trade safety library for MQL5. Together we can make algorithmic trading safer for everyone.
