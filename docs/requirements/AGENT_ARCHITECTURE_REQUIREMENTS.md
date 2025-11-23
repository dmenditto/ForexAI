# Multi-Agent Forex Trading System - Architecture Requirements

## Overview

High-level requirements for implementing a multi-agent AI system for Forex trading using AWS Bedrock Agents with DeepSeek LLM. This document defines the agent architecture, interfaces, and MCP servers needed to integrate with an existing Pepperstone-based trading framework.

## System Architecture

### Agent Hierarchy

```
Orchestrator Agent (Master Coordinator)
    ├── Technical Analysis Agent
    ├── Fundamental Analysis Agent  
    ├── Sentiment Analysis Agent
    ├── Risk Management Agent
    └── Execution Agent
```

### Technology Stack

- **Agent Platform**: AWS Bedrock Agents
- **LLM**: DeepSeek (via API Gateway + Lambda proxy)
- **MCP Servers**: AWS Lambda functions
- **Data Storage**: DynamoDB (signals/decisions), Timestream (price data)
- **Broker**: Pepperstone API (existing integration)

## Agent Specifications

### 1. Orchestrator Agent

**Role**: Central coordinator and final decision maker

**Inputs**:
- Technical analysis signals
- Fundamental analysis reports
- Sentiment scores
- Risk assessments
- Current portfolio state

**Outputs**:
- Trading decisions (BUY/SELL/HOLD/REJECT)
- Position sizing
- Stop-loss and take-profit levels

**Decision Logic**:
- Requires consensus from ≥2 specialized agents
- Always validates with Risk Management Agent
- Resolves conflicts based on confidence scores and timeframes
- Prioritizes capital preservation

### 2. Technical Analysis Agent

**Role**: Chart patterns and technical indicators

**Responsibilities**:
- Analyze price action and patterns
- Calculate technical indicators (RSI, MACD, MA, Bollinger Bands)
- Identify support/resistance levels
- Generate entry/exit signals

**Required Tools**:
- Historical price data (OHLCV)
- Real-time price feeds
- Technical indicator calculations

### 3. Fundamental Analysis Agent

**Role**: Economic and policy analysis

**Responsibilities**:
- Monitor economic indicators (GDP, inflation, employment)
- Analyze central bank policies and interest rates
- Track economic calendar events
- Assess geopolitical factors

**Required Tools**:
- Economic calendar API
- Interest rate data
- Economic indicators (CPI, GDP, etc.)

### 4. Sentiment Analysis Agent

**Role**: Market sentiment and news analysis

**Responsibilities**:
- Analyze financial news
- Monitor social media sentiment
- Track institutional positioning (COT reports)
- Identify market narratives

**Required Tools**:
- News aggregation API
- Sentiment analysis (NLP)
- COT reports
- Social media feeds (optional)

### 5. Risk Management Agent

**Role**: Portfolio risk assessment and control

**Responsibilities**:
- Calculate position sizes
- Monitor exposure limits
- Assess correlation risks
- Calculate Value at Risk (VaR)
- Enforce risk parameters

**Risk Parameters**:
- Max risk per trade: 1-2% of account
- Max exposure per currency: 10%
- Daily loss limit: 5% of account
- Min risk/reward ratio: 2:1

### 6. Execution Agent

**Role**: Order execution via Pepperstone

**Responsibilities**:
- Execute trades via existing Pepperstone integration
- Monitor order status
- Manage open positions
- Report execution quality

## MCP Server Requirements

### Server 1: Market Data
**Tools**:
- `get_live_price(pair, timeframe)` - Current price
- `get_historical_data(pair, start, end, timeframe)` - OHLCV data
- `get_spread(pair)` - Bid-ask spread

**Data Source**: Pepperstone API (existing integration)

### Server 2: Economic Data
**Tools**:
- `get_economic_calendar(start, end, countries)` - Upcoming events
- `get_interest_rates(countries)` - Central bank rates
- `get_economic_indicator(country, indicator, period)` - Historical data

**Data Sources**: Trading Economics API or FRED (free)

### Server 3: News & Sentiment
**Tools**:
- `get_news(keywords, sources, time_range)` - News articles
- `analyze_sentiment(text)` - NLP sentiment score
- `get_cot_report(currency)` - Institutional positioning

