# Correlation & Confluence Analysis Agent Specification

## Overview

The Correlation & Confluence Agent specializes in analyzing currency pair correlations, US Dollar Index (DXY), Japanese Yen strength indicators, and cross-pair confluence to validate trading signals and identify hidden risks.

## Role & Responsibilities

### Primary Role
Monitor currency correlations, DXY movements, JPY strength, and cross-pair confluence to validate signals and identify portfolio risks.

### Key Responsibilities
1. **Monitor DXY (US Dollar Index)**: Track USD strength across all pairs
2. **Track JPY Strength**: Monitor Yen movements and safe-haven flows
3. **Analyze Pair Correlations**: Identify correlated and divergent movements
4. **Validate Confluence**: Confirm signals across multiple related pairs
5. **Detect Hidden Risks**: Identify overexposure through correlated positions

## Agent Configuration

### AWS Bedrock Agent Setup

```yaml
AgentName: CorrelationConfluenceAgent
AgentType: Collaborator
Model: deepseek-chat
Description: Currency correlation and confluence specialist

ActionGroups:
  - CorrelationAnalysisActions
  - DXYAnalysisActions
  - JPYAnalysisActions
  - ConfluenceValidationActions
```

## System Prompt ("Monk Mode" Style)

```
You are a Correlation & Confluence Expert for Forex trading. Analyze currency relationships, DXY, JPY strength, and cross-pair confluence.

ANALYSIS FRAMEWORK:
- Monitor DXY for overall USD strength/weakness
- Track JPY strength for risk-on/risk-off sentiment
- Analyze correlations between currency pairs
- Validate signals through cross-pair confluence
- Identify hidden portfolio risks

STRICT GUARDRAILS:
- Flag if DXY contradicts USD pair signals
- Flag if JPY pairs show divergence (risk sentiment shift)
- Reject trades with >0.8 correlation to existing positions
- Require confluence across ≥2 related pairs for validation
- Alert on unusual correlation breakdowns

KEY INDICATORS:
1. DXY (US Dollar Index) - USD strength vs basket
2. JPY Crosses (USD/JPY, EUR/JPY, GBP/JPY) - Risk sentiment
3. Correlation Matrix - 30-day rolling correlation
4. Currency Strength Index - Individual currency strength
5. Safe Haven Flows - CHF, JPY, USD movements

CORRELATION RULES:
- Positive correlation >0.7: Pairs move together
- Negative correlation <-0.7: Pairs move opposite
- Correlation 0.8-1.0: Highly correlated (risk of overexposure)
- Correlation breakdown: When historical correlation fails

DXY ANALYSIS:
- DXY rising → USD strengthening (bullish USD pairs)
- DXY falling → USD weakening (bearish USD pairs)
- DXY divergence from USD pairs → Warning signal
- DXY at key levels (100, 105, 110) → Potential reversal

JPY ANALYSIS:
- USD/JPY rising → Risk-on sentiment
- USD/JPY falling → Risk-off (safe haven demand)
- JPY strengthening across all pairs → Flight to safety
- JPY weakening across all pairs → Risk appetite
- Divergence in JPY crosses → Mixed risk sentiment

CONFLUENCE VALIDATION:
For EUR/USD BUY signal, check:
- Is DXY falling? (confirms USD weakness)
- Is EUR/GBP rising? (confirms EUR strength)
- Is EUR/JPY rising? (confirms EUR strength + risk-on)
- Are other USD pairs falling? (confirms USD weakness)

OUTPUT FORMAT:
Analysis Type: [DXY/JPY/CORRELATION/CONFLUENCE]
DXY: {
  value: [current level],
  trend: [RISING/FALLING/NEUTRAL],
  strength: [0-1],
  keyLevels: [support/resistance]
}
JPY Strength: {
  sentiment: [RISK_ON/RISK_OFF/NEUTRAL],
  usdJpy: [value and trend],
  crossesAlignment: [ALIGNED/DIVERGENT]
}
Correlations: {
  pair1_pair2: [correlation coefficient],
  ...
}
Confluence: {
  validated: [true/false],
  supportingPairs: [list],
  contradictingPairs: [list]
}
Warnings: [list of risk flags]
Reasoning: [2-3 sentences]
```

## Input Requirements

### Data Needed

```typescript
interface CorrelationInput {
  targetPair: string;
  proposedAction: "BUY" | "SELL";
  existingPositions: Array<{
    pair: string;
    side: "BUY" | "SELL";
    size: number;
  }>;
  timeframe: string;
  lookbackPeriod: number; // days for correlation calculation
}
```

### MCP Tools Required

