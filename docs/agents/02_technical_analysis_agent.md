# Technical Analysis Agent Specification

## Overview

The Technical Analysis Agent specializes in chart patterns, technical indicators, and price action analysis to identify trading opportunities and optimal entry/exit points.

## Role & Responsibilities

### Primary Role
Analyze price charts, technical indicators, and patterns to generate trading signals with specific entry, stop-loss, and take-profit levels.

### Key Responsibilities
1. **Analyze Price Action**: Identify trends, support/resistance, and chart patterns
2. **Calculate Indicators**: RSI, MACD, Moving Averages, Bollinger Bands, etc.
3. **Generate Signals**: Provide specific entry/exit recommendations
4. **Assess Trend Strength**: Determine conviction level of technical setup
5. **Identify Key Levels**: Support, resistance, and invalidation points

## Agent Configuration

### AWS Bedrock Agent Setup

```yaml
AgentName: TechnicalAnalysisAgent
AgentType: Collaborator
Model: deepseek-chat
Description: Technical analysis specialist for chart patterns and indicators

ActionGroups:
  - MarketDataActions
  - TechnicalIndicatorActions
  - PatternRecognitionActions
```

## System Prompt ("Monk Mode" Style)

```
You are a Technical Analysis Expert for Forex trading. Analyze charts and indicators to generate precise trading signals.

ANALYSIS FRAMEWORK:
- Use multiple timeframes (H1, H4, D1) for confluence
- Require ≥3 indicators aligning before signaling
- Provide specific price levels (entry, stop-loss, take-profit)
- Assign confidence based on indicator confluence

STRICT GUARDRAILS:
- Only signal when confidence >0.7
- Only signal when ≥3 indicators align
- Stop-loss must be at logical level (support/resistance)
- Risk/reward ratio must be ≥2:1
- Never signal during low liquidity periods (Asian session open)

INDICATORS TO USE:
- Trend: SMA20, SMA50, EMA200
- Momentum: RSI(14), MACD(12,26,9)
- Volatility: Bollinger Bands(20,2)
- Support/Resistance: Pivot Points, Fibonacci
- Volume: If available

SIGNAL CRITERIA:
BULLISH:
- Price above SMA20 and SMA50
- RSI between 40-60 (not overbought)
- MACD histogram positive and increasing
- Price near support level
- Confluence of ≥3 indicators

BEARISH:
- Price below SMA20 and SMA50
- RSI between 40-60 (not oversold)
- MACD histogram negative and decreasing
- Price near resistance level
- Confluence of ≥3 indicators

OUTPUT FORMAT:
Pair: [currency pair]
Trend: [BULLISH/BEARISH/NEUTRAL]
Strength: [0-1, how strong is the trend]
Confidence: [0-1, based on indicator confluence]
Entry: [specific price level or null]
Stop Loss: [specific price level or null]
Take Profit: [specific price level or null]
Risk/Reward: [ratio]
Indicators: {
  RSI: [value],
  MACD: {value, signal, histogram},
  SMA20: [value],
  SMA50: [value],
  EMA200: [value],
  BB: {upper, middle, lower}
}
Patterns: [list of identified patterns]
Support: [list of support levels]
Resistance: [list of resistance levels]
Reasoning: [2-3 sentences explaining the setup]
```

## Input Requirements

### Market Data Needed

```typescript
interface MarketDataInput {
  pair: string;
  timeframes: ["M15", "H1", "H4", "D1"];
  candleCount: 200; // For indicator calculations
  includeVolume: boolean;
}
```

### MCP Tools Required

1. **get_live_price(pair, timeframe)** - Current price
2. **get_historical_data(pair, start, end, timeframe)** - OHLCV data
3. **calculate_indicator(pair, indicator, params)** - Technical indicators
4. **detect_patterns(pair, timeframe)** - Chart patterns
5. **find_support_resistance(pair, timeframe)** - Key levels

## Output Schema

```typescript
interface TechnicalAnalysis {
  pair: string;
  timeframe: string;
  timestamp: Date;
  trend: "BULLISH" | "BEARISH" | "NEUTRAL";
  strength: number; // 0-1
  confidence: number; // 0-1
  signals: {
    entry: number | null;
    stopLoss: number | null;
    takeProfit: number | null;
    riskRewardRatio: number | null;
  };
  indicators: {
    rsi: number;
    macd: {
      value: number;
      signal: number;
      histogram: number;
    };
    movingAverages: {
      sma20: number;
      sma50: number;
      ema200: number;
    };
    bollingerBands: {
      upper: number;
      middle: number;
      lower: number;
    };
  };
  patterns: Array<{
    name: string;
    type: "BULLISH" | "BEARISH";
    confidence: number;
  }>;
  supportResistance: {
    support: number[];
    resistance: number[];
  };
  reasoning: string;
  invalidationLevel: number; // Price level that invalidates the setup
}
```

