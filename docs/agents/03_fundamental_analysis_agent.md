# Fundamental Analysis Agent Specification

## Overview

The Fundamental Analysis Agent specializes in economic data, central bank policies, and macroeconomic trends to assess long-term currency strength and identify fundamental trading opportunities.

## Role & Responsibilities

### Primary Role
Analyze economic indicators, interest rates, and geopolitical factors to determine currency pair bias and long-term outlook.

### Key Responsibilities
1. **Monitor Economic Indicators**: GDP, inflation, employment, trade balance
2. **Analyze Interest Rate Differentials**: Compare central bank policies
3. **Track Economic Calendar**: Identify high-impact events
4. **Assess Geopolitical Factors**: Political stability, policy changes
5. **Provide Long-Term Outlook**: Medium to long-term currency bias

## Agent Configuration

### AWS Bedrock Agent Setup

```yaml
AgentName: FundamentalAnalysisAgent
AgentType: Collaborator
Model: deepseek-chat
Description: Economic and fundamental analysis specialist

ActionGroups:
  - EconomicDataActions
  - InterestRateActions
  - CalendarActions
```

## System Prompt ("Monk Mode" Style)

```
You are a Fundamental Analysis Expert for Forex trading. Analyze economic data and central bank policies to determine currency strength.

ANALYSIS FRAMEWORK:
- Focus on interest rate differentials (primary driver)
- Consider upcoming high-impact events (within 48 hours)
- Assess economic divergence between currency pairs
- Provide medium to long-term outlook (days to weeks)

STRICT GUARDRAILS:
- Only signal when confidence >0.6
- Flag high-impact events within 24 hours (avoid trading)
- Never trade during major central bank announcements
- Require clear economic divergence for strong bias
- Consider both currencies in the pair

KEY INDICATORS (Priority Order):
1. Interest Rates & Differentials (highest weight)
2. Inflation (CPI, PPI)
3. Employment (NFP, unemployment rate)
4. GDP Growth
5. Trade Balance
6. Central Bank Rhetoric (hawkish/dovish)

BIAS CRITERIA:
BULLISH (Base Currency):
- Higher interest rate or rising rate expectations
- Stronger economic growth vs quote currency
- Lower inflation (controlled)
- Improving employment
- Hawkish central bank stance

BEARISH (Base Currency):
- Lower interest rate or falling rate expectations
- Weaker economic growth vs quote currency
- Higher inflation (uncontrolled)
- Deteriorating employment
- Dovish central bank stance

OUTPUT FORMAT:
Pair: [currency pair]
Bias: [BULLISH/BEARISH/NEUTRAL]
Confidence: [0-1]
Timeframe: [SHORT/MEDIUM/LONG term outlook]
Interest Rate Differential: [base - quote]
Base Currency Outlook: [POSITIVE/NEGATIVE/NEUTRAL]
Quote Currency Outlook: [POSITIVE/NEGATIVE/NEUTRAL]
Upcoming Events: [list of high-impact events within 48h]
Key Factors: [list of main drivers]
Reasoning: [2-3 sentences explaining the bias]
```

## Input Requirements

### Economic Data Needed

```typescript
interface EconomicDataInput {
  baseCurrency: string;
  quoteCurrency: string;
  dataPoints: {
    interestRates: boolean;
    gdp: boolean;
    inflation: boolean;
    employment: boolean;
    tradeBalance: boolean;
  };
  calendarLookAhead: number; // hours
}
```

### MCP Tools Required

1. **get_economic_calendar(start, end, countries)** - Upcoming events
2. **get_interest_rates(countries)** - Current central bank rates
3. **get_economic_indicator(country, indicator, period)** - Historical data
4. **get_rate_differential(base, quote)** - Interest rate spread

## Output Schema

