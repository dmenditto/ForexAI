# ForexAI - Multi-Agent Forex Trading System

Autonomous Forex trading system using AWS Bedrock Agents, DeepSeek LLM, and Pepperstone broker integration.

## Architecture

7-agent hierarchical system with specialized analysis:

- **Orchestrator Agent** - Master coordinator and decision maker
- **Technical Analysis Agent** - Charts, indicators, patterns
- **Fundamental Analysis Agent** - Economic data, interest rates
- **Correlation & Confluence Agent** - DXY, JPY, currency correlations
- **Sentiment Analysis Agent** - News, social media, COT reports
- **Risk Management Agent** - Position sizing, exposure validation
- **Execution Agent** - Pepperstone trade execution

## Technology Stack

- **Agent Platform**: AWS Bedrock Agents
- **LLM**: DeepSeek (via API Gateway + Lambda proxy)
- **MCP Servers**: 7 AWS Lambda functions, 35 tools
- **Data Storage**: DynamoDB, Timestream
- **Broker**: Pepperstone API

## Documentation

Complete specifications in `/docs`:

- **[AGENT_ARCHITECTURE_REQUIREMENTS.md](docs/requirements/AGENT_ARCHITECTURE_REQUIREMENTS.md)** - Complete system architecture
- **[RESEARCH_REFERENCES.md](docs/requirements/RESEARCH_REFERENCES.md)** - nof1.ai Alpha Arena research
- **[agents/](docs/agents/)** - Individual agent specifications (7 files)
- **[mcp-tools/](docs/mcp-tools/)** - MCP tool specifications (7 servers, 35 tools)

## Key Features

### Forex-Specific Analysis
- **DXY monitoring** - Validates USD pair signals against US Dollar Index
- **JPY crosses** - Risk-on/risk-off sentiment detection
- **Currency correlations** - 30-day rolling correlation matrix
- **Cross-pair confluence** - Signal validation across related pairs

### "Monk Mode" Design Philosophy
Inspired by nof1.ai Alpha Arena research:
- Concise, opinionated prompts with strict guardrails
- Real-money validated approach
- Behavioral metrics tracking
- A/B testing framework for prompt optimization

### Risk Management
- 1-2% risk per trade
- Max 10% exposure per currency
- Auto-reject on correlation >0.8
- Mandatory stop-loss on every trade
- Daily loss limit (5%)

## Implementation Status

**Current Phase**: Architecture Complete - Ready for Implementation

### Next Steps
1. Deploy AWS infrastructure (Lambda, DynamoDB, Timestream)
2. Implement 7 MCP server Lambda functions
3. Configure Bedrock Agents with DeepSeek integration
4. Integration testing
5. Backtesting with historical data
6. Paper trading (2-4 weeks)
7. Live trading with small capital

## Cost Estimate

- **Development**: ~$100-180/month
- **Production**: ~$430-950/month

## License

ISC