**Data Sources**: NewsAPI, CFTC COT reports

### Server 4: Technical Analysis
**Tools**:
- `calculate_indicator(pair, indicator, params)` - Any indicator
- `detect_patterns(pair, timeframe)` - Chart patterns
- `find_support_resistance(pair, timeframe)` - Key levels

**Implementation**: TA-Lib or Pandas-TA

### Server 5: Risk Management
**Tools**:
- `calculate_position_size(balance, risk_pct, stop_loss_pips)` - Position sizing
- `calculate_var(portfolio, confidence)` - Value at Risk
- `check_correlation(pairs)` - Currency correlation
- `validate_trade(trade, portfolio, risk_params)` - Risk approval

### Server 6: Broker Integration
**Tools**:
- `place_order(pair, side, size, type, sl, tp)` - Execute trade
- `get_open_positions()` - List positions
- `close_position(position_id)` - Close trade
- `get_account_info()` - Balance and margin

**Integration**: Use existing Pepperstone API implementation

## Interface Definitions

### Agent Communication Message
```typescript
interface AgentMessage {
  id: string;
  timestamp: Date;
  sender: AgentType;
  recipient: AgentType;
  messageType: "request" | "response" | "signal" | "alert";
  payload: any;
  priority: 1 | 2 | 3 | 4;
}
```

### Trading Signal
```typescript
interface TradingSignal {
  agent: AgentType;
  timestamp: Date;
  pair: string;
  action: "BUY" | "SELL" | "HOLD" | "CLOSE";
  confidence: number; // 0-1
  reasoning: string;
  timeframe: string;
  metadata: {
    indicators?: Record<string, number>;
    patterns?: string[];
    fundamentals?: any;
    sentiment?: number;
  };
}
```

### Technical Analysis Output
```typescript
interface TechnicalAnalysis {
  pair: string;
  timeframe: string;
  timestamp: Date;
  trend: "BULLISH" | "BEARISH" | "NEUTRAL";
  strength: number; // 0-1
  signals: {
    entry: number | null;
    stopLoss: number | null;
    takeProfit: number | null;
  };
  indicators: {
    rsi: number;
    macd: { value: number; signal: number; histogram: number };
    movingAverages: { sma20: number; sma50: number; ema200: number };
  };
  patterns: Array<{ name: string; type: string; confidence: number }>;
}
```

### Risk Assessment Output
```typescript
interface RiskAssessment {
  timestamp: Date;
  proposedTrade: {
    pair: string;
    size: number;
    riskAmount: number;
    riskPercent: number;
    approved: boolean;
    reason?: string;
  };
  currentExposure: Record<string, number>;
  var95: number;
  recommendations: string[];
}
```

### Orchestrator Decision
```typescript
interface OrchestratorDecision {
  timestamp: Date;
  pair: string;
  decision: "EXECUTE" | "HOLD" | "CLOSE" | "REJECT";
  action?: "BUY" | "SELL";
  reasoning: string;
  agentInputs: {
    technical: TechnicalAnalysis;
    fundamental: any;
    sentiment: any;
    risk: RiskAssessment;
  };
  positionSize?: number;
  stopLoss?: number;
  takeProfit?: number;
  confidence: number;
}
```

## AWS Infrastructure Requirements

### Core Services
- **Amazon Bedrock Agents**: Host 6 AI agents
- **AWS Lambda**: 6 MCP server functions
- **DynamoDB**: 3 tables (Trades, Decisions, Signals)
- **Amazon Timestream**: Price data storage
- **S3**: Historical data and logs
- **EventBridge**: Scheduling (1min for market data, 5min for news)
- **Secrets Manager**: API credentials
- **CloudWatch**: Monitoring and alerts

### DeepSeek Integration
**Recommended**: API Gateway + Lambda proxy

```python
# Lambda proxy for DeepSeek API
def lambda_handler(event, context):
    response = requests.post(
        'https://api.deepseek.com/v1/chat/completions',
        headers={'Authorization': f'Bearer {api_key}'},
        json={'model': 'deepseek-chat', 'messages': event['messages']}
    )
    return response.json()
```

