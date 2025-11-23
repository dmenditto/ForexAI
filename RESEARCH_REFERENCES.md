# Multi-Agent AI Trading Systems - Research References

## Overview

This document compiles key references and insights from leading platforms, frameworks, and research on multi-agent AI systems, with specific focus on trading applications and AWS deployment.

---

## 1. nof1.ai Alpha Arena - Real-World AI Trading Validation

**URL**: https://nof1.ai/

### Key Findings

**What It Is**:
- First benchmark designed to measure AI's investing abilities
- Multiple LLMs compete with $10,000 real money in real markets
- 2-week competitions to maximize trading profits
- Currently focused on US equities (TSLA, NVDA, MSFT, AMZN, GOOGL, PLTR)

**Competition Formats**:
1. **New Baseline**: Trades equities, ingests news/sentiment data, wider action space
2. **Monk Mode**: ~50% shorter system prompt, more opinionated guardrails on trading thresholds and risk management ⭐ **Most Successful Approach**
3. **Situational Awareness**: Awareness of competition & leaderboard + other model's positions
4. **Max Leverage**: Tests risk management with maximum leverage on every trade

**Current Performance (Season 1.5)**:
- mystery-model: $9,807 (best performer)
- kimi-k2-thinking: $9,524
- gpt-5.1: $9,652
- gemini-3-pro: $9,355
- claude-sonnet-4-5: $8,699
- deepseek-chat-v3.1: $8,617
- qwen3-max: $8,237
- grok-4: $5,989

**Critical Insights for Forex Trading**:
1. **Prompt Engineering Matters**: Small prompt changes significantly impact performance
2. **Shorter is Better**: "Monk Mode" with ~50% shorter prompts shows competitive results
3. **Opinionated Guardrails Work**: Strict risk management rules in prompts improve outcomes
4. **Autonomous Trading is Proven**: AI can successfully trade with real money autonomously
5. **Models Must**:
   - Generate alpha (find opportunities)
   - Size trades (position sizing)
   - Time trades (entry/exit timing)
   - Manage risk
   - Operate completely autonomously

**Relevance to Your System**:
- Validates multi-agent approach for trading
- Proves AI can handle real money in real markets
- "Monk Mode" approach directly applicable to your agent prompts
- Demonstrates importance of risk management guardrails

---

## 2. AWS Bedrock Agents - Multi-Agent Collaboration

**URL**: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-multi-agent-collaboration.html

### Key Features

**Multi-Agent Collaboration**:
- Multiple Amazon Bedrock Agents collaboratively plan and solve complex tasks
- Hierarchical collaboration model: Supervisor + Collaborator agents
- Synchronous real-time responses
- Each agent can have its own tools, action groups, knowledge bases, and guardrails

**Architecture Pattern**:
```
Supervisor Agent (Orchestrator)
    ├── Collaborator Agent 1 (Specialist)
    ├── Collaborator Agent 2 (Specialist)
    └── Collaborator Agent 3 (Specialist)
```

**Example Use Case** (Mortgage Assistant):
- **Supervisor**: Routes questions to appropriate collaborator
- **Collaborator 1**: Handles existing mortgages
- **Collaborator 2**: Handles new mortgage applications
- **Collaborator 3**: Handles general questions

**Key Capabilities**:
- Break down complex tasks
- Assign tasks to domain specialists
- Work in parallel
- Leverage each agent's strengths
- Centralized planning and orchestration

**Design Requirements**:
- Clearly designate role and responsibilities for each agent
- Minimize overlapping responsibilities
- Use natural language to describe roles
- Supervisor must understand structure and role of each collaborator

**Relevance to Your System**:
- Perfect match for your Orchestrator + 5 specialist agents architecture
- Native AWS support for multi-agent collaboration
- Built-in guardrails and security
- Scalable and production-ready

---

## 3. DeepSeek-V3 - State-of-the-Art Open Source LLM

**URL**: https://github.com/deepseek-ai/DeepSeek-V3

### Model Specifications

**Architecture**:
- 671B total parameters
- 37B activated per token (Mixture-of-Experts)
- 128K context window
- Multi-head Latent Attention (MLA)
- DeepSeekMoE architecture

**Performance Highlights**:
- Outperforms most open-source models
- Comparable to leading closed-source models (GPT-4, Claude)
- Excellent on math and code tasks
- Strong reasoning capabilities

**Key Benchmarks**:
- MMLU: 87.1% (vs GPT-4: ~86%)
- HumanEval (Code): 65.2% Pass@1
- MATH: 61.6% EM
- GSM8K: 89.3% EM

**Training Efficiency**:
- Only 2.788M H800 GPU hours for full training
- FP8 mixed precision training
- Remarkably stable training (no rollbacks)
- Cost-effective at scale

**Post-Training Innovation**:
- Knowledge distillation from DeepSeek-R1 (reasoning model)
- Incorporates verification and reflection patterns
- Improved reasoning performance
- Controlled output style and length

