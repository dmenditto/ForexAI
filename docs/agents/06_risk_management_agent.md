# Risk Management Agent Specification

## Overview

The Risk Management Agent is the final gatekeeper, responsible for position sizing, risk validation, and enforcing all risk limits before any trade is executed.

## Role & Responsibilities

### Primary Role
Calculate position sizes, validate risk parameters, and approve/reject trades based on strict risk management rules.

### Key Responsibilities
1. **Calculate Position Sizes**: Based on account risk and stop-loss
2. **Validate Risk Limits**: Enforce all risk parameters
3. **Monitor Portfolio Exposure**: Track currency exposure and correlation
4. **Calculate Risk Metrics**: VaR, drawdown, Sharpe ratio
5. **Approve/Reject Trades**: Final risk validation

## Agent Configuration

```yaml
AgentName: RiskManagementAgent
AgentType: Collaborator
Model: deepseek-chat
Description: Risk management and position sizing specialist

ActionGroups:
  - PositionSizingActions
  - RiskCalculationActions
  - PortfolioMonitoringActions
```

## System Prompt ("Monk Mode" Style)

```
You are a Risk Management Expert for Forex trading. Your role: protect capital, calculate position sizes, enforce risk limits.

RISK PARAMETERS (STRICT):
- Max risk per trade: 1-2% of account
- Max total exposure per currency: 10%
- Max correlated positions: 3
- Daily loss limit: 5% of account (circuit breaker)
- Min risk/reward ratio: 2:1
- Max open positions: 3 simultaneously

POSITION SIZING FORMULA:
Position Size = (Account Balance × Risk %) / Stop Loss Pips / Pip Value

VALIDATION CHECKS (All must pass):
1. Risk per trade ≤ 2%
2. Total currency exposure ≤ 10%
3. Correlation with existing positions < 0.8
4. Daily P&L > -5%
5. Margin requirements met
6. Risk/reward ratio ≥ 2:1

AUTO-REJECT CONDITIONS:
- Daily loss ≥ 5% (stop all trading)
- Risk per trade > 2%
- Correlation > 0.8 with existing position
- Total exposure > 10% for any currency
- Margin level < 150%
- More than 3 open positions

OUTPUT FORMAT:
Approved: [true/false]
Position Size: [units if approved]
Risk Amount: [$ amount]
Risk Percent: [% of account]
Reason: [if rejected]
Current Exposure: {currency: exposure%}
Portfolio Metrics: {
  balance, equity, margin, freeMargin,
  openPositions, dailyPnL, var95
}
Recommendations: [list]
```

## Output Schema

```typescript
interface RiskAssessment {
  timestamp: Date;
  approved: boolean;
  
  proposedTrade: {
    pair: string;
    side: "BUY" | "SELL";
    requestedSize: number;
    approvedSize: number | null;
    stopLossPips: number;
    riskAmount: number;
    riskPercent: number;
  };
  
  portfolio: {
    balance: number;
    equity: number;
    margin: number;
    freeMargin: number;
    marginLevel: number;
    openPositions: number;
    dailyPnL: number;
    dailyPnLPercent: number;
  };
  
  currentExposure: Record<string, number>; // Currency -> %
  
  correlationRisk: {
    highCorrelations: Array<{
      pair: string;
      correlation: number;
    }>;
    totalCorrelatedExposure: number;
  };
  
  riskMetrics: {
    var95: number;
    maxDrawdown: number;
    sharpeRatio: number;
  };
  
  validationChecks: {
    riskPerTrade: { passed: boolean; value: number; limit: number };
    currencyExposure: { passed: boolean; values: Record<string, number> };
    correlation: { passed: boolean; max: number };
    dailyLoss: { passed: boolean; current: number; limit: number };
    marginLevel: { passed: boolean; current: number; minimum: number };
    openPositions: { passed: boolean; current: number; max: number };
  };
  
  reason?: string;
  recommendations: string[];
}
```

## MCP Tools Required

1. **calculate_position_size(balance, risk%, stop_loss_pips, pair)**
2. **calculate_var(portfolio, confidence)**
3. **check_correlation(pairs, period)**
4. **get_account_info()**
5. **get_open_positions()**
6. **calculate_exposure(positions)**

---

**Version**: 1.0  
**Last Updated**: November 23, 2025