## Analysis Logic

### Step 1: Gather Data

```python
def gather_market_data(pair):
    """Collect price data across multiple timeframes"""
    data = {
        "H1": get_historical_data(pair, "H1", 200),
        "H4": get_historical_data(pair, "H4", 200),
        "D1": get_historical_data(pair, "D1", 200)
    }
    return data
```

### Step 2: Calculate Indicators

```python
def calculate_all_indicators(data):
    """Calculate all technical indicators"""
    indicators = {
        "rsi": calculate_indicator(data, "RSI", {"period": 14}),
        "macd": calculate_indicator(data, "MACD", {"fast": 12, "slow": 26, "signal": 9}),
        "sma20": calculate_indicator(data, "SMA", {"period": 20}),
        "sma50": calculate_indicator(data, "SMA", {"period": 50}),
        "ema200": calculate_indicator(data, "EMA", {"period": 200}),
        "bb": calculate_indicator(data, "BB", {"period": 20, "std": 2})
    }
    return indicators
```

### Step 3: Identify Trend

```python
def identify_trend(price, indicators):
    """Determine trend direction and strength"""
    bullish_signals = 0
    bearish_signals = 0
    
    # Moving average alignment
    if price > indicators["sma20"] > indicators["sma50"]:
        bullish_signals += 1
    elif price < indicators["sma20"] < indicators["sma50"]:
        bearish_signals += 1
    
    # MACD
    if indicators["macd"]["histogram"] > 0:
        bullish_signals += 1
    elif indicators["macd"]["histogram"] < 0:
        bearish_signals += 1
    
    # RSI (not extreme)
    if 40 < indicators["rsi"] < 60:
        # Neutral momentum, good for entry
        pass
    elif indicators["rsi"] > 70:
        bearish_signals += 1  # Overbought
    elif indicators["rsi"] < 30:
        bullish_signals += 1  # Oversold
    
    if bullish_signals >= 2:
        return "BULLISH", bullish_signals / 3
    elif bearish_signals >= 2:
        return "BEARISH", bearish_signals / 3
    else:
        return "NEUTRAL", 0.0
```

### Step 4: Calculate Confidence

```python
def calculate_confidence(trend, indicators, patterns, support_resistance):
    """Calculate confidence based on confluence"""
    confidence = 0.0
    
    # Indicator alignment (max 0.4)
    aligned_indicators = count_aligned_indicators(trend, indicators)
    confidence += min(0.4, aligned_indicators * 0.1)
    
    # Pattern confirmation (max 0.3)
    if patterns:
        pattern_confidence = max([p["confidence"] for p in patterns])
        confidence += pattern_confidence * 0.3
    
    # Support/Resistance proximity (max 0.3)
    if is_near_key_level(price, support_resistance):
        confidence += 0.3
    
    return min(1.0, confidence)
```

### Step 5: Determine Entry/Exit Levels

```python
def calculate_entry_exit(trend, price, support_resistance, indicators):
    """Calculate specific entry, stop-loss, and take-profit levels"""
    if trend == "BULLISH":
        entry = price  # Current price or pullback to support
        stop_loss = find_nearest_support(price, support_resistance)
        risk_pips = abs(entry - stop_loss)
        take_profit = entry + (risk_pips * 2)  # 2:1 R/R minimum
    elif trend == "BEARISH":
        entry = price
        stop_loss = find_nearest_resistance(price, support_resistance)
        risk_pips = abs(entry - stop_loss)
        take_profit = entry - (risk_pips * 2)
    else:
        return None, None, None
    
    return entry, stop_loss, take_profit
```

## Guardrails Implementation

### Pre-Signal Checks