### Bedrock Agent Configuration
Each agent configured with:
- Model: DeepSeek (via custom endpoint)
- Instructions: Agent-specific system prompt
- Action Groups: Linked to MCP server Lambda functions (OpenAPI schema)

## System Workflow

### 1. Continuous Monitoring
- Technical Agent: Price updates every 1-5 seconds
- Fundamental Agent: Economic calendar every hour
- Sentiment Agent: News feeds every 5-15 minutes
- Risk Agent: Portfolio state every minute

### 2. Signal Generation
Agent detects opportunity → Generates signal → Sends to Orchestrator

### 3. Decision Making
```
Orchestrator receives signal
    ↓
Requests analysis from all agents
    ↓
Aggregates inputs and resolves conflicts
    ↓
Consults Risk Agent for approval
    ↓
Makes final decision (EXECUTE/HOLD/REJECT)
    ↓
If approved → Execution Agent → Pepperstone API
```

### 4. Position Management
Risk Agent monitors positions → Alerts on threshold breach → Orchestrator may close/modify

## Agent Prompts (Summary)

### Design Philosophy (Inspired by nof1.ai Alpha Arena)

**Key Insight**: nof1.ai's "Monk Mode" approach shows that **shorter, more opinionated prompts with strict guardrails** can outperform verbose instructions. Their experiments with real money trading reveal:
- Sensitivity to prompt changes significantly impacts performance
- Opinionated guardrails on trading thresholds improve risk management
- Concise system prompts (~50% shorter) can be more effective
- Real behavioral differences emerge from prompt design (risk tolerance, position sizing, holding time)

**Recommended Approach**: "Monk Mode" style prompts - concise, opinionated, with strict guardrails

### Orchestrator
- Coordinate specialized agents
- Aggregate analyses and make final decisions
- Require ≥2 agents in agreement
- Always validate with Risk Agent
- Prioritize capital preservation
- **Strict guardrails**: No trade without risk approval, max 3 positions simultaneously

### Technical Analysis
- Analyze charts and indicators
- Provide specific entry/exit levels
- Assign confidence based on indicator confluence
- Consider multiple timeframes
- **Guardrails**: Only signal when ≥3 indicators align, confidence >0.7

### Fundamental Analysis
- Focus on interest rate differentials
- Consider upcoming high-impact events
- Assess economic divergence
- Provide medium to long-term outlook
- **Guardrails**: Flag high-impact events within 24h, avoid trading during major announcements

### Sentiment Analysis
- Identify prevailing market narrative
- Detect sentiment extremes (contrarian signals)
- Note divergences between sentiment and price
- Weight by source reliability
- **Guardrails**: Extreme sentiment (>0.8 or <-0.8) triggers contrarian consideration

### Risk Management
- Calculate position sizes (1-2% risk per trade)
- Check exposure and correlation
- Enforce all risk limits
- Approve/reject with clear reasoning
- **Strict guardrails**: Auto-reject if daily loss >3%, correlation >0.7, or exposure >10% per currency

## Data Sources

### Market Data
- Pepperstone API (primary, existing integration)
- Alpha Vantage (backup, free tier)

### Economic Data
- FRED API (free, US + global data)
- Trading Economics (paid, comprehensive)

### News & Sentiment
- NewsAPI (free tier: 100 req/day)
- CFTC COT reports (free, weekly)
- Twitter/Reddit APIs (optional)

### Technical Analysis
- TA-Lib (open source)
- Pandas-TA (open source)

## Security Requirements

- Store all API keys in AWS Secrets Manager
- Use IAM roles with least-privilege access
- Enable encryption at rest (DynamoDB, S3)
- Implement rate limiting and retry logic
- Enable CloudTrail for audit logging

## Testing Requirements

### Unit Tests
- Test each agent's decision logic
- Test MCP tool functions
- Test data validation

### Integration Tests
- Test agent communication
- Test MCP server connections
- Test Pepperstone API integration (use existing tests)

### Backtesting
- Historical data replay
- Measure performance metrics
- Optimize parameters

### Paper Trading
- Live data, simulated execution
- 2-4 weeks minimum before live trading
- Validate full system integration

## Cost Estimate

**Development**: ~$100-180/month
**Production**: ~$430-950/month

Breakdown:
- Bedrock Agents (DeepSeek): $50-500/month
- Lambda: $10-100/month
- DynamoDB: $5-50/month
- Timestream: $20-200/month
- Other services: $15-100/month

