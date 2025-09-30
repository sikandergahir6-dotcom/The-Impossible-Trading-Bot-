# The-Impossible-Trading-Bot-
On dark rainy night filled with monster and pizza I was scrolling through trading forums on Reddit. Everyone seems to have the same issues; repainting too strict, too soft etc etc. 
So I can up with the idea of combining all reasonable trading confluences into one strategy/indicator.
However, of course i can't just give it to you for free 😂 
So here is the trick ive included the script but the filters are extremely strict and not the in the right order.

I've included below everything the bit includes incase your curious 
Clean Heikin Ashi calc
✅ Session (London/NY) filter
✅ Volume spike filter
✅ ATR filter (dynamic)
✅ EMA 1H + 4H HTF confluence
✅ Candle body strength check
✅ Doji/wick cluster rejection
✅ 2-candle trend confirmation
✅ Trend flip entry-only logic
✅ Min bar gap between entries
✅ Breakout structure (HH/LL)
✅ Buy/Sell labels on chart
✅ Candle coloring by HA trend
✅ Strategy logic + position
✅ Dynamic Stop Loss (based on ATR or previous wick)
✅ Optional Take Profit (R-multiplier or trend flip)
✅ Dynamic risk-based position sizing (20%/10%/5%/3% scaling)
✅ Break-even logic
✅ Clean exits when trend flips or opposite signal appears
✅ Safe capital preservation logic
✅ Zero syntax/runtime errors

✅ATR-based Stop Loss (customizable)
✅Wick-based Stop Loss (toggle)
✅Take Profit (via R-multiple)
✅Break-even Logic
✅Dynamic risk % per phase
✅Full capital protection scaling
✅Clean exit when target hit
✅Entry/SL/TP plotted as debug lines

✅ Win rate tracking (%)
✅Trade counters: wins, losses, total
✅ Dynamic R-multiple display per trade
✅ Equity gain/loss tracker
✅ Visual “Confidence Zones” (Volume, Session, EMA trend)
✅ Debug table on chart for instant performance feedback
✅ Clean overlay visuals for analysis

✅ Heikin Ashi candles (your original version untouched)
✅ Full candle color changes (green = bullish, red = bearish)
✅ Entry only at trend flips (no mid-trend re-entries)
✅ Buy/Sell labels displayed when entry is triggered

✅ Trend confirmation (2+ same color candles → green or red)
✅ Strong candle body filter (body ≥ 50% of candle range)
✅ Dynamic ATR filter (entry only if ATR > user-defined min)

✅ 1H + 4H EMA 200 confluence
✅ NY + London session filter
✅ Volume spike filter (volume > 1.5× average)

✅ Breakout structure check (price must break local high/low within N candles)

✅ Time-in-trend filter (minimum bars between trades to avoid chop)
✅ No re-entry immediately after signal (3-bar spacing)

✅ Buy/Sell entry arrows
✅ Candle color changes
✅ Bar colors for live signals


NOTE: Designed and implemented algorithmic trading strategies in PineScript (Nasdaq index, Gold futures), including signal generation, backtesting, and performance evaluation. Currently extending models into Python for more advanced testing and automation

so this script is more of avisual inicator for before the finale on python

// This Pine Script® code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © sikandergahir6



//@version=6
strategy("UT Bot Gold Maxed Win Rate Strategy (Final)", overlay=true, default_qty_type=strategy.percent_of_equity, default_qty_value=20)

// === DEBUGGING TOGGLES (Insert at Top for Filtering) === //
enableStrongBody    = input.bool(true,  "Filter: Strong Candle Body")
enableATRFilter     = input.bool(false,  "Filter: High ATR")
enableVolumeSpike   = input.bool(false,  "Filter: Volume Spike")
enableBreakout      = input.bool(false,  "Filter: Breakout Structure")
enableNoDoji        = input.bool(false,  "Filter: No Doji")
enableHTFEMAFilter  = input.bool(false,  "Filter: HTF EMA Confluence")
enableSession       = input.bool(false,  "Filter: NY/London Session")
enableMinBarsTrend  = input.bool(false,  "Filter: Time-in-Trend Minimum")

// === INPUTS === //
emaLen            = input.int(200, title="EMA Length (HTF)")
volMultiplier     = input.float(1.5, title="Volume Spike Multiplier")
atrPeriod         = input.int(14, title="ATR Period")
atrMult           = input.float(1.0, title="ATR Multiplier")
minBodySize       = input.float(0.5, title="Min Body % (No Doji)")
breakoutLookback  = input.int(10, title="Breakout Lookback Bars")
minBarsBetween    = input.int(3, title="Min Bars Between Entries")
enableLondonNY    = input.bool(true, title="Only Trade NY/London Sessions")
useEMAFilter      = input.bool(true, title="Use 1H/4H EMA Confluence")