```typescript
interface FundamentalAnalysis {
  pair: string;
  timestamp: Date;
  bias: "BULLISH" | "BEARISH" | "NEUTRAL";
  confidence: number; // 0-1
  timeframe: "SHORT" | "MEDIUM" | "LONG"; // Days, weeks, months
  
  baseCurrency: {
    country: string;
    currency: string;
    interestRate: number;
    gdpGrowth: number;
    inflation: number;
    unemployment: number;
    outlook: "POSITIVE" | "NEGATIVE" | "NEUTRAL";
    centralBankStance: "HAWKISH" | "DOVISH" | "NEUTRAL";
  };
  
  quoteCurrency: {
    country: string;
    currency: string;
    interestRate: number;
    gdpGrowth: number;
    inflation: number;
    unemployment: number;
    outlook: "POSITIVE" | "NEGATIVE" | "NEUTRAL";
    centralBankStance: "HAWKISH" | "DOVISH" | "NEUTRAL";
  };
  
  interestRateDifferential: number; // base - quote
  
  upcomingEvents: Array<{
    date: Date;
    country: string;
    event: string;
    importance: "LOW" | "MEDIUM" | "HIGH";
    forecast: string;
    previous: string;
  }>;
  
  keyFactors: string[]; // Main drivers of the bias
  reasoning: string;
  warnings: string[]; // e.g., "High-impact event in 12 hours"
}
```

## Analysis Logic

### Step 1: Gather Economic Data

```python
def gather_economic_data(base_currency, quote_currency):
    """Collect economic data for both currencies"""
    base_data = {
        "interest_rate": get_interest_rate(base_currency),
        "gdp": get_economic_indicator(base_currency, "GDP", "latest"),
        "inflation": get_economic_indicator(base_currency, "CPI", "latest"),
        "employment": get_economic_indicator(base_currency, "UNEMPLOYMENT", "latest")
    }
    
    quote_data = {
        "interest_rate": get_interest_rate(quote_currency),
        "gdp": get_economic_indicator(quote_currency, "GDP", "latest"),
        "inflation": get_economic_indicator(quote_currency, "CPI", "latest"),
        "employment": get_economic_indicator(quote_currency, "UNEMPLOYMENT", "latest")
    }
    
    return base_data, quote_data
```

### Step 2: Calculate Interest Rate Differential

```python
def calculate_rate_differential(base_data, quote_data):
    """Calculate interest rate spread"""
    differential = base_data["interest_rate"] - quote_data["interest_rate"]
    
    # Assess significance
    if abs(differential) > 2.0:
        significance = "HIGH"
    elif abs(differential) > 0.5:
        significance = "MEDIUM"
    else:
        significance = "LOW"
    
    return differential, significance
```

### Step 3: Assess Economic Divergence

```python
def assess_divergence(base_data, quote_data):
    """Compare economic strength between currencies"""
    score = 0
    factors = []
    
    # Interest rate comparison (highest weight)
    if base_data["interest_rate"] > quote_data["interest_rate"]:
        score += 3
        factors.append("Higher interest rate")
    elif base_data["interest_rate"] < quote_data["interest_rate"]:
        score -= 3
        factors.append("Lower interest rate")
    
    # GDP growth comparison
    if base_data["gdp"] > quote_data["gdp"]:
        score += 2
        factors.append("Stronger GDP growth")
    elif base_data["gdp"] < quote_data["gdp"]:
        score -= 2
        factors.append("Weaker GDP growth")
    
    # Inflation comparison (controlled inflation is good)
    base_inflation_ok = 1.5 < base_data["inflation"] < 3.0
    quote_inflation_ok = 1.5 < quote_data["inflation"] < 3.0
    
    if base_inflation_ok and not quote_inflation_ok:
        score += 1
        factors.append("Better inflation control")
    elif not base_inflation_ok and quote_inflation_ok:
        score -= 1
        factors.append("Worse inflation control")
    
    # Employment comparison (lower unemployment is better)
    if base_data["employment"] < quote_data["employment"]:
        score += 1
        factors.append("Lower unemployment")
    elif base_data["employment"] > quote_data["employment"]:
        score -= 1
        factors.append("Higher unemployment")
    
    return score, factors
```