1. **get_dxy_data()** - US Dollar Index current and historical
2. **get_currency_strength(currency)** - Individual currency strength index
3. **calculate_correlation(pair1, pair2, period)** - Correlation coefficient
4. **get_jpy_crosses()** - All JPY pair data
5. **get_related_pairs(pair)** - Pairs sharing currencies

## Output Schema

```typescript
interface CorrelationAnalysis {
  timestamp: Date;
  targetPair: string;
  
  dxy: {
    current: number;
    trend: "RISING" | "FALLING" | "NEUTRAL";
    change24h: number;
    keyLevels: {
      support: number[];
      resistance: number[];
    };
    implication: string; // For USD pairs
  };
  
  jpyAnalysis: {
    sentiment: "RISK_ON" | "RISK_OFF" | "NEUTRAL";
    usdJpy: {
      value: number;
      trend: "RISING" | "FALLING";
      change24h: number;
    };
    crosses: Array<{
      pair: string;
      value: number;
      trend: "RISING" | "FALLING";
    }>;
    alignment: "ALIGNED" | "DIVERGENT";
    implication: string;
  };
  
  currencyStrength: {
    base: {
      currency: string;
      strength: number; // 0-100
      rank: number; // 1-8 among major currencies
    };
    quote: {
      currency: string;
      strength: number;
      rank: number;
    };
  };
  
  correlations: {
    withExistingPositions: Array<{
      pair: string;
      correlation: number;
      risk: "HIGH" | "MEDIUM" | "LOW";
    }>;
    relatedPairs: Array<{
      pair: string;
      correlation: number;
      currentTrend: "ALIGNED" | "DIVERGENT";
    }>;
  };
  
  confluence: {
    validated: boolean;
    supportingPairs: string[];
    contradictingPairs: string[];
    confluenceScore: number; // 0-1
  };
  
  warnings: string[];
  recommendations: string[];
  reasoning: string;
}
```

## Analysis Logic

### Step 1: Analyze DXY

```python
def analyze_dxy(target_pair):
    """Analyze US Dollar Index for USD pairs"""
    dxy = get_dxy_data()
    
    # Check if target pair involves USD
    if "USD" not in target_pair:
        return None
    
    # Determine DXY trend
    if dxy.change_24h > 0.5:
        trend = "RISING"
    elif dxy.change_24h < -0.5:
        trend = "FALLING"
    else:
        trend = "NEUTRAL"
    
    # Check for divergence
    is_usd_base = target_pair.startswith("USD")
    
    if is_usd_base:
        # USD/XXX - DXY rising should support USD strength
        expected_alignment = "DXY rising supports USD/XXX bullish"
    else:
        # XXX/USD - DXY rising should support XXX/USD bearish
        expected_alignment = "DXY rising supports XXX/USD bearish"
    
    return {
        "current": dxy.value,
        "trend": trend,
        "change_24h": dxy.change_24h,
        "implication": expected_alignment
    }
```

### Step 2: Analyze JPY Strength & Risk Sentiment

```python
def analyze_jpy_sentiment():
    """Analyze JPY crosses for risk sentiment"""
    usd_jpy = get_price("USD/JPY")
    eur_jpy = get_price("EUR/JPY")
    gbp_jpy = get_price("GBP/JPY")
    aud_jpy = get_price("AUD/JPY")
    
    # USD/JPY is primary risk sentiment indicator
    if usd_jpy.change_24h > 0.5:
        sentiment = "RISK_ON"  # JPY weakening
    elif usd_jpy.change_24h < -0.5:
        sentiment = "RISK_OFF"  # JPY strengthening (safe haven)
    else:
        sentiment = "NEUTRAL"
    
    # Check alignment across JPY crosses
    jpy_crosses = [eur_jpy, gbp_jpy, aud_jpy]
    trends = [c.change_24h > 0 for c in jpy_crosses]
    
    if all(trends) or all(not t for t in trends):
        alignment = "ALIGNED"
    else:
        alignment = "DIVERGENT"
    
    return {
        "sentiment": sentiment,
        "usd_jpy": usd_jpy,
        "crosses": jpy_crosses,
        "alignment": alignment
    }
```

### Step 3: Calculate Currency Strength

```python
def calculate_currency_strength(currency):
    """Calculate individual currency strength index"""
    # Get all major pairs involving this currency
    pairs = get_pairs_with_currency(currency)
    
    strength_sum = 0
    for pair in pairs:
        price_change = get_price_change(pair, period="24h")
        
        if pair.startswith(currency):
            # Currency is base
            strength_sum += price_change
        else:
            # Currency is quote
            strength_sum -= price_change
    
    # Normalize to 0-100
    strength = normalize(strength_sum, 0, 100)
    
    return strength
```

### Step 4: Check Correlations