## Implementation Phases

1. **Infrastructure** (1-2 weeks): Deploy AWS resources
2. **MCP Servers** (2-3 weeks): Implement Lambda functions
3. **Agents** (2-3 weeks): Configure Bedrock Agents with DeepSeek
4. **Integration** (1-2 weeks): Connect all components
5. **Testing** (2-4 weeks): Unit, integration, backtesting
6. **Paper Trading** (2-4 weeks): Live data validation
7. **Live Trading** (ongoing): Start small, monitor closely

## Integration Points with Existing Framework

### Required from Existing System
- Pepperstone API client (authentication, order execution)
- Account management functions
- Position tracking
- Historical data storage

### Provided by Multi-Agent System
- AI-driven decision making
- Multi-factor analysis (technical, fundamental, sentiment)
- Risk management validation
- Automated signal generation

### Integration Method
Execution Agent calls existing Pepperstone integration via:
- Direct function calls (if same codebase)
- REST API (if separate service)
- Message queue (for async processing)

## Success Metrics

### Performance Metrics
- Win rate > 50%
- Sharpe ratio > 1.0
- Maximum drawdown < 15%
- Monthly return > 5%
- Agent agreement rate (measure consensus quality)
- Execution quality (slippage < 1 pip average)
- System uptime > 99%

### Behavioral Metrics (Inspired by nof1.ai)
- **Risk consistency**: Standard deviation of position sizes
- **Holding time distribution**: Average trade duration
- **Sizing discipline**: Adherence to risk limits
- **Prompt sensitivity**: A/B test different prompt variations
- **Guardrail effectiveness**: % of trades blocked by risk rules

### Benchmark Comparison
- Compare against nof1.ai Alpha Arena results (equities)
- Adapt learnings to Forex market characteristics
- Track model performance across different market conditions

## Key Learnings from nof1.ai Alpha Arena

### Validated Concepts
1. **AI can trade autonomously in real markets** - Multiple LLMs successfully trading with real money
2. **Prompt engineering is critical** - Small prompt changes significantly impact performance
3. **Guardrails improve outcomes** - Opinionated risk management rules in prompts work better
4. **Shorter prompts can outperform** - "Monk Mode" (~50% shorter) shows competitive results
5. **Behavioral differences matter** - Models show distinct risk tolerance, sizing, and timing patterns

### Competition Formats (Applicable to Forex)
- **Baseline**: Standard approach with news/sentiment data
- **Monk Mode**: Shorter prompts, stricter guardrails (recommended starting point)
- **Situational Awareness**: Context about market state and other positions
- **Max Leverage**: Stress-test risk management

### Recommended Implementation Strategy
1. **Start with "Monk Mode" approach**: Concise prompts with strict guardrails
2. **A/B test prompt variations**: Measure sensitivity to prompt changes
3. **Track behavioral metrics**: Monitor risk consistency, holding times, sizing discipline
4. **Iterate based on real performance**: Use paper trading data to refine prompts
5. **Consider multiple "modes"**: Different prompt strategies for different market conditions

### Differences: Equities vs Forex
- **Market hours**: Forex is 24/5 vs equities 9:30-4:00 ET
- **Leverage**: Forex typically higher leverage (50:1-500:1 vs 2:1-4:1)
- **Volatility**: Currency pairs generally less volatile than individual stocks
- **Fundamentals**: Interest rates and macro data more important in Forex
- **Liquidity**: Major Forex pairs have higher liquidity than most stocks

### Action Items
- [ ] Design "Monk Mode" style prompts for each agent (concise, opinionated)
- [ ] Define strict guardrails for risk management
- [ ] Plan A/B testing framework for prompt variations
- [ ] Set up behavioral metrics tracking
- [ ] Consider multiple competition formats for robustness testing

---

**Document Purpose**: High-level requirements for integrating multi-agent AI architecture into existing Forex trading framework with Pepperstone API.

**Inspiration**: nof1.ai Alpha Arena - validated AI autonomous trading with real money in real markets.

**Next Steps**: Review requirements, adapt to existing architecture, implement "Monk Mode" style prompts with strict guardrails, start with infrastructure and MCP servers.
