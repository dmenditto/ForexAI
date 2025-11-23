# MCP Tools Specification - Complete Reference

## Overview

This document defines all MCP (Model Context Protocol) tools required for the Forex trading multi-agent system. Each tool is implemented as an AWS Lambda function.

---

## MCP Server 1: Market Data Server

### Purpose
Provide real-time and historical Forex price data from Pepperstone and backup sources.

### Tools

#### 1. get_live_price
```json
{
  "name": "get_live_price",
  "description": "Get current bid/ask price for a currency pair",
  "parameters": {
    "pair": {
      "type": "string",
      "description": "Currency pair (e.g., EUR/USD)",
      "required": true
    },
    "timeframe": {
      "type": "string",
      "description": "Timeframe (M1, M5, M15, M30, H1, H4, D1)",
      "required": false
    }
  },
  "returns": {
    "pair": "string",
    "bid": "number",
    "ask": "number",
    "spread": "number",
    "timestamp": "string (ISO 8601)"
  }
}
```

#### 2. get_historical_data
```json
{
  "name": "get_historical_data",
  "description": "Get historical OHLCV data",
  "parameters": {
    "pair": {"type": "string", "required": true},
    "start_date": {"type": "string", "required": true},
    "end_date": {"type": "string", "required": true},
    "timeframe": {"type": "string", "required": true}
  },
  "returns": {
    "pair": "string",
    "timeframe": "string",
    "data": [
      {
        "timestamp": "string",
        "open": "number",
        "high": "number",
        "low": "number",
        "close": "number",
        "volume": "number"
      }
    ]
  }
}
```

#### 3. get_spread
```json
{
  "name": "get_spread",
  "description": "Get current bid-ask spread",
  "parameters": {
    "pair": {"type": "string", "required": true}
  },
  "returns": {
    "pair": "string",
    "spread_pips": "number",
    "spread_percent": "number",
    "timestamp": "string"
  }
}
```

#### 4. get_multiple_pairs
```json
{
  "name": "get_multiple_pairs",
  "description": "Get live prices for multiple pairs at once",
  "parameters": {
    "pairs": {
      "type": "array",
      "items": "string",
      "required": true
    }
  },
  "returns": {
    "prices": [
      {"pair": "string", "bid": "number", "ask": "number"}
    ],
    "timestamp": "string"
  }
}
```

**Data Sources**: Pepperstone API (primary), Alpha Vantage (backup)

---

## MCP Server 2: Economic Data Server

### Purpose
Access economic indicators, calendar events, and central bank data.

### Tools

#### 1. get_economic_calendar
```json
{
  "name": "get_economic_calendar",
  "description": "Get upcoming economic events",
  "parameters": {
    "start_date": {"type": "string", "required": true},
    "end_date": {"type": "string", "required": true},
    "countries": {"type": "array", "items": "string", "required": false},
    "importance": {"type": "string", "enum": ["LOW", "MEDIUM", "HIGH"], "required": false}
  },
  "returns": {
    "events": [
      {
        "date": "string",
        "country": "string",
        "event": "string",
        "importance": "string",
        "forecast": "string",
        "previous": "string",
        "currency": "string"
      }
    ]
  }
}
```

#### 2. get_interest_rates
```json
{
  "name": "get_interest_rates",
  "description": "Get current central bank interest rates",
  "parameters": {
    "countries": {"type": "array", "items": "string", "required": true}
  },
  "returns": {
    "rates": [
      {
        "country": "string",
        "currency": "string",
        "rate": "number",
        "last_change": "string",
        "next_meeting": "string"
      }
    ]
  }
}
```

#### 3. get_economic_indicator
```json
{
  "name": "get_economic_indicator",
  "description": "Get historical economic indicator data",
  "parameters": {
    "country": {"type": "string", "required": true},
    "indicator": {"type": "string", "enum": ["GDP", "CPI", "PPI", "UNEMPLOYMENT", "TRADE_BALANCE"], "required": true},
    "period": {"type": "string", "required": true}
  },
  "returns": {
    "country": "string",
    "indicator": "string",
    "unit": "string",
    "data": [
      {"date": "string", "value": "number"}
    ]
  }
}
```

