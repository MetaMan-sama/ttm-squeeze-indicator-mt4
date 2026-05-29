# TTM Squeeze Indicator Alert — MQL4 Script

A MetaTrader 4 script that detects **TTM Squeeze consolidation and breakout conditions** by comparing Bollinger Band width against Keltner Channel width each cycle — the squeeze is "on" when both Bollinger Band boundaries are inside the Keltner Channel boundaries — and fires state-change alerts when the squeeze transitions from off to on (consolidation entering) or on to off (breakout imminent) using a persistent `IsSqueezeOn` boolean global.

---

## Overview

The TTM Squeeze, developed by John Carter and popularized through his book *Mastering the Trade*, is a momentum-based volatility contraction detector built on a precise structural relationship between two volatility envelopes. Bollinger Bands — built on standard deviation — and Keltner Channels — built on ATR — both measure price envelope width, but they respond differently to volatility regimes. The squeeze condition identifies the specific market state when Bollinger Bands have contracted inside Keltner Channels: `bbUpper < kcUpper && bbLower > kcLower`. This structural compression signals that the market is coiling — volatility is at a relative minimum and a breakout is statistically likely in the near term. The direction of the breakout is determined by momentum indicators applied after the squeeze fires; the squeeze itself signals timing. When the Bollinger Bands re-expand outside the Keltner Channels, the squeeze is "released" — the breakout has begun and momentum is expanding. This script monitors both transitions: squeeze entry (compression) and squeeze release (expansion), giving traders advance warning of both the setup and the trigger.

---

## Features

- **Bollinger Band vs Keltner Channel structural comparison** — `bbUpper = iBands(..., BBPeriod, BBDeviation, ..., MODE_UPPER, 0)`; `bbLower = iBands(..., MODE_LOWER, 0)`; `kcMiddle = iMA(..., KCPeriod, 0, MODE_EMA, PRICE_TYPICAL, 0)`; `atr = iATR(..., KCPeriod, 0)`; `kcUpper = kcMiddle + KCMultiplier × atr`; `kcLower = kcMiddle − KCMultiplier × atr`
- **Squeeze state boolean return** — `CalculateTTMSqueeze()` returns `true` when `bbUpper < kcUpper && bbLower > kcLower` (squeeze on); `false` when Bollinger Bands are outside Keltner Channels (squeeze off)
- **State-change-only alerting** — `IsSqueezeOn != currentSqueezeState` gate ensures alerts fire only on transitions: `currentSqueezeState == true` → **Squeeze On (Consolidation Detected)**; `currentSqueezeState == false` → **Squeeze Off (Breakout Detected)** — not on every cycle where the state is unchanged
- **`IsSqueezeOn` persistent state** — global bool updated to `currentSqueezeState` immediately after each state-change detection
- **Sufficient-data guard** — `iBars() < BBPeriod || iBars() < KCPeriod` returns `false` before any indicator calls
- **Independent BB and KC period inputs** — `BBPeriod`, `BBDeviation`, `KCPeriod`, and `KCMultiplier` all independently configurable for strategy-specific tuning
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateTTMSqueeze()` fetches BB and KC values; returns `true` if BB is inside KC
2. `IsSqueezeOn != currentSqueezeState` gate fires alert on transition only:
   - Transition to `true` → **Squeeze On (Consolidation Detected)**
   - Transition to `false` → **Squeeze Off (Breakout Detected)**
3. `IsSqueezeOn = currentSqueezeState` updated after each state change

---

## Input Parameters

| Parameter       | Type            | Default     | Description                                                      |
|-----------------|-----------------|-------------|------------------------------------------------------------------|
| `TradeSymbol`   | string          | `EURUSD`    | Symbol for analysis                                              |
| `Timeframe`     | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for analysis                                           |
| `BBPeriod`      | int             | `20`        | Bollinger Bands period                                           |
| `BBDeviation`   | double          | `2.0`       | Bollinger Bands standard deviation multiplier                    |
| `KCPeriod`      | int             | `20`        | Keltner Channel EMA period                                       |
| `KCMultiplier`  | double          | `1.5`       | Keltner Channel ATR multiplier for band width                    |
| `EnableAlerts`  | bool            | `true`      | Fire an on-screen/sound alert                                    |
| `EnableEmail`   | bool            | `false`     | Send an email notification                                       |
| `EnablePush`    | bool            | `false`     | Send a mobile push notification                                  |

---

## Alert Message Format

```
Squeeze Off (Breakout Detected) detected on EURUSD (Timeframe: PERIOD_H1)
```

---

## Installation

1. Copy `TTM_Squeeze_Indicator_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