**Relevance to Your System**:
- Cost-effective LLM option for trading agents
- Strong reasoning capabilities for complex trading decisions
- 128K context window for extensive market data
- Open-source with commercial license
- Can be self-hosted or used via API

---

## 4. Microsoft AutoGen - Multi-Agent Framework

**URL**: https://github.com/microsoft/autogen

### Framework Overview

**What It Is**:
- Framework for creating multi-agent AI applications
- Agents can act autonomously or work alongside humans
- Layered and extensible design
- Cross-language support (Python and .NET)

**Architecture Layers**:
1. **Core API**: Message passing, event-driven agents, distributed runtime
2. **AgentChat API**: Simpler, opinionated API for rapid prototyping
3. **Extensions API**: First- and third-party extensions (LLM clients, code execution)

**Key Features**:
- **Multi-Agent Orchestration**: Single agent, multi-agent, hierarchical, sequential
- **MCP Server Support**: Native integration with Model Context Protocol
- **Human-in-the-Loop**: Agents can collaborate with humans
- **Streaming**: Token-by-token streaming for better UX
- **AutoGen Studio**: No-code GUI for building multi-agent applications

**Multi-Agent Pattern Example**:
```python
# Supervisor with specialist agents
math_agent = AssistantAgent("math_expert", ...)
chemistry_agent = AssistantAgent("chemistry_expert", ...)

supervisor = AssistantAgent(
    "assistant",
    tools=[math_agent_tool, chemistry_agent_tool],
    max_tool_iterations=10
)
```

**Developer Tools**:
- **AutoGen Studio**: No-code GUI for prototyping
- **AutoGen Bench**: Benchmarking suite for agent performance
- **Magentic-One**: State-of-the-art multi-agent team example

**Relevance to Your System**:
- Alternative to AWS Bedrock for agent orchestration
- Strong MCP support (aligns with your architecture)
- Proven multi-agent patterns
- Active community and ecosystem
- Can be deployed on AWS Lambda/ECS

---

## 5. LangGraph - Controllable Cognitive Architecture

**URL**: https://www.langchain.com/langgraph

### Framework Capabilities

**What It Is**:
- Framework for building stateful, multi-actor applications with LLMs
- Flexible control flows: single agent, multi-agent, hierarchical, sequential
- Built-in statefulness and memory
- Human-in-the-loop support

**Key Features**:

**1. Flexible Agent Workflows**:
- Diverse control flows (single, multi-agent, hierarchical)
- Low-level primitives for full customization
- Easy-to-add moderation and quality loops

**2. Human-Agent Collaboration**:
- Built-in statefulness for seamless collaboration
- Agents can write drafts for review
- Await approval before acting
- Time-travel to roll back and correct course

**3. Persistent Context**:
- Built-in memory stores conversation histories
- Maintains context over time
- Enables personalized interactions across sessions

**4. First-Class Streaming**:
- Token-by-token streaming
- Show agent reasoning in real-time
- Better user experience

**5. LangGraph Platform**:
- Deploy agents at scale
- Fault-tolerant scalability
- Dynamic APIs for agent experience
- Integrated with LangSmith for monitoring

**Production Features**:
- Horizontally-scaling servers
- Task queues
- Built-in persistence
- Intelligent caching
- Automated retries
- Long-term memory across sessions
- Background jobs for long-running tasks

**Relevance to Your System**:
- Alternative orchestration framework
- Strong state management (important for trading)
- Built-in memory for context retention
- Production-ready deployment options
- Can integrate with AWS services

---

## 6. Personal LLM Agents Research

**Paper**: "Personal LLM Agents: Insights and Survey about the Capability, Efficiency and Security"  
**URL**: https://arxiv.org/abs/2401.05459  
**Authors**: Yuanchun Li et al. (24 authors from multiple institutions)

### Key Concepts

**What Are Personal LLM Agents**:
- LLM-based agents deeply integrated with personal data and devices
- Used for personal assistance
- Envisioned as major software paradigm for end-users

**Key Components**:
1. **User Intent Understanding**: Semantic understanding of user goals
2. **Task Planning**: Breaking down complex tasks into steps
3. **Tool Using**: Ability to use external tools and APIs
4. **Personal Data Management**: Secure handling of user data

**Architecture Considerations**:
- Capability: What can the agent do?
- Efficiency: How fast and resource-efficient?
- Security: How to protect user data and prevent misuse?

**Challenges Addressed**:
- Intelligent task execution
- Efficient resource usage
- Secure data handling
- Privacy preservation

**Relevance to Your System**:
- Framework for thinking about agent capabilities
- Security and privacy considerations for trading data
- Efficiency optimization strategies
- Tool integration patterns

---

## Comparative Analysis

### Framework Comparison