#### 4. get_rate_differential
```json
{
  "name": "get_rate_differential",
  "description": "Calculate interest rate differential",
  "parameters": {
    "base_currency": {"type": "string", "required": true},
    "quote_currency": {"type": "string", "required": true}
  },
  "returns": {
    "pair": "string",
    "base_rate": "number",
    "quote_rate": "number",
    "differential": "number",
    "advantage": "string"
  }
}
```

**Data Sources**: FRED API (free), Trading Economics (paid)

---

## MCP Server 3: News & Sentiment Server

### Purpose
Aggregate news, analyze sentiment, and track institutional positioning.

### Tools

#### 1. get_news
```json
{
  "name": "get_news",
  "description": "Fetch recent news articles",
  "parameters": {
    "keywords": {"type": "array", "items": "string", "required": true},
    "sources": {"type": "array", "items": "string", "required": false},
    "time_range": {
      "type": "object",
      "properties": {
        "start": "string",
        "end": "string"
      },
      "required": true
    },
    "limit": {"type": "number", "required": false}
  },
  "returns": {
    "articles": [
      {
        "title": "string",
        "source": "string",
        "url": "string",
        "published_at": "string",
        "sentiment": "number",
        "summary": "string"
      }
    ],
    "overall_sentiment": "number"
  }
}
```

#### 2. analyze_sentiment
```json
{
  "name": "analyze_sentiment",
  "description": "Perform NLP sentiment analysis on text",
  "parameters": {
    "text": {"type": "string", "required": true}
  },
  "returns": {
    "sentiment": "number",
    "confidence": "number",
    "keywords": ["string"],
    "entities": ["string"]
  }
}
```

#### 3. get_social_sentiment
```json
{
  "name": "get_social_sentiment",
  "description": "Get social media sentiment",
  "parameters": {
    "pair": {"type": "string", "required": true},
    "platforms": {"type": "array", "items": "string", "required": false},
    "time_range": {"type": "string", "required": false}
  },
  "returns": {
    "pair": "string",
    "sentiment": "number",
    "volume": "number",
    "trending_topics": ["string"],
    "bullish_percent": "number",
    "bearish_percent": "number"
  }
}
```

#### 4. get_cot_report
```json
{
  "name": "get_cot_report",
  "description": "Get Commitment of Traders positioning",
  "parameters": {
    "currency": {"type": "string", "required": true},
    "report_type": {"type": "string", "enum": ["legacy", "disaggregated"], "required": false}
  },
  "returns": {
    "currency": "string",
    "report_date": "string",
    "commercial": {
      "long": "number",
      "short": "number",
      "net": "number"
    },
    "non_commercial": {
      "long": "number",
      "short": "number",
      "net": "number"
    },
    "change_from_previous": "number"
  }
}
```

**Data Sources**: NewsAPI, CFTC COT reports, Twitter API (optional)

---

## MCP Server 4: Technical Analysis Server

### Purpose
Calculate technical indicators and detect chart patterns.

### Tools

#### 1. calculate_indicator
```json
{
  "name": "calculate_indicator",
  "description": "Calculate any technical indicator",
  "parameters": {
    "pair": {"type": "string", "required": true},
    "indicator": {"type": "string", "enum": ["RSI", "MACD", "SMA", "EMA", "BB", "ATR", "STOCH"], "required": true},
    "timeframe": {"type": "string", "required": true},
    "params": {"type": "object", "required": false}
  },
  "returns": {
    "indicator": "string",
    "current_value": "number or object",
    "values": ["number or object"],
    "signal": "string",
    "timestamp": "string"
  }
}
```

#### 2. detect_patterns
```json
{
  "name": "detect_patterns",
  "description": "Identify chart patterns",
  "parameters": {
    "pair": {"type": "string", "required": true},
    "timeframe": {"type": "string", "required": true},
    "lookback_periods": {"type": "number", "required": false}
  },
  "returns": {
    "patterns": [
      {
        "name": "string",
        "type": "string",
        "confidence": "number",
        "completion_price": "number",
        "target": "number"
      }
    ]
  }
}
```

#### 3. find_support_resistance
```json
{
  "name": "find_support_resistance",
  "description": "Identify key support and resistance levels",
  "parameters": {
    "pair": {"type": "string", "required": true},
    "timeframe": {"type": "string", "required": true},
    "lookback_days": {"type": "number", "required": false}
  },
  "returns": {
    "support": ["number"],
    "resistance": ["number"],
    "current_price": "number"
  }
}
```

