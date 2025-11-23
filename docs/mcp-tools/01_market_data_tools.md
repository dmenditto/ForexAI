# Market Data MCP Tools

**Server**: Market Data Server  
**Lambda**: `forex-market-data`  
**Purpose**: Real-time and historical Forex price data  
**Data Source**: Pepperstone API (primary), Alpha Vantage (backup)

---

## Tool 1: get_live_price

Get current bid/ask price for a currency pair

**Parameters**:
- `pair` (string, required): Currency pair (e.g., "EUR/USD")
- `timeframe` (string, optional): Timeframe (M1, M5, H1, H4, D1)

**Returns**:
```json
{
  "pair": "EUR/USD",
  "bid": 1.0850,
  "ask": 1.0852,
  "spread": 0.0002,
  "timestamp": "2025-11-23T10:30:00Z"
}
```

**Used By**: Technical Analysis Agent, Correlation Agent, Execution Agent

---

## Tool 2: get_historical_data

Get historical OHLCV data

**Parameters**:
- `pair` (string, required): Currency pair
- `start_date` (string, required): ISO date
- `end_date` (string, required): ISO date
- `timeframe` (string, required): M1, M5, M15, M30, H1, H4, D1

**Returns**:
```json
{
  "pair": "EUR/USD",
  "timeframe": "H1",
  "data": [
    {
      "timestamp": "2025-11-23T10:00:00Z",
      "open": 1.0850,
      "high": 1.0865,
      "low": 1.0845,
      "close": 1.0860,
      "volume": 15000
    }
  ]
}
```

**Used By**: Technical Analysis Agent

---

## Tool 3: get_spread

Get current bid-ask spread

**Parameters**:
- `pair` (string, required): Currency pair

**Returns**:
```json
{
  "pair": "EUR/USD",
  "spread_pips": 2.0,
  "spread_percent": 0.018,
  "timestamp": "2025-11-23T10:30:00Z"
}
```

**Used By**: Execution Agent, Risk Management Agent

---

## Tool 4: get_multiple_pairs

Get live prices for multiple pairs at once

**Parameters**:
- `pairs` (array, required): List of currency pairs

**Returns**:
```json
{
  "prices": [
    {"pair": "EUR/USD", "bid": 1.0850, "ask": 1.0852},
    {"pair": "GBP/USD", "bid": 1.2650, "ask": 1.2652}
  ],
  "timestamp": "2025-11-23T10:30:00Z"
}
```

**Used By**: Correlation & Confluence Agent