| Feature | AWS Bedrock | AutoGen | LangGraph |
|---------|-------------|---------|-----------|
| **Deployment** | AWS Native | Self-hosted/Cloud | Self-hosted/Cloud |
| **Multi-Agent** | ✅ Native | ✅ Native | ✅ Native |
| **MCP Support** | ✅ Via Lambda | ✅ Native | ⚠️ Via Integration |
| **Statefulness** | ✅ Built-in | ✅ Built-in | ✅ Built-in |
| **Streaming** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Human-in-Loop** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Cost** | Pay-per-use | Open source | Open source + Platform |
| **Scalability** | ✅ Auto-scale | Manual | Platform auto-scale |
| **Monitoring** | CloudWatch | Custom | LangSmith |
| **Learning Curve** | Medium | Medium | Medium-High |

### LLM Comparison for Trading

| Model | Cost | Reasoning | Speed | Context | Best For |
|-------|------|-----------|-------|---------|----------|
| **DeepSeek-V3** | Low | ⭐⭐⭐⭐⭐ | Fast | 128K | Cost-effective, strong reasoning |
| **GPT-4** | High | ⭐⭐⭐⭐⭐ | Medium | 128K | Proven, reliable |
| **Claude Sonnet** | Medium | ⭐⭐⭐⭐ | Fast | 200K | Long context, fast |
| **Gemini Pro** | Medium | ⭐⭐⭐⭐ | Fast | 1M | Massive context |

---

## Recommended Architecture for Your Forex System

Based on all research, here's the recommended approach:

### 1. Platform: AWS Bedrock Agents
**Why**:
- Native multi-agent collaboration
- Seamless AWS integration (Lambda, DynamoDB, Secrets Manager)
- Built-in security and guardrails
- Auto-scaling and production-ready
- Pay-per-use pricing

### 2. LLM: DeepSeek-V3 (via API Gateway + Lambda)
**Why**:
- Cost-effective ($0.14/M input tokens, $0.28/M output tokens)
- Strong reasoning capabilities (proven in nof1.ai)
- 128K context window (sufficient for market data)
- Can be self-hosted if needed
- Comparable performance to GPT-4

### 3. Prompt Strategy: "Monk Mode" Approach
**Why**:
- Proven effective in nof1.ai Alpha Arena
- Shorter prompts (~50% of baseline)
- Opinionated guardrails on risk management
- Clearer decision boundaries
- Less token usage = lower cost

### 4. MCP Servers: AWS Lambda Functions
**Why**:
- Serverless, auto-scaling
- Easy integration with Bedrock Agents
- Cost-effective (pay per invocation)
- Native AWS service integration
- OpenAPI schema support

### 5. Alternative: AutoGen + DeepSeek (if more control needed)
**Why**:
- More flexibility in agent design
- Native MCP support
- Can deploy on AWS ECS/Lambda
- Open source (no vendor lock-in)
- Active community

---

## Implementation Priorities

### Phase 1: Validate with nof1.ai Insights
1. Design "Monk Mode" style prompts (concise, opinionated)
2. Define strict risk management guardrails
3. Set up A/B testing framework for prompt variations

### Phase 2: Build on AWS Bedrock
1. Create supervisor agent (Orchestrator)
2. Create 5 collaborator agents (Technical, Fundamental, Sentiment, Risk, Execution)
3. Implement MCP servers as Lambda functions
4. Connect to Pepperstone API

### Phase 3: Test and Iterate
1. Backtest with historical data
2. Paper trade with live data
3. Monitor behavioral metrics (not just P&L)
4. Iterate on prompts based on performance

### Phase 4: Production Deployment
1. Start with small positions
2. Strict risk limits
3. Continuous monitoring
4. Gradual scaling

---

## Key Takeaways

1. **AI Trading is Proven**: nof1.ai validates that AI can trade autonomously with real money
2. **Prompt Engineering is Critical**: "Monk Mode" approach shows shorter, opinionated prompts work better
3. **Multi-Agent is the Way**: AWS Bedrock, AutoGen, and LangGraph all support hierarchical multi-agent systems
4. **DeepSeek is Competitive**: Cost-effective alternative to GPT-4 with strong reasoning
5. **Guardrails are Essential**: Strict risk management rules in prompts improve outcomes
6. **AWS is Production-Ready**: Bedrock Agents provide native multi-agent support with enterprise features

---

## Additional Resources

### Documentation
- AWS Bedrock Agents: https://docs.aws.amazon.com/bedrock/
- DeepSeek API: https://platform.deepseek.com/
- AutoGen Docs: https://microsoft.github.io/autogen/
- LangGraph Docs: https://langchain-ai.github.io/langgraph/

### Communities
- nof1.ai Discord: (check their website)
- AWS Bedrock Discord: (via AWS)
- AutoGen Discord: https://aka.ms/autogen-discord
- LangChain Discord: https://discord.gg/langchain

### Papers
- Personal LLM Agents: https://arxiv.org/abs/2401.05459
- DeepSeek-V3 Paper: https://github.com/deepseek-ai/DeepSeek-V3

---

**Last Updated**: November 23, 2025  
**Research Focus**: Multi-agent AI trading systems, AWS deployment, LLM selection
