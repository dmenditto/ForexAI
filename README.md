# Forex Trading Multi-Agent AI System

Complete architecture and specifications for an autonomous Forex trading system using AWS Bedrock Agents, DeepSeek LLM, and Pepperstone broker integration.

## 📋 Project Status

**Phase**: Architecture & Requirements Complete  
**Deployment**: Not yet deployed  
**Purpose**: Production-ready specifications for implementation

## 🎯 System Overview

A hierarchical multi-agent AI system where specialized agents collaborate to make informed Forex trading decisions:

```
Orchestrator Agent (Master Coordinator)
    ├── Technical Analysis Agent (Charts & Indicators)
    ├── Fundamental Analysis Agent (Economic Data)
    ├── Correlation & Confluence Agent (DXY, JPY, Correlations) ⭐
    ├── Sentiment Analysis Agent (News & Social Media)
    ├── Risk Management Agent (Position Sizing & Validation)
    └── Execution Agent (Pepperstone Trade Execution)
```

## 📚 Documentation Structure

```
docs/
├── requirements/
│   ├── AGENT_ARCHITECTURE_REQUIREMENTS.md    # High-level system architecture
│   └── RESEARCH_REFERENCES.md                # Research on multi-agent systems
│
├── agents/
│   ├── 01_orchestrator_agent.md              # Master coordinator
│   ├── 02_technical_analysis_agent.md        # Chart & indicator analysis
│   ├── 03_fundamental_analysis_agent.md      # Economic data analysis
│   ├── 04_correlation_confluence_agent.md    # DXY, JPY, correlations ⭐
│   ├── 05_sentiment_analysis_agent.md        # News & sentiment
│   ├── 06_risk_management_agent.md           # Risk validation
│   └── 07_execution_agent.md                 # Trade execution
│
└── mcp-tools/
    └── MCP_TOOLS_SPECIFICATION.md            # All 35 MCP tools (7 servers)
```

## 🌟 Key Features

### Multi-Agent Collaboration
- **Orchestrator** coordinates all specialist agents
- **Consensus-based** decision making (requires ≥2 agents agreeing)
- **Conflict resolution** with weighted voting
- **Risk validation** mandatory before execution

### Forex-Specific Analysis ⭐
- **DXY (US Dollar Index)** monitoring for USD pair validation
- **JPY crosses** analysis for risk-on/risk-off sentiment
- **Currency correlations** to prevent overexposure
- **Cross-pair confluence** validation (e.g., EUR/USD validated by EUR/GBP, EUR/JPY)
- **Currency strength index** for individual currency analysis

