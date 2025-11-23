# Orchestrator Agent Specification

## Overview

The Orchestrator Agent is the master coordinator and final decision maker in the Forex trading multi-agent system. It aggregates insights from specialized agents, resolves conflicts, and makes final trading decisions.

## Role & Responsibilities

### Primary Role
Central coordinator that synthesizes inputs from all specialist agents to make informed trading decisions.

### Key Responsibilities
1. **Coordinate Specialist Agents**: Request and collect analysis from Technical, Fundamental, Sentiment, and Risk agents
2. **Aggregate Insights**: Synthesize multiple perspectives into coherent trading signals
3. **Resolve Conflicts**: Handle disagreements between agents using weighted decision logic
4. **Make Final Decisions**: Execute, hold, or reject trading opportunities
5. **Enforce Risk Limits**: Ensure all trades comply with risk management rules
6. **Monitor Portfolio**: Track overall portfolio health and performance

## Agent Configuration

### AWS Bedrock Agent Setup

```yaml
AgentName: OrchestratorAgent
AgentType: Supervisor
Model: deepseek-chat (via API Gateway Lambda proxy)
Description: Master coordinator for Forex trading decisions

CollaboratorAgents:
  - TechnicalAnalysisAgent
  - FundamentalAnalysisAgent
  - CorrelationConfluenceAgent
  - SentimentAnalysisAgent
  - RiskManagementAgent
  - ExecutionAgent

ActionGroups:
  - RequestAnalysis
  - MakeDecision
  - MonitorPortfolio
```

## System Prompt ("Monk Mode" Style)

```
You are the Orchestrator for a Forex trading system. Your role: coordinate specialists, aggregate analysis, make final trading decisions.

DECISION FRAMEWORK:
- Require ≥2 specialist agents in agreement before trading
- Always validate with Risk Agent (mandatory)
- Weight signals by agent confidence and historical accuracy
- Prioritize capital preservation over profit maximization

STRICT GUARDRAILS:
- No trade without Risk Agent approval
- Maximum 3 positions simultaneously
- Never override Risk Agent rejection
- Close all positions if daily loss exceeds 3%

WORKFLOW:
1. Receive opportunity signal from any agent
2. Request analysis from Technical, Fundamental, Correlation, Sentiment agents
3. Check DXY/JPY alignment and confluence (mandatory for USD/JPY pairs)
4. Evaluate consensus (need ≥2 agents agreeing)
5. Consult Risk Agent for position sizing and approval
6. Make decision: EXECUTE, HOLD, or REJECT
7. If EXECUTE: send to Execution Agent
8. Log decision with detailed reasoning

CORRELATION CHECKS (Mandatory):
- For USD pairs: Verify DXY alignment
- For JPY pairs: Check risk sentiment alignment
- For all pairs: Validate confluence across related pairs
- Reject if correlation with existing position >0.8

CONFLICT RESOLUTION:
- Technical + Sentiment agree, Fundamental disagrees → Consider timeframe (short vs long)
- All agents disagree → REJECT
- 2+ agents agree + Risk approves → EXECUTE
- Risk rejects → Always REJECT

OUTPUT FORMAT:
Decision: [EXECUTE/HOLD/REJECT]
Action: [BUY/SELL/null]
Pair: [currency pair]
Reasoning: [2-3 sentences explaining decision]
Consensus: [which agents agreed]
Confidence: [0-1]
Position Size: [if EXECUTE]
Stop Loss: [if EXECUTE]
Take Profit: [if EXECUTE]
```

## Input Schema

### From Specialist Agents

```typescript
interface AgentInput {
  technical: TechnicalAnalysis;
  fundamental: FundamentalAnalysis;
  correlation: CorrelationAnalysis;
  sentiment: SentimentAnalysis;
  risk: RiskAssessment;
}

interface TechnicalAnalysis {
  pair: string;
  trend: "BULLISH" | "BEARISH" | "NEUTRAL";
  strength: number; // 0-1
  confidence: number; // 0-1
  signals: {
    entry: number | null;
    stopLoss: number | null;
    takeProfit: number | null;
  };
  reasoning: string;
}

interface FundamentalAnalysis {
  pair: string;
  bias: "BULLISH" | "BEARISH" | "NEUTRAL";
  confidence: number; // 0-1
  interestRateDifferential: number;
  upcomingEvents: Array<{
    date: Date;
    event: string;
    importance: "LOW" | "MEDIUM" | "HIGH";
  }>;
  reasoning: string;
}

interface SentimentAnalysis {
  pair: string;
  overallSentiment: number; // -1 to 1
  confidence: number; // 0-1
  marketNarrative: string;
  reasoning: string;
}

interface RiskAssessment {
  approved: boolean;
  positionSize: number;
  riskAmount: number;
  riskPercent: number;
  reason?: string;
  recommendations: string[];
}
```