```python
def validate_signal(analysis):
    """Validate signal meets all guardrails"""
    checks = []
    
    # Confidence threshold
    if analysis.confidence < 0.7:
        return False, "Confidence below 0.7 threshold"
    
    # Indicator alignment
    aligned = count_aligned_indicators(analysis.trend, analysis.indicators)
    if aligned < 3:
        return False, f"Only {aligned} indicators aligned, need ≥3"
    
    # Risk/Reward ratio
    if analysis.signals.riskRewardRatio < 2.0:
        return False, f"R/R ratio {analysis.signals.riskRewardRatio} < 2.0"
    
    # Liquidity check
    if is_low_liquidity_period():
        return False, "Low liquidity period (Asian session)"
    
    return True, "All checks passed"
```

## Example Scenarios

### Scenario 1: Strong Bullish Setup

```
EUR/USD @ 1.0850

Indicators:
- RSI: 52 (neutral, room to move up)
- MACD: Histogram positive and increasing
- Price above SMA20 (1.0840) and SMA50 (1.0820)
- Near support at 1.0845
- Bollinger Bands: Price at middle band, room to upper band

Patterns:
- Bullish flag pattern (0.75 confidence)
- Higher lows forming

Analysis:
Trend: BULLISH
Strength: 0.80
Confidence: 0.85 (4 indicators aligned + pattern + support)
Entry: 1.0850
Stop Loss: 1.0820 (below SMA50 and support)
Take Profit: 1.0910 (2:1 R/R)
Risk/Reward: 2.0
Reasoning: Strong bullish alignment with price above key MAs, MACD positive, 
bullish flag pattern, and price at support. RSI neutral allows room for upside.
```

### Scenario 2: Weak Setup (No Signal)

```
GBP/USD @ 1.2650

Indicators:
- RSI: 48 (neutral)
- MACD: Histogram near zero, indecisive
- Price between SMA20 (1.2660) and SMA50 (1.2640)
- No clear support/resistance nearby

Patterns:
- None identified

Analysis:
Trend: NEUTRAL
Strength: 0.30
Confidence: 0.45 (only 1 indicator weakly aligned)
Entry: null
Stop Loss: null
Take Profit: null
Reasoning: Insufficient confluence. Price choppy between MAs, MACD indecisive, 
no clear pattern. Wait for clearer setup.
```

### Scenario 3: Overbought (No Signal)

```
USD/JPY @ 150.50

Indicators:
- RSI: 78 (overbought)
- MACD: Histogram positive but declining
- Price above all MAs
- At resistance level 150.50

Analysis:
Trend: BULLISH (but extended)
Strength: 0.70
Confidence: 0.55 (indicators aligned but overbought)
Entry: null
Stop Loss: null
Take Profit: null
Reasoning: While trend is bullish, RSI overbought at 78 and price at resistance. 
Risk of pullback too high. Wait for retracement to support.
```

## Performance Metrics

### Signal Quality
- Win rate of signals generated
- Average R/R ratio achieved
- Confidence calibration (does 0.8 confidence = 80% win rate?)

### Indicator Effectiveness
- Which indicators most predictive
- Optimal indicator combinations
- False signal rate per indicator

### Pattern Recognition
- Pattern success rate
- Most reliable patterns
- Pattern + indicator confluence performance

## Integration with MCP Servers

### Market Data Server
```python
# Get current price
price = mcp.call("get_live_price", {"pair": "EUR/USD", "timeframe": "H1"})

# Get historical data
history = mcp.call("get_historical_data", {
    "pair": "EUR/USD",
    "start_date": "2025-11-01",
    "end_date": "2025-11-23",
    "timeframe": "H1"
})
```

### Technical Analysis Server
```python
# Calculate RSI
rsi = mcp.call("calculate_indicator", {
    "pair": "EUR/USD",
    "indicator": "RSI",
    "params": {"period": 14}
})

# Detect patterns
patterns = mcp.call("detect_patterns", {
    "pair": "EUR/USD",
    "timeframe": "H4"
})

# Find support/resistance
levels = mcp.call("find_support_resistance", {
    "pair": "EUR/USD",
    "timeframe": "D1"
})
```

## Testing Scenarios

1. **Strong trend with confluence** → Should signal with high confidence
2. **Choppy market** → Should return NEUTRAL with low confidence
3. **Overbought/oversold extremes** → Should wait for retracement
4. **Only 2 indicators aligned** → Should not signal (need ≥3)
5. **Good setup but R/R < 2:1** → Should not signal
6. **Low liquidity period** → Should not signal

---

**Version**: 1.0  
**Last Updated**: November 23, 2025  
**Status**: Specification - Ready for Implementation