#### 4. calculate_multiple_indicators
```json
{
  "name": "calculate_multiple_indicators",
  "description": "Calculate multiple indicators at once",
  "parameters": {
    "pair": {"type": "string", "required": true},
    "timeframe": {"type": "string", "required": true},
    "indicators": {"type": "array", "items": "string", "required": true}
  },
  "returns": {
    "RSI": "number",
    "MACD": {"value": "number", "signal": "number", "histogram": "number"},
    "SMA_20": "number",
    "SMA_50": "number",
    "BB": {"upper": "number", "middle": "number", "lower": "number"}
  }
}
```

**Implementation**: TA-Lib or Pandas-TA

---

## MCP Server 5: Correlation & DXY Server

### Purpose
Analyze currency correlations, DXY, and JPY strength.

### Tools

#### 1. get_dxy_data
```json
{
  "name": "get_dxy_data",
  "description": "Get US Dollar Index data",
  "parameters": {},
  "returns": {
    "current": "number",
    "change_24h": "number",
    "trend": "string",
    "key_levels": {
      "support": ["number"],
      "resistance": ["number"]
    }
  }
}
```

#### 2. get_currency_strength
```json
{
  "name": "get_currency_strength",
  "description": "Get individual currency strength index",
  "parameters": {
    "currency": {"type": "string", "required": true}
  },
  "returns": {
    "currency": "string",
    "strength": "number",
    "rank": "number",
    "change_24h": "number"
  }
}
```

#### 3. calculate_correlation
```json
{
  "name": "calculate_correlation",
  "description": "Calculate correlation between pairs",
  "parameters": {
    "pair1": {"type": "string", "required": true},
    "pair2": {"type": "string", "required": true},
    "period_days": {"type": "number", "required": false}
  },
  "returns": {
    "pair1": "string",
    "pair2": "string",
    "correlation": "number",
    "period": "number"
  }
}
```

#### 4. get_jpy_crosses
```json
{
  "name": "get_jpy_crosses",
  "description": "Get all JPY pair data for risk sentiment",
  "parameters": {},
  "returns": {
    "usd_jpy": {"value": "number", "change_24h": "number", "trend": "string"},
    "eur_jpy": {"value": "number", "change_24h": "number", "trend": "string"},
    "gbp_jpy": {"value": "number", "change_24h": "number", "trend": "string"},
    "aud_jpy": {"value": "number", "change_24h": "number", "trend": "string"},
    "sentiment": "string",
    "alignment": "string"
  }
}
```

#### 5. validate_confluence
```json
{
  "name": "validate_confluence",
  "description": "Validate signal through cross-pair confluence",
  "parameters": {
    "target_pair": {"type": "string", "required": true},
    "proposed_action": {"type": "string", "enum": ["BUY", "SELL"], "required": true}
  },
  "returns": {
    "validated": "boolean",
    "supporting_pairs": ["string"],
    "contradicting_pairs": ["string"],
    "confluence_score": "number"
  }
}
```

---

## MCP Server 6: Risk Management Server

### Purpose
Calculate risk metrics and validate trade proposals.

### Tools

#### 1. calculate_position_size
```json
{
  "name": "calculate_position_size",
  "description": "Calculate optimal position size",
  "parameters": {
    "account_balance": {"type": "number", "required": true},
    "risk_percent": {"type": "number", "required": true},
    "stop_loss_pips": {"type": "number", "required": true},
    "pair": {"type": "string", "required": true}
  },
  "returns": {
    "position_size": "number",
    "lot_size": "number",
    "risk_amount": "number",
    "pip_value": "number"
  }
}
```

#### 2. calculate_var
```json
{
  "name": "calculate_var",
  "description": "Calculate Value at Risk",
  "parameters": {
    "portfolio": {"type": "object", "required": true},
    "confidence_level": {"type": "number", "required": true},
    "time_horizon_days": {"type": "number", "required": false}
  },
  "returns": {
    "var_95": "number",
    "var_percent": "number",
    "method": "string"
  }
}
```

#### 3. check_exposure
```json
{
  "name": "check_exposure",
  "description": "Check current currency exposure",
  "parameters": {
    "positions": {"type": "array", "required": true}
  },
  "returns": {
    "exposure": {
      "USD": "number",
      "EUR": "number",
      "GBP": "number",
      "JPY": "number"
    },
    "warnings": ["string"]
  }
}
```

