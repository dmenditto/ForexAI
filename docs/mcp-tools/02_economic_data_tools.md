# Economic Data MCP Tools

**Server**: Economic Data Server  
**Lambda**: `forex-economic-data`  
**Purpose**: Economic indicators, calendar events, central bank data  
**Data Source**: FRED API (free), Trading Economics (paid)

---

## Tool 1: get_economic_calendar

Get upcoming economic events

**Parameters**:
- `start_date` (string, required)
- `end_date` (string, required)
- `countries` (array, optional): ["US", "EU", "GB", "JP"]
- `importance` (string, optional): "LOW", "MEDIUM", "HIGH"

**Returns**:
```json
{
  "events": [
    {
      "date": "2025-11-25T13:30:00Z",
      "country": "US",
      "event": "GDP Growth Rate",
      "importance": "HIGH",
      "forecast": "2.8%",
      "previous": "2.5%",
      "currency": "USD"
    }
  ]
}
```

**Used By**: Fundamental Analysis Agent

---

## Tool 2: get_interest_rates

Get current central bank interest rates

**Parameters**:
- `countries` (array, required): ["US", "EU", "GB", "JP"]

**Returns**:
```json
{
  "rates": [
    {
      "country": "US",
      "currency": "USD",
      "rate": 5.25,
      "last_change": "2025-09-20",
      "next_meeting": "2025-12-15"
    }
  ]
}
```

**Used By**: Fundamental Analysis Agent

---

## Tool 3: get_economic_indicator

Get historical economic indicator data

**Parameters**:
- `country` (string, required)
- `indicator` (string, required): "GDP", "CPI", "PPI", "UNEMPLOYMENT", "TRADE_BALANCE"
- `period` (string, required)

**Returns**:
```json
{
  "country": "US",
  "indicator": "CPI",
  "unit": "percent",
  "data": [
    {"date": "2025-10-01", "value": 3.2}
  ]
}
```

**Used By**: Fundamental Analysis Agent

---

## Tool 4: get_rate_differential

Calculate interest rate differential

**Parameters**:
- `base_currency` (string, required)
- `quote_currency` (string, required)

**Returns**:
```json
{
  "pair": "EUR/USD",
  "base_rate": 4.00,
  "quote_rate": 5.25,
  "differential": -1.25,
  "advantage": "USD"
}
```

**Used By**: Fundamental Analysis Agent