// === HEIKIN ASHI CALCULATION === //
haClose = (open + high + low + close)/4
var float haOpen = na
haOpen := na(haOpen[1]) ? (open + close)/2 : (haOpen[1] + haClose[1])/2
haHigh = math.max(high, math.max(haOpen, haClose))
haLow  = math.min(low, math.min(haOpen, haClose))

// === HTF EMA FILTER (using haClose) === //
ema1H = request.security(syminfo.tickerid, "60", ta.ema(haClose, emaLen))
ema4H = request.security(syminfo.tickerid, "240", ta.ema(haClose, emaLen))
htfBull = haClose > ema1H and haClose > ema4H
htfBear = haClose < ema1H and haClose < ema4H

// === SESSION FILTER === //
currentHour = hour(time, "UTC")
isLondon    = (currentHour >= 7 and currentHour < 10)
isNewYork   = (currentHour >= 13 and currentHour < 17)
inSession   = enableLondonNY ? (isLondon or isNewYork) : true

// === VOLUME & ATR FILTER === //
volMA     = ta.sma(volume, 20)
volSpike  = volume > (volMA * volMultiplier)
atr       = ta.atr(atrPeriod)
isATRBig  = atr > ta.sma(atr, 20)

// === CANDLE TREND DETECTION === //
haBull = haClose > haOpen
haBear = haClose < haOpen
haCandleDir = haBull ? 1 : haBear ? -1 : 0

// === BODY STRENGTH & NO-DOJI FILTER === //
bodySize = math.abs(haClose - haOpen)
fullRange = haHigh - haLow
isStrongBody = bodySize >= (fullRange * minBodySize)
isDoji = bodySize <= (fullRange * 0.2)


//Part 2\\


// === TREND CONFIRMATION === //
trendUp   = haCandleDir == 1 and haCandleDir[1] == 1
trendDown = haCandleDir == -1 and haCandleDir[1] == -1

// === BREAKOUT STRUCTURE FILTER === //
breakoutHigh = ta.highest(haHigh, breakoutLookback)
breakoutLow  = ta.lowest(haLow, breakoutLookback)
bullBreakout = haClose > breakoutHigh
bearBreakout = haClose < breakoutLow

// === TIME-IN-TREND FILTER === //
barsSinceFlip = ta.barssince(haCandleDir != haCandleDir[1])
enoughBarsInTrend = barsSinceFlip >= minBarsBetween

// === FINAL LONG & SHORT CONDITIONS === //
longCondition = trendUp and isStrongBody and not isDoji and isATRBig and volSpike and enoughBarsInTrend and bullBreakout
longCondition := useEMAFilter ? (longCondition and htfBull) : longCondition
longCondition := enableLondonNY ? (longCondition and inSession) : longCondition

shortCondition = trendDown and isStrongBody and not isDoji and isATRBig and volSpike and enoughBarsInTrend and bearBreakout
shortCondition := useEMAFilter ? (shortCondition and htfBear) : shortCondition
shortCondition := enableLondonNY ? (shortCondition and inSession) : shortCondition

// === EXECUTION === //
if (longCondition)
    strategy.entry("BUY", strategy.long)

if (shortCondition)
    strategy.entry("SELL", strategy.short)

// === CHART LABELS === //
plotshape(longCondition, title="Buy Signal", location=location.belowbar, color=color.green, style=shape.labelup, text="BUY")
plotshape(shortCondition, title="Sell Signal", location=location.abovebar, color=color.red, style=shape.labeldown, text="SELL")


//Part 3\\


// === DYNAMIC RISK MODEL === //
capital = strategy.equity

// Dynamic % risk per trade by capital phase
riskPercent = capital <= 250     ? 20 :
              capital <= 2500    ? 10 :
              capital <= 25000   ? 5 :
              capital <= 1000000 ? 3 : 2

// You can later use this to size your position dynamically if needed:
// positionSize = (capital * (riskPercent / 100)) / atr

// === ALERT CONDITIONS === //
alertcondition(longCondition, title="Buy Alert", message="BUY Signal - UT Bot Gold")
alertcondition(shortCondition, title="Sell Alert", message="SELL Signal - UT Bot Gold")

// === ALERT VISUALS === //
plotshape(longCondition, title="Buy Triangle", location=location.belowbar, color=color.green, style=shape.triangleup, size=size.small)
plotshape(shortCondition, title="Sell Triangle", location=location.abovebar, color=color.red, style=shape.triangledown, size=size.small)

// === WEBHOOK JSON PAYLOADS === //
longMessage  = '{"action":"buy", "symbol":"{{ticker}}", "price":"{{close}}", "time":"{{time}}"}'
shortMessage = '{"action":"sell", "symbol":"{{ticker}}", "price":"{{close}}", "time":"{{time}}"}'

// === ALERT INSTRUCTIONS === //
// Use {{longMessage}} for Buy webhook
// Use {{shortMessage}} for Sell webhook