```python
def check_correlations(target_pair, existing_positions, period=30):
    """Check correlations with existing positions"""
    correlations = []
    
    for position in existing_positions:
        corr = calculate_correlation(
            target_pair, 
            position.pair, 
            period_days=period
        )
        
        # Assess risk
        if abs(corr) > 0.8:
            risk = "HIGH"
        elif abs(corr) > 0.6:
            risk = "MEDIUM"
        else:
            risk = "LOW"
        
        correlations.append({
            "pair": position.pair,
            "correlation": corr,
            "risk": risk
        })
    
    return correlations
```

### Step 5: Validate Confluence

```python
def validate_confluence(target_pair, proposed_action):
    """Validate signal through cross-pair confluence"""
    base, quote = target_pair.split("/")
    
    # Get related pairs
    base_pairs = get_pairs_with_currency(base)
    quote_pairs = get_pairs_with_currency(quote)
    
    supporting = []
    contradicting = []
    
    # Check if related pairs support the signal
    for pair in base_pairs:
        if pair == target_pair:
            continue
        
        trend = get_trend(pair)
        
        # If buying EUR/USD, EUR should be strong in other pairs
        if proposed_action == "BUY":
            if pair.startswith(base) and trend == "BULLISH":
                supporting.append(pair)
            elif pair.endswith(base) and trend == "BEARISH":
                supporting.append(pair)
            else:
                contradicting.append(pair)
    
    confluence_score = len(supporting) / (len(supporting) + len(contradicting))
    
    return {
        "validated": confluence_score > 0.6,
        "supporting": supporting,
        "contradicting": contradicting,
        "score": confluence_score
    }
```

## Key Currency Relationships

### Major Correlations

```
POSITIVE CORRELATIONS (move together):
- EUR/USD & GBP/USD: ~0.85 (both vs USD)
- EUR/USD & AUD/USD: ~0.75 (risk currencies vs USD)
- EUR/USD & NZD/USD: ~0.70 (risk currencies vs USD)
- AUD/USD & NZD/USD: ~0.90 (commodity currencies)
- EUR/GBP & EUR/USD: ~0.60 (EUR strength)

NEGATIVE CORRELATIONS (move opposite):
- EUR/USD & USD/CHF: ~-0.95 (inverse pairs)
- EUR/USD & USD/JPY: ~-0.60 (USD strength)
- GBP/USD & USD/JPY: ~-0.55 (USD strength)
- AUD/USD & USD/JPY: ~-0.65 (risk-on/risk-off)

SPECIAL RELATIONSHIPS:
- DXY vs all USD pairs (inverse for XXX/USD)
- JPY crosses for risk sentiment
- CHF as safe haven (like JPY)
- Commodity currencies (AUD, NZD, CAD) move with commodities
```

### DXY Key Levels

```
CRITICAL LEVELS:
- 110.00: Strong resistance, USD very strong
- 105.00: Major psychological level
- 100.00: Major support/resistance, neutral level
- 95.00: Support, USD weakness
- 90.00: Strong support, USD very weak

IMPLICATIONS:
- DXY > 105: USD strength, bearish for XXX/USD pairs
- DXY 100-105: Neutral to USD strength
- DXY 95-100: Neutral to USD weakness
- DXY < 95: USD weakness, bullish for XXX/USD pairs
```

### JPY Risk Sentiment Levels

```
USD/JPY LEVELS:
- > 150: Extreme risk-on, JPY very weak
- 145-150: Strong risk-on
- 140-145: Moderate risk-on
- 135-140: Neutral
- 130-135: Moderate risk-off
- < 130: Strong risk-off, safe haven demand

IMPLICATIONS:
- Rising USD/JPY: Risk-on → Bullish for risk currencies (AUD, NZD, EUR)
- Falling USD/JPY: Risk-off → Bearish for risk currencies, bullish safe havens
```

## Example Scenarios

### Scenario 1: EUR/USD BUY - Strong Confluence

```
Target: EUR/USD BUY @ 1.0850

DXY Analysis:
- Current: 103.50
- Trend: FALLING (-0.8% in 24h)
- Implication: USD weakness supports EUR/USD bullish

JPY Analysis:
- USD/JPY: 148.50 (rising, +0.5%)
- Sentiment: RISK_ON
- Implication: Risk appetite supports EUR strength

Currency Strength:
- EUR: 72/100 (Rank 2)
- USD: 45/100 (Rank 6)
- Clear EUR > USD

Correlations:
- No existing USD positions
- Risk: LOW

Confluence:
- EUR/GBP: Rising (EUR strength confirmed)
- EUR/JPY: Rising (EUR strength + risk-on)
- GBP/USD: Rising (USD weakness confirmed)
- AUD/USD: Rising (USD weakness confirmed)
- Supporting: 4 pairs
- Contradicting: 0 pairs
- Score: 1.0

Result: VALIDATED with HIGH confidence
Reasoning: Perfect confluence - DXY falling confirms USD weakness, JPY crosses 
show risk-on supporting EUR, all related pairs align with EUR/USD bullish signal.
```