### Step 4: Check Upcoming Events

```python
def check_upcoming_events(base_currency, quote_currency, hours_ahead=48):
    """Check for high-impact events"""
    events = get_economic_calendar(
        start=now(),
        end=now() + timedelta(hours=hours_ahead),
        countries=[base_currency, quote_currency]
    )
    
    high_impact = [e for e in events if e["importance"] == "HIGH"]
    
    warnings = []
    if any(e for e in high_impact if (e["date"] - now()).hours < 24):
        warnings.append("High-impact event within 24 hours - avoid trading")
    
    return events, warnings
```

### Step 5: Determine Bias and Confidence

```python
def determine_bias(divergence_score, rate_differential, events):
    """Determine overall bias and confidence"""
    
    # Base bias on divergence score
    if divergence_score >= 4:
        bias = "BULLISH"
        confidence = 0.75
    elif divergence_score <= -4:
        bias = "BEARISH"
        confidence = 0.75
    elif divergence_score >= 2:
        bias = "BULLISH"
        confidence = 0.60
    elif divergence_score <= -2:
        bias = "BEARISH"
        confidence = 0.60
    else:
        bias = "NEUTRAL"
        confidence = 0.40
    
    # Adjust confidence based on rate differential
    if abs(rate_differential) > 2.0:
        confidence += 0.10
    
    # Reduce confidence if high-impact events imminent
    high_impact_soon = any(
        e for e in events 
        if e["importance"] == "HIGH" and (e["date"] - now()).hours < 24
    )
    if high_impact_soon:
        confidence *= 0.7
    
    return bias, min(1.0, confidence)
```

## Guardrails Implementation

### Pre-Signal Checks

```python
def validate_fundamental_signal(analysis):
    """Validate signal meets all guardrails"""
    
    # Confidence threshold
    if analysis.confidence < 0.6:
        return False, "Confidence below 0.6 threshold"
    
    # High-impact event check
    imminent_events = [
        e for e in analysis.upcomingEvents
        if e.importance == "HIGH" and (e.date - now()).hours < 24
    ]
    if imminent_events:
        return False, f"High-impact event in {imminent_events[0].event} within 24h"
    
    # Central bank announcement check
    cb_events = [
        e for e in analysis.upcomingEvents
        if "central bank" in e.event.lower() or "interest rate" in e.event.lower()
    ]
    if cb_events and (cb_events[0].date - now()).hours < 2:
        return False, "Central bank announcement imminent"
    
    # Economic divergence check
    if analysis.bias != "NEUTRAL":
        if abs(analysis.interestRateDifferential) < 0.25:
            return False, "Insufficient interest rate differential"
    
    return True, "All checks passed"
```

## Example Scenarios

### Scenario 1: Strong Bullish Bias (EUR/USD)

```
EUR/USD Analysis

Base Currency (EUR):
- Interest Rate: 4.00%
- GDP Growth: 2.5%
- Inflation: 2.2% (controlled)
- Unemployment: 6.5%
- Central Bank: Hawkish (considering further hikes)

Quote Currency (USD):
- Interest Rate: 5.25%
- GDP Growth: 2.0%
- Inflation: 3.5% (elevated)
- Unemployment: 3.8%
- Central Bank: Neutral (pause mode)

Interest Rate Differential: -1.25% (USD favor)

Upcoming Events (48h):
- None high-impact

Analysis:
Bias: BEARISH (USD stronger on rates)
Confidence: 0.70
Timeframe: MEDIUM (weeks)
Key Factors:
- USD has 1.25% higher interest rate
- EUR inflation better controlled
- USD central bank in pause mode
Reasoning: Despite EUR's better inflation control, USD's significantly higher 
interest rate and stronger employment outweigh. Medium-term bearish bias on EUR/USD.
```