#### 4. validate_trade
```json
{
  "name": "validate_trade",
  "description": "Validate if trade meets risk parameters",
  "parameters": {
    "proposed_trade": {"type": "object", "required": true},
    "account_balance": {"type": "number", "required": true},
    "current_exposure": {"type": "object", "required": true},
    "risk_parameters": {"type": "object", "required": true}
  },
  "returns": {
    "approved": "boolean",
    "risk_amount": "number",
    "risk_percent": "number",
    "new_exposure": "object",
    "warnings": ["string"],
    "recommendations": ["string"]
  }
}
```

---

## MCP Server 7: Broker Integration Server (Pepperstone)

### Purpose
Execute trades and manage positions via Pepperstone API.

### Tools

#### 1. place_order
```json
{
  "name": "place_order",
  "description": "Place a trading order via Pepperstone",
  "parameters": {
    "pair": {"type": "string", "required": true},
    "side": {"type": "string", "enum": ["BUY", "SELL"], "required": true},
    "size": {"type": "number", "required": true},
    "order_type": {"type": "string", "enum": ["MARKET", "LIMIT", "STOP"], "required": true},
    "price": {"type": "number", "required": false},
    "stop_loss": {"type": "number", "required": false},
    "take_profit": {"type": "number", "required": false}
  },
  "returns": {
    "order_id": "string",
    "status": "string",
    "executed_price": "number",
    "slippage": "number",
    "timestamp": "string",
    "error_message": "string"
  }
}
```

#### 2. get_open_positions
```json
{
  "name": "get_open_positions",
  "description": "Get all open positions",
  "parameters": {},
  "returns": {
    "positions": [
      {
        "position_id": "string",
        "pair": "string",
        "side": "string",
        "size": "number",
        "entry_price": "number",
        "current_price": "number",
        "unrealized_pnl": "number",
        "stop_loss": "number",
        "take_profit": "number",
        "opened_at": "string"
      }
    ]
  }
}
```

#### 3. close_position
```json
{
  "name": "close_position",
  "description": "Close an open position",
  "parameters": {
    "position_id": {"type": "string", "required": true}
  },
  "returns": {
    "position_id": "string",
    "closed_price": "number",
    "realized_pnl": "number",
    "timestamp": "string"
  }
}
```

#### 4. get_account_info
```json
{
  "name": "get_account_info",
  "description": "Get account balance and margin info",
  "parameters": {},
  "returns": {
    "balance": "number",
    "equity": "number",
    "margin_used": "number",
    "free_margin": "number",
    "margin_level": "number",
    "open_positions": "number"
  }
}
```

#### 5. modify_position
```json
{
  "name": "modify_position",
  "description": "Modify stop loss or take profit",
  "parameters": {
    "position_id": {"type": "string", "required": true},
    "stop_loss": {"type": "number", "required": false},
    "take_profit": {"type": "number", "required": false}
  },
  "returns": {
    "position_id": "string",
    "updated": "boolean",
    "new_stop_loss": "number",
    "new_take_profit": "number"
  }
}
```

**Integration**: Uses existing Pepperstone API implementation

---

## Implementation Notes

### AWS Lambda Configuration

Each MCP server is deployed as a Lambda function:

```yaml
Runtime: python3.11
Timeout: 30 seconds
Memory: 512 MB
Environment Variables:
  - PEPPERSTONE_API_KEY
  - PEPPERSTONE_API_SECRET
  - PEPPERSTONE_ACCOUNT_ID
  - NEWS_API_KEY
  - FRED_API_KEY
  - etc.
```

### Error Handling

All tools return errors in standard format:

```json
{
  "error": true,
  "error_code": "string",
  "message": "string",
  "retry_after": "number"
}
```

### Rate Limiting

- Market Data: 100 requests/minute
- Economic Data: 60 requests/hour
- News: 100 requests/day (free tier)
- Broker API: 50 requests/minute

### Caching Strategy

- Market data: Cache for 1-5 seconds
- Economic calendar: Cache for 1 hour
- News: Cache for 15 minutes
- Technical indicators: Cache for timeframe period

---

**Version**: 1.0  
**Last Updated**: November 23, 2025  
**Total Tools**: 35 across 7 MCP servers