### "Monk Mode" Prompts
Inspired by [nof1.ai Alpha Arena](https://nof1.ai/) research:
- **Concise prompts** (~50% shorter than baseline)
- **Opinionated guardrails** with strict risk limits
- **Clear decision boundaries** for consistent behavior
- **Proven effective** in real-money AI trading competitions

### Risk Management
- Max 1-2% risk per trade
- Max 10% exposure per currency
- Correlation checks (<0.8 with existing positions)
- Daily loss limit (5% circuit breaker)
- Mandatory stop-loss on every trade

### AWS-Native Architecture
- **AWS Bedrock Agents** for multi-agent orchestration
- **DeepSeek LLM** for cost-effective reasoning
- **AWS Lambda** for MCP server functions
- **DynamoDB** for signals and decisions
- **Timestream** for price data
- **Secrets Manager** for API credentials

## 🔧 Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Agent Platform** | AWS Bedrock Agents | Multi-agent orchestration |
| **LLM** | DeepSeek-V3 | Cost-effective reasoning (671B params, 37B activated) |
| **MCP Servers** | AWS Lambda (Python) | Tool functions (35 tools across 7 servers) |
| **Broker** | Pepperstone API | Trade execution |
| **Data Storage** | DynamoDB + Timestream | Signals, decisions, price data |
| **Market Data** | Pepperstone + Alpha Vantage | Real-time and historical prices |
| **Economic Data** | FRED API (free) | Economic indicators |
| **News** | NewsAPI | Financial news aggregation |

## 📊 Agent Specifications

### 1. Orchestrator Agent
- **Role**: Master coordinator and final decision maker
- **Key Features**: Consensus evaluation, conflict resolution, risk enforcement
- **Guardrails**: Requires ≥2 agents agreeing, mandatory risk approval, max 3 positions
- **Decision Logic**: Weighted voting by confidence and historical accuracy

### 2. Technical Analysis Agent
- **Role**: Chart patterns and technical indicators
- **Key Features**: Multi-timeframe analysis, indicator confluence, pattern recognition
- **Guardrails**: Requires ≥3 indicators aligned, confidence >0.7, R/R ratio ≥2:1
- **Indicators**: RSI, MACD, Moving Averages, Bollinger Bands, Support/Resistance

### 3. Fundamental Analysis Agent
- **Role**: Economic data and central bank policies
- **Key Features**: Interest rate differentials, economic calendar, GDP/inflation analysis
- **Guardrails**: Confidence >0.6, flags high-impact events within 24h
- **Focus**: Interest rates (primary), GDP, inflation, employment, central bank stance

### 4. Correlation & Confluence Agent ⭐
- **Role**: Currency correlations, DXY, and JPY analysis
- **Key Features**: 
  - DXY monitoring for USD strength validation
  - JPY crosses for risk sentiment (risk-on/risk-off)
  - Currency pair correlations (30-day rolling)
  - Cross-pair confluence validation
  - Currency strength index
- **Guardrails**: Rejects correlation >0.8, requires confluence >0.6
- **Critical For**: Preventing overexposure, validating USD/JPY signals, detecting risk shifts

### 5. Sentiment Analysis Agent
- **Role**: News and market sentiment
- **Key Features**: News aggregation, NLP sentiment, COT reports, social media
- **Guardrails**: Confidence >0.6, flags extreme sentiment (contrarian signals)
- **Sources**: Financial news (primary), institutional positioning, social media

### 6. Risk Management Agent
- **Role**: Position sizing and risk validation
- **Key Features**: Position size calculation, VaR, exposure monitoring, correlation checks
- **Guardrails**: Max 2% risk per trade, max 10% per currency, auto-reject on violations
- **Critical**: Final gatekeeper - can reject any trade

### 7. Execution Agent
- **Role**: Trade execution via Pepperstone
- **Key Features**: Order placement, execution monitoring, slippage tracking
- **Guardrails**: Only executes approved trades, mandatory stop-loss, max 3 retry attempts
- **Integration**: Uses existing Pepperstone API implementation

## 🛠️ MCP Tools (35 Total)

### Server 1: Market Data (4 tools)
- get_live_price, get_historical_data, get_spread, get_multiple_pairs

### Server 2: Economic Data (4 tools)
- get_economic_calendar, get_interest_rates, get_economic_indicator, get_rate_differential

### Server 3: News & Sentiment (4 tools)
- get_news, analyze_sentiment, get_social_sentiment, get_cot_report

### Server 4: Technical Analysis (4 tools)
- calculate_indicator, detect_patterns, find_support_resistance, calculate_multiple_indicators

### Server 5: Correlation & DXY (5 tools) ⭐
- get_dxy_data, get_currency_strength, calculate_correlation, get_jpy_crosses, validate_confluence

### Server 6: Risk Management (4 tools)
- calculate_position_size, calculate_var, check_exposure, validate_trade

### Server 7: Broker Integration (5 tools)
- place_order, get_open_positions, close_position, get_account_info, modify_position

## 🎓 Research Foundation

### nof1.ai Alpha Arena Insights
- **Real-world validation**: AI models trading with $10K real money
- **"Monk Mode" approach**: Shorter prompts with strict guardrails outperform
- **Proven results**: DeepSeek-v3.1 achieved $8,617 (86% return in 2 weeks)
- **Key learning**: Prompt engineering and risk guardrails are critical

### AWS Bedrock Agents
- Native multi-agent collaboration (Supervisor + Collaborators)
- Perfect match for hierarchical architecture
- Built-in security and guardrails
- Production-ready with auto-scaling

### DeepSeek-V3
- 671B parameters, 37B activated (MoE)
- Comparable to GPT-4 at fraction of cost
- Strong reasoning capabilities
- 128K context window

## 💰 Cost Estimates

### Development Environment
- **~$100-180/month**
- Suitable for testing and backtesting

### Production Environment
- **~$430-950/month**
- Moderate trading activity
- Includes all AWS services and API costs

## 🚀 Implementation Roadmap

### Phase 1: Infrastructure (Weeks 1-2)
- [ ] Deploy AWS infrastructure (Terraform)
- [ ] Set up DynamoDB tables
- [ ] Configure Secrets Manager
- [ ] Set up EventBridge rules

### Phase 2: MCP Servers (Weeks 3-4)
- [ ] Implement Market Data Lambda
- [ ] Implement Economic Data Lambda
- [ ] Implement Correlation & DXY Lambda ⭐
- [ ] Implement News & Sentiment Lambda
- [ ] Implement Technical Analysis Lambda
- [ ] Implement Risk Management Lambda
- [ ] Implement Broker Integration Lambda

### Phase 3: Bedrock Agents (Weeks 5-7)
- [ ] Create Orchestrator Agent
- [ ] Create Technical Analysis Agent
- [ ] Create Fundamental Analysis Agent
- [ ] Create Correlation & Confluence Agent ⭐
- [ ] Create Sentiment Analysis Agent
- [ ] Create Risk Management Agent
- [ ] Create Execution Agent
- [ ] Configure action groups and prompts

### Phase 4: Integration & Testing (Weeks 8-10)
- [ ] Connect agents to MCP servers
- [ ] Test inter-agent communication
- [ ] Backtest with historical data
- [ ] Validate DXY and JPY analysis ⭐
- [ ] Test correlation checks ⭐

### Phase 5: Paper Trading (Weeks 11-12)
- [ ] Deploy to paper trading environment
- [ ] Monitor for 2-4 weeks
- [ ] Track behavioral metrics
- [ ] Optimize prompts based on performance

### Phase 6: Live Trading (Week 13+)
- [ ] Start with small positions
- [ ] Strict risk limits (0.5% per trade initially)
- [ ] Continuous monitoring
- [ ] Gradual scaling

## ⚠️ Important Notes

### Risk Disclaimers
- **Start with demo account** - Never test with real money initially
- **Use strict risk limits** - Protect your capital
- **Monitor constantly** - Especially in the first weeks
- **Past performance ≠ future results** - Markets are unpredictable
- **Trade responsibly** - Only risk what you can afford to lose

### Forex-Specific Considerations
- **DXY validation is critical** for USD pairs
- **JPY crosses indicate risk sentiment** - monitor closely
- **Correlations change** - recalculate regularly (30-day rolling)
- **Major pairs are most liquid** - start with EUR/USD, GBP/USD, USD/JPY
- **Avoid trading during major news** - high volatility and slippage

## 📖 Quick Start

1. **Review Documentation**
   ```bash
   # Read in this order:
   docs/requirements/AGENT_ARCHITECTURE_REQUIREMENTS.md
   docs/requirements/RESEARCH_REFERENCES.md
   docs/agents/01_orchestrator_agent.md
   docs/agents/04_correlation_confluence_agent.md  # Critical for Forex
   docs/mcp-tools/MCP_TOOLS_SPECIFICATION.md
   ```

2. **Set Up AWS**
   - Create AWS account
   - Configure Bedrock access
   - Set up Secrets Manager with API keys

3. **Get Pepperstone Account**
   - Open demo account
   - Get API credentials
   - Test API connection

4. **Deploy Infrastructure**
   - Use Terraform configurations (to be created)
   - Deploy Lambda functions
   - Configure Bedrock Agents

5. **Test System**
   - Backtest with historical data
   - Paper trade with live data
   - Monitor and optimize

## 🔗 Resources

### Documentation
- AWS Bedrock Agents: https://docs.aws.amazon.com/bedrock/
- DeepSeek: https://github.com/deepseek-ai/DeepSeek-V3
- Pepperstone API: https://pepperstone.com/api-docs
- nof1.ai Alpha Arena: https://nof1.ai/

### Research Papers
- Personal LLM Agents: https://arxiv.org/abs/2401.05459
- DeepSeek-V3 Paper: https://github.com/deepseek-ai/DeepSeek-V3

### Communities
- r/algotrading
- r/forex
- AWS Bedrock Discord
- AutoGen Discord

## 📝 License

This is a reference architecture for educational purposes. Use at your own risk.

---

**Version**: 1.0  
**Last Updated**: November 23, 2025  
**Status**: Architecture Complete - Ready for Implementation  
**Key Innovation**: Forex-specific correlation and confluence analysis with DXY and JPY monitoring ⭐
