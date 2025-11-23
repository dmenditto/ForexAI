# Multi-Agent Forex Trading System - Architecture Requirements

## Document Overview

Complete architecture and requirements for an autonomous Forex trading system using AWS Bedrock Agents, DeepSeek LLM, and Pepperstone broker integration. This document consolidates all specifications needed for implementation.

**Status**: Architecture Complete - Ready for Implementation  
**Last Updated**: November 23, 2025  
**Version**: 1.0

## System Architecture

### Agent Hierarchy

```
Orchestrator Agent (Master Coordinator)
    ├── Technical Analysis Agent (Charts & Indicators)
    ├── Fundamental Analysis Agent (Economic Data)
    ├── Correlation & Confluence Agent (DXY, JPY, Correlations) ⭐
    ├── Sentiment Analysis Agent (News & Social Media)
    ├── Risk Management Agent (Position Sizing & Validation)
    └── Execution Agent (Pepperstone Trade Execution)
```

### Technology Stack

- **Agent Platform**: AWS Bedrock Agents
- **LLM**: DeepSeek (via API Gateway + Lambda proxy)
- **MCP Servers**: AWS Lambda functions
- **Data Storage**: DynamoDB (signals/decisions), Timestream (price data)
- **Broker**: Pepperstone API (existing integration)

## Agent Specifications

## Documentation Structure

```
docs/
├── requirements/
│   ├── AGENT_ARCHITECTURE_REQUIREMENTS.md (this document)
│   └── RESEARCH_REFERENCES.md
├── agents/
│   ├── 01_orchestrator_agent.md
│   ├── 02_technical_analysis_agent.md
│   ├── 03_fundamental_analysis_agent.md
│   ├── 04_correlation_confluence_agent.md ⭐
│   ├── 05_sentiment_analysis_agent.md
│   ├── 06_risk_management_agent.md
│   └── 07_execution_agent.md
└── mcp-tools/
    ├── 01_market_data_tools.md
    ├── 02_economic_data_tools.md
    ├── 03_news_sentiment_tools.md
    ├── 04_technical_analysis_tools.md
    ├── 05_correlation_dxy_tools.md ⭐
    ├── 06_risk_management_tools.md
    └── 07_broker_integration_tools.md
```

## Agent Specifications Summary

### 1. Orchestrator Agent

**Role**: Central coordinator and final decision maker  
**Detailed Spec**: [docs/agents/01_orchestrator_agent.md](../agents/01_orchestrator_agent.md)

**Key Features**:
- Coordinates all specialist agents
- Requires consensus from ≥2 agents
- Mandatory risk validation
- Resolves conflicts with weighted voting
- Enforces strict guardrails (max 3 positions, no trade without risk approval)

**Decision Logic**:
- Requires ≥2 specialized agents in agreement
- Always validates with Risk Management Agent
- Checks DXY/JPY alignment for USD/JPY pairs (mandatory)
- Validates confluence across related pairs
- Rejects if correlation with existing position >0.8
- Prioritizes capital preservation

### 2. Technical Analysis Agent

**Role**: Chart patterns and technical indicators  
**Detailed Spec**: [docs/agents/02_technical_analysis_agent.md](../agents/02_technical_analysis_agent.md)

**Key Features**:
- Multi-timeframe analysis (H1, H4, D1)
- Indicator confluence (requires ≥3 indicators aligned)
- Pattern recognition
- Support/resistance identification
- Strict guardrails: confidence >0.7, R/R ratio ≥2:1

**Indicators**: RSI, MACD, Moving Averages, Bollinger Bands, Support/Resistance

### 3. Fundamental Analysis Agent

**Role**: Economic data and central bank policies  
**Detailed Spec**: [docs/agents/03_fundamental_analysis_agent.md](../agents/03_fundamental_analysis_agent.md)

**Key Features**:
- Interest rate differentials (primary driver)
- Economic calendar monitoring
- GDP, inflation, employment analysis
- Central bank stance assessment
- Flags high-impact events within 24h

**Focus**: Interest rates, GDP, inflation, employment, central bank rhetoric

### 4. Correlation & Confluence Agent ⭐

**Role**: Currency correlations, DXY, and JPY analysis  
**Detailed Spec**: [docs/agents/04_correlation_confluence_agent.md](../agents/04_correlation_confluence_agent.md)

**Key Features** (Critical for Forex):
- **DXY monitoring**: Validates USD pair signals against US Dollar Index
- **JPY crosses analysis**: Risk-on/risk-off sentiment detection
- **Currency correlations**: 30-day rolling correlation matrix
- **Cross-pair confluence**: Validates signals across related pairs
- **Currency strength index**: Individual currency strength (0-100)

**Guardrails**: Rejects correlation >0.8, requires confluence >0.6

### 5. Sentiment Analysis Agent

**Role**: News and market sentiment  
**Detailed Spec**: [docs/agents/05_sentiment_analysis_agent.md](../agents/05_sentiment_analysis_agent.md)

**Key Features**:
- Financial news aggregation and NLP sentiment
- Social media monitoring (Twitter, Reddit)
- COT reports (institutional positioning)
- Contrarian signals at sentiment extremes
- Flags extreme sentiment (>0.8 or <-0.8)