### Scenario 2: Neutral (GBP/JPY)

```
GBP/JPY Analysis

Base Currency (GBP):
- Interest Rate: 5.25%
- GDP Growth: 0.5% (weak)
- Inflation: 4.5% (high)
- Central Bank: Dovish (considering cuts)

Quote Currency (JPY):
- Interest Rate: -0.10%
- GDP Growth: 1.2%
- Inflation: 2.8%
- Central Bank: Hawkish (ending negative rates)

Interest Rate Differential: +5.35% (GBP favor)

Upcoming Events (48h):
- Bank of Japan meeting in 18 hours (HIGH impact)

Analysis:
Bias: NEUTRAL
Confidence: 0.45
Warnings: ["High-impact BoJ meeting in 18 hours - avoid trading"]
Reasoning: While GBP has massive rate advantage, weak growth and high inflation 
concerning. BoJ meeting imminent could shift dynamics. Wait for clarity.
```

### Scenario 3: Strong Bearish Bias (AUD/USD)

```
AUD/USD Analysis

Base Currency (AUD):
- Interest Rate: 4.35%
- GDP Growth: 1.8%
- Inflation: 5.2% (high)
- Unemployment: 3.7%
- Central Bank: Dovish (considering cuts due to weak growth)

Quote Currency (USD):
- Interest Rate: 5.25%
- GDP Growth: 2.8%
- Inflation: 3.2%
- Unemployment: 3.8%
- Central Bank: Hawkish (data-dependent, bias to hold)

Interest Rate Differential: -0.90% (USD favor)

Upcoming Events (48h):
- Australian GDP tomorrow (MEDIUM impact)

Analysis:
Bias: BEARISH
Confidence: 0.75
Timeframe: MEDIUM
Key Factors:
- USD higher interest rate
- USD stronger GDP growth
- AUD high inflation with weak growth (stagflation risk)
- RBA dovish, Fed hawkish
Reasoning: Clear economic divergence favoring USD. AUD facing stagflation risks 
with high inflation and weak growth. RBA likely to cut while Fed holds. Strong 
bearish bias on AUD/USD.
```

## Performance Metrics

### Forecast Accuracy
- Bias direction accuracy (% correct)
- Confidence calibration
- Timeframe accuracy (short vs medium vs long)

### Event Impact
- High-impact event prediction accuracy
- Event-driven volatility anticipation
- Post-event bias adjustment

### Economic Indicator Weighting
- Which indicators most predictive
- Optimal indicator combinations
- Regional differences (EUR vs USD vs JPY)

## Integration with MCP Servers

### Economic Data Server
```python
# Get interest rates
rates = mcp.call("get_interest_rates", {
    "countries": ["US", "EU", "GB", "JP"]
})

# Get economic calendar
calendar = mcp.call("get_economic_calendar", {
    "start_date": "2025-11-23",
    "end_date": "2025-11-25",
    "countries": ["US", "EU"],
    "importance": "HIGH"
})

# Get specific indicator
gdp = mcp.call("get_economic_indicator", {
    "country": "US",
    "indicator": "GDP",
    "period": "Q3-2025"
})

# Calculate rate differential
diff = mcp.call("get_rate_differential", {
    "base_currency": "EUR",
    "quote_currency": "USD"
})
```

## Testing Scenarios

1. **Clear rate differential (>2%)** → Should signal with high confidence
2. **High-impact event within 24h** → Should flag warning, reduce confidence
3. **Central bank meeting imminent** → Should avoid signaling
4. **Mixed economic data** → Should return NEUTRAL
5. **Strong divergence + upcoming event** → Should signal but flag event risk
6. **Stagflation scenario** → Should recognize and bias accordingly

---

**Version**: 1.0  
**Last Updated**: November 23, 2025  
**Status**: Specification - Ready for Implementation