### Scenario 2: GBP/USD BUY - Divergence Warning

```
Target: GBP/USD BUY @ 1.2650

DXY Analysis:
- Current: 105.20
- Trend: RISING (+0.6% in 24h)
- WARNING: DXY rising contradicts GBP/USD bullish signal

JPY Analysis:
- USD/JPY: 149.80 (rising)
- Sentiment: RISK_ON
- Implication: Supports risk currencies

Currency Strength:
- GBP: 68/100 (Rank 3)
- USD: 71/100 (Rank 2)
- WARNING: USD stronger than GBP

Correlations:
- Existing EUR/USD LONG position
- Correlation: 0.85
- Risk: HIGH (overexposure to USD weakness)

Confluence:
- EUR/USD: Flat (no USD weakness confirmation)
- GBP/JPY: Rising (GBP strength)
- AUD/USD: Falling (USD strength)
- Supporting: 1 pair
- Contradicting: 2 pairs
- Score: 0.33

Result: NOT VALIDATED
Warnings:
- DXY rising contradicts signal
- High correlation with existing position
- Mixed confluence (only 33%)
- USD actually stronger than GBP

Reasoning: Multiple red flags - DXY rising shows USD strength, high correlation 
with existing EUR/USD position creates overexposure, poor confluence. REJECT.
```

### Scenario 3: USD/JPY SELL - Risk-Off Detected

```
Target: USD/JPY SELL @ 149.50

DXY Analysis:
- Current: 104.80
- Trend: NEUTRAL
- Implication: Neutral for USD

JPY Analysis:
- USD/JPY: 149.50 (falling rapidly, -1.2% in 4h)
- EUR/JPY: Falling (-0.9%)
- GBP/JPY: Falling (-0.8%)
- AUD/JPY: Falling (-1.1%)
- Sentiment: RISK_OFF (strong safe haven demand)
- Alignment: ALIGNED (all JPY crosses falling)

Currency Strength:
- USD: 58/100
- JPY: 82/100 (Rank 1 - strongest)

Confluence:
- All JPY crosses falling (JPY strength)
- Risk currencies (AUD, NZD) weakening
- Safe havens (CHF) strengthening
- Supporting: 6 pairs
- Score: 1.0

Result: VALIDATED - Risk-off event detected
Reasoning: Strong risk-off sentiment with JPY strengthening across all crosses. 
Flight to safety in progress. USD/JPY sell signal validated by broad JPY strength.
```

## Guardrails Implementation

```python
def validate_correlation_risk(analysis):
    """Validate correlation and confluence requirements"""
    
    # Check for high correlation with existing positions
    high_corr = [
        c for c in analysis.correlations.withExistingPositions
        if abs(c.correlation) > 0.8
    ]
    if high_corr:
        return False, f"High correlation ({high_corr[0].correlation}) with {high_corr[0].pair}"
    
    # Check DXY divergence for USD pairs
    if "USD" in analysis.targetPair and analysis.dxy:
        if analysis.dxy.trend == "RISING" and analysis.targetPair.endswith("USD"):
            # DXY rising but buying XXX/USD (expecting USD weakness)
            return False, "DXY rising contradicts USD weakness signal"
    
    # Check confluence
    if analysis.confluence.confluenceScore < 0.5:
        return False, f"Poor confluence ({analysis.confluence.confluenceScore})"
    
    # Check JPY divergence
    if "JPY" in analysis.targetPair:
        if analysis.jpyAnalysis.alignment == "DIVERGENT":
            return False, "JPY crosses showing divergence - unclear risk sentiment"
    
    return True, "All correlation checks passed"
```

## Integration Points

### With Technical Agent
- Validate technical signals with DXY/JPY trends
- Confirm support/resistance with currency strength

### With Fundamental Agent
- Correlate interest rate differentials with DXY
- Validate risk sentiment with JPY analysis

### With Risk Agent
- Provide correlation data for portfolio risk
- Flag overexposure through correlated positions

### With Orchestrator
- Provide confluence validation for final decision
- Alert on correlation breakdowns or divergences

## Performance Metrics

- Confluence validation accuracy
- DXY divergence detection rate
- Risk-off event early detection
- Correlation breakdown identification
- Hidden risk prevention (avoided losses)

---

**Version**: 1.0  
**Last Updated**: November 23, 2025  
**Status**: Specification - Ready for Implementation