## Output Schema

```typescript
interface OrchestratorDecision {
  timestamp: Date;
  pair: string;
  decision: "EXECUTE" | "HOLD" | "REJECT";
  action?: "BUY" | "SELL";
  reasoning: string;
  agentInputs: AgentInput;
  consensus: {
    agreeing: string[]; // List of agreeing agents
    disagreeing: string[];
    conflictResolution?: string;
  };
  positionSize?: number;
  stopLoss?: number;
  takeProfit?: number;
  confidence: number; // 0-1
  riskApproved: boolean;
}
```

## Decision Logic

### Consensus Requirements

```python
def evaluate_consensus(technical, fundamental, sentiment):
    """Evaluate if there's sufficient consensus"""
    signals = []
    
    if technical.trend == "BULLISH" and technical.confidence > 0.7:
        signals.append(("technical", "BUY", technical.confidence))
    elif technical.trend == "BEARISH" and technical.confidence > 0.7:
        signals.append(("technical", "SELL", technical.confidence))
    
    if fundamental.bias == "BULLISH" and fundamental.confidence > 0.6:
        signals.append(("fundamental", "BUY", fundamental.confidence))
    elif fundamental.bias == "BEARISH" and fundamental.confidence > 0.6:
        signals.append(("fundamental", "SELL", fundamental.confidence))
    
    if sentiment.overallSentiment > 0.3 and sentiment.confidence > 0.6:
        signals.append(("sentiment", "BUY", sentiment.confidence))
    elif sentiment.overallSentiment < -0.3 and sentiment.confidence > 0.6:
        signals.append(("sentiment", "SELL", sentiment.confidence))
    
    # Count agreements
    buy_signals = [s for s in signals if s[1] == "BUY"]
    sell_signals = [s for s in signals if s[1] == "SELL"]
    
    if len(buy_signals) >= 2:
        return "BUY", buy_signals
    elif len(sell_signals) >= 2:
        return "SELL", sell_signals
    else:
        return "HOLD", []
```

### Confidence Calculation

```python
def calculate_confidence(agreeing_signals):
    """Calculate overall confidence based on agreeing agents"""
    if not agreeing_signals:
        return 0.0
    
    # Weight by agent confidence
    weighted_sum = sum(signal[2] for signal in agreeing_signals)
    confidence = weighted_sum / len(agreeing_signals)
    
    # Bonus for more agents agreeing
    if len(agreeing_signals) >= 3:
        confidence = min(1.0, confidence * 1.1)
    
    return confidence
```

## Action Groups

### 1. Request Analysis

**Tools**:
- `request_technical_analysis(pair, timeframe)`
- `request_fundamental_analysis(pair)`
- `request_sentiment_analysis(pair)`
- `request_risk_assessment(pair, proposed_trade)`

### 2. Make Decision

**Tools**:
- `evaluate_consensus(agent_inputs)`
- `calculate_confidence(signals)`
- `resolve_conflicts(agent_inputs)`
- `log_decision(decision)`

### 3. Monitor Portfolio

**Tools**:
- `get_portfolio_state()`
- `get_open_positions()`
- `check_daily_pnl()`
- `evaluate_portfolio_risk()`

## Guardrails

### Hard Limits (Auto-Reject)
1. Risk Agent rejects → Always REJECT
2. Daily loss > 3% → Close all positions, stop trading
3. More than 3 open positions → Reject new trades
4. Less than 2 agents agreeing → REJECT
5. High-impact event within 2 hours → HOLD

### Soft Limits (Warning)
1. Confidence < 0.6 → Flag for review
2. Conflicting signals → Require manual review
3. Unusual market conditions → Reduce position size by 50%

## Performance Metrics

### Decision Quality
- Consensus rate: % of decisions with ≥2 agents agreeing
- Confidence accuracy: Correlation between confidence and outcome
- Override rate: % of Risk Agent approvals vs rejections

### Trading Performance
- Win rate by decision type
- Average profit per decision
- Sharpe ratio
- Maximum drawdown

### Behavioral Metrics
- Average time to decision
- Conflict resolution patterns
- Agent agreement patterns

## Example Scenarios

### Scenario 1: Strong Consensus (EXECUTE)

```
Technical: BULLISH (0.85 confidence) - RSI oversold, MACD crossover
Fundamental: BULLISH (0.75 confidence) - Interest rate differential favors EUR
Sentiment: NEUTRAL (0.50 confidence) - Mixed signals
Risk: APPROVED (1% risk, position size: 10,000 units)

Decision: EXECUTE BUY EUR/USD
Reasoning: Technical and Fundamental agents strongly agree on bullish outlook. 
Sentiment neutral but not contradicting. Risk approved with appropriate sizing.
Confidence: 0.80
```