**Sources**: Financial news (primary), institutional positioning, social media

### 6. Risk Management Agent

**Role**: Position sizing and risk validation  
**Detailed Spec**: [docs/agents/06_risk_management_agent.md](../agents/06_risk_management_agent.md)

**Key Features**:
- Position size calculation (1-2% risk per trade)
- VaR and exposure monitoring
- Correlation checks
- Final gatekeeper (can reject any trade)

**Strict Guardrails**:
- Max 2% risk per trade
- Max 10% exposure per currency
- Auto-reject if daily loss ≥5%
- Auto-reject if correlation >0.8
- Auto-reject if margin level <150%

### 7. Execution Agent

**Role**: Trade execution via Pepperstone  
**Detailed Spec**: [docs/agents/07_execution_agent.md](../agents/07_execution_agent.md)

**Key Features**:
- Order placement via Pepperstone API
- Execution monitoring and slippage tracking
- Position management
- Only executes approved trades
- Mandatory stop-loss on every trade

## MCP Server Requirements

**Total**: 7 MCP Servers, 35 Tools  
**Detailed Specifications**: See [docs/mcp-tools/](../mcp-tools/) for complete tool schemas

### Server 1: Market Data Server
**Lambda**: `forex-market-data`  
**Detailed Spec**: [01_market_data_tools.md](../mcp-tools/01_market_data_tools.md)

**Tools** (5):
- `get_live_price` - Real-time price data
- `get_historical_data` - OHLCV historical data
- `get_spread` - Current bid-ask spread
- `get_tick_data` - Tick-level data for analysis
- `get_market_hours` - Trading session status

**Data Source**: Pepperstone API (existing integration)  
**Used By**: Technical Analysis Agent, Correlation Agent

### Server 2: Economic Data Server
**Lambda**: `forex-economic-data`  
**Detailed Spec**: [02_economic_data_tools.md](../mcp-tools/02_economic_data_tools.md)

**Tools** (4):
- `get_economic_calendar` - Upcoming economic events
- `get_interest_rates` - Central bank rates
- `get_economic_indicator` - Historical economic data
- `get_rate_differential` - Interest rate differentials

**Data Sources**: FRED API (free), Trading Economics (paid)  
**Used By**: Fundamental Analysis Agent

### Server 3: News & Sentiment Server
**Lambda**: `forex-news-sentiment`  
**Detailed Spec**: [03_news_sentiment_tools.md](../mcp-tools/03_news_sentiment_tools.md)

**Tools** (4):
- `get_news` - News articles with sentiment
- `analyze_sentiment` - NLP sentiment analysis
- `get_social_sentiment` - Social media sentiment
- `get_cot_report` - CFTC Commitment of Traders

**Data Sources**: NewsAPI, CFTC COT reports, Twitter API  
**Used By**: Sentiment Analysis Agent

### Server 4: Technical Analysis Server
**Lambda**: `forex-technical-analysis`  
**Detailed Spec**: [04_technical_analysis_tools.md](../mcp-tools/04_technical_analysis_tools.md)

**Tools** (6):
- `calculate_indicator` - Any technical indicator
- `detect_patterns` - Chart pattern recognition
- `find_support_resistance` - Key price levels
- `calculate_pivot_points` - Pivot levels
- `analyze_volume` - Volume analysis
- `get_fibonacci_levels` - Fibonacci retracements

**Implementation**: TA-Lib or Pandas-TA  
**Used By**: Technical Analysis Agent

### Server 5: Correlation & DXY Server ⭐
**Lambda**: `forex-correlation-dxy`  
**Detailed Spec**: [05_correlation_dxy_tools.md](../mcp-tools/05_correlation_dxy_tools.md)

**Tools** (7):
- `get_dxy_data` - US Dollar Index data
- `get_jpy_crosses` - All JPY pairs for risk sentiment
- `calculate_correlation` - Pair correlation matrix
- `get_currency_strength` - Individual currency strength
- `check_confluence` - Cross-pair signal validation
- `get_related_pairs` - Find correlated pairs
- `analyze_divergence` - Price divergence detection

**Data Sources**: TradingView, Investing.com  
**Used By**: Correlation & Confluence Agent (critical for Forex)

### Server 6: Risk Management Server
**Lambda**: `forex-risk-management`  
**Detailed Spec**: [06_risk_management_tools.md](../mcp-tools/06_risk_management_tools.md)

**Tools** (4):
- `calculate_position_size` - Optimal position sizing
- `calculate_var` - Value at Risk (95% confidence)
- `check_exposure` - Current currency exposure
- `validate_trade` - Trade validation against risk rules

**Implementation**: Custom risk calculations  
**Used By**: Risk Management Agent

### Server 7: Broker Integration Server
**Lambda**: `forex-broker-integration`  
**Detailed Spec**: [07_broker_integration_tools.md](../mcp-tools/07_broker_integration_tools.md)

**Tools** (5):
- `place_order` - Execute trade (market/limit/stop)
- `get_open_positions` - List all positions
- `close_position` - Close a position
- `get_account_info` - Balance and margin info
- `modify_position` - Update SL/TP

**Integration**: Existing Pepperstone API implementation  
**Used By**: Execution Agent, Risk Management Agent

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
