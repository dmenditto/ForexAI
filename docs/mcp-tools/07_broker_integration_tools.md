# Broker Integration MCP Tools (Pepperstone)

**Server**: Broker Integration Server  
**Lambda**: `forex-broker-integration`  
**Purpose**: Trade execution and position management  
**Integration**: Existing Pepperstone API implementation

## Tools

1. **place_order** - Place trading order (market, limit, stop)
2. **get_open_positions** - List all open positions
3. **close_position** - Close an open position
4. **get_account_info** - Account balance and margin
5. **modify_position** - Modify stop-loss or take-profit

**Used By**: Execution Agent, Risk Management Agent