### Scenario 2: Conflict (HOLD)

```
Technical: BEARISH (0.80 confidence) - Downtrend, resistance level
Fundamental: BULLISH (0.70 confidence) - Strong economic data
Sentiment: BEARISH (0.60 confidence) - Negative news flow
Risk: APPROVED (conditional)

Decision: HOLD
Reasoning: Technical and Sentiment suggest SELL, but Fundamental suggests BUY. 
Conflicting timeframes (short-term bearish, long-term bullish). Wait for clarity.
Confidence: 0.45
```

### Scenario 3: Risk Rejection (REJECT)

```
Technical: BULLISH (0.90 confidence)
Fundamental: BULLISH (0.80 confidence)
Sentiment: BULLISH (0.75 confidence)
Risk: REJECTED (correlation with existing position too high)

Decision: REJECT
Reasoning: All agents agree on bullish signal, but Risk Agent rejected due to 
high correlation with existing EUR/GBP position. Respect risk limits.
Confidence: N/A
```

### Scenario 4: Correlation Divergence (REJECT)

```
Pair: GBP/USD BUY @ 1.2650

Technical: BULLISH (0.85 confidence) - Strong uptrend, indicators aligned
Fundamental: BULLISH (0.70 confidence) - BoE hawkish
Correlation: NOT VALIDATED
  - DXY rising (+0.6%) contradicts USD weakness
  - Existing EUR/USD LONG (correlation 0.85 - HIGH RISK)
  - Poor confluence (33% - only GBP/JPY supporting)
  - USD strength index higher than GBP
Sentiment: NEUTRAL (0.55 confidence)
Risk: CONDITIONAL (would approve if correlation passes)

Decision: REJECT
Reasoning: Despite Technical and Fundamental agreement, Correlation Agent flags 
critical issues: DXY rising shows USD strength (contradicts signal), high 
correlation with existing EUR/USD creates overexposure, poor cross-pair confluence. 
Multiple red flags override positive signals.
Confidence: N/A
```

### Scenario 5: Perfect Confluence (EXECUTE)

```
Pair: EUR/USD BUY @ 1.0850

Technical: BULLISH (0.85 confidence) - Uptrend, RSI neutral, MACD positive
Fundamental: BULLISH (0.75 confidence) - ECB hawkish, rate differential favors EUR
Correlation: VALIDATED (1.0 confluence score)
  - DXY falling (-0.8%) confirms USD weakness
  - EUR/GBP rising (EUR strength confirmed)
  - EUR/JPY rising (EUR strength + risk-on)
  - GBP/USD, AUD/USD rising (USD weakness confirmed)
  - No existing correlated positions
  - Currency strength: EUR 72/100, USD 45/100
Sentiment: BULLISH (0.70 confidence) - Positive EUR news flow
Risk: APPROVED (1% risk, position size: 10,000 units)

Decision: EXECUTE BUY EUR/USD
Reasoning: Perfect alignment across all agents with exceptional confluence. 
DXY falling confirms USD weakness, all related pairs support EUR strength, 
risk-on sentiment favorable. Technical, Fundamental, Correlation, and Sentiment 
all agree. Risk approved. High confidence trade.
Confidence: 0.90
Position Size: 10,000 units
Stop Loss: 1.0800
Take Profit: 1.0950
```

## Integration Points

### With Specialist Agents
- Sends analysis requests
- Receives structured responses
- Tracks agent performance over time

### With Execution Agent
- Sends approved trades
- Monitors execution status
- Receives execution reports

### With Risk Agent
- Mandatory consultation before every trade
- Receives position sizing recommendations
- Gets portfolio risk updates

## Monitoring & Logging

### Log Every Decision
```json
{
  "timestamp": "2025-11-23T10:30:00Z",
  "decision_id": "dec_12345",
  "pair": "EUR/USD",
  "decision": "EXECUTE",
  "action": "BUY",
  "agent_inputs": {...},
  "consensus": {...},
  "confidence": 0.80,
  "position_size": 10000,
  "stop_loss": 1.0800,
  "take_profit": 1.0900,
  "reasoning": "Strong technical and fundamental agreement..."
}
```

### Alert Conditions
- Daily loss approaching 3%
- Unusual number of rejections
- Low confidence decisions
- Agent disagreement patterns changing

## Testing Scenarios

1. **All agents agree** → Should EXECUTE with high confidence
2. **2 agents agree, 1 disagrees** → Should EXECUTE if risk approves
3. **All agents disagree** → Should REJECT
4. **Risk rejects** → Should always REJECT regardless of consensus
5. **Daily loss limit hit** → Should close all positions and stop trading
6. **High-impact event imminent** → Should HOLD new trades

---

**Version**: 1.0  
**Last Updated**: November 23, 2025  
**Status**: Specification - Ready for Implementation
