# Thetanuts MCP Server

A read-only MCP (Model Context Protocol) server that exposes the Thetanuts SDK to Claude Desktop, Claude Code, Cursor, and any other MCP-compatible client.

## Who this is for

- **Builders integrating the Thetanuts SDK** — use it inside Cursor / Claude Code while writing app code. The agent can call live tools to see real return shapes, real RFQ payloads, and real encoded calldata while drafting your integration. Types can lie; the MCP shows ground truth.
- **Quants and researchers** — query market data, MM pricing, the orderbook, and historical events in plain English, no scripts to write.
- **Active traders** — explore positions, model strategies, and generate ready-to-sign transaction calldata that you paste into your own wallet.
- **New users learning the protocol** — ask questions, get real answers backed by current chain state, learn by interacting instead of reading docs.

## What this is *not*

- **Not a one-click trading bot.** The encode tools (`encode_fill_order`, `encode_request_for_quotation`, etc.) return `{to, data}` payloads — you sign and submit yourself, in your own wallet.
- **Not a substitute for the SDK** if you're shipping a production app. Use this *alongside* the SDK during development; ship the SDK in your runtime.

## Safety: read-only, no keys

This server **never holds private keys, never signs, never submits transactions**. It only:
- Reads data (RPC + Indexer + RFQ APIs)
- Encodes transaction calldata for your wallet to sign

State-changing actions are intentionally outside its scope.

## Quick install

One-click install for the most common MCP clients. Each pre-fills the config — no JSON editing required.

| Client | Action |
|---|---|
| **Cursor** | [![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](cursor://anysphere.cursor-deeplink/mcp/install?name=thetanuts&config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsIkBzaGF3bmNoZWUvdGhldGFudXRzLW1jcC1zZXJ2ZXIiXX0%3D) |
| **VS Code** | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](vscode:mcp/install?%7B%22name%22%3A%22thetanuts%22%2C%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40shawnchee%2Fthetanuts-mcp-server%22%5D%7D) |
| **Claude Code** | `claude mcp add thetanuts -- npx -y @shawnchee/thetanuts-mcp-server` |
| **Claude Desktop / Cline / Continue / others** | See the [manual JSON snippet](#manual-installation-claude-desktop--others) below. |

After installing, restart your client. A `thetanuts` indicator should appear once the server connects (~5 seconds on first run while npx fetches the package).

### Available Tools

#### Indexer API Tools (OptionBook data)
| Tool | Description |
|------|-------------|
| `get_stats` | Protocol statistics (from Indexer API) |
| `get_user_positions` | User's option positions (from Indexer API) |
| `get_user_history` | User's trade history (from Indexer API) |
| `get_referrer_stats` | Aggregated stats for a referrer address (book side, from Indexer API) |
| `get_factory_referrer_stats` | Referrer stats scoped to factory/RFQ side (RFQs, referral IDs, protocol stats) |
| `get_fees` | Accumulated referrer fees for a specific token on OptionBook |
| `get_all_claimable_fees` | All claimable fees across every collateral token for a referrer |

#### State/RFQ API Tools
| Tool | Description |
|------|-------------|
| `get_rfq` | Get specific RFQ by ID (from State/RFQ API) |
| `get_user_rfqs` | User's RFQ history (from State/RFQ API) |

#### Market & Orders
| Tool | Description |
|------|-------------|
| `get_market_data` | Current prices for BTC, ETH, SOL, etc. |
| `get_market_prices` | All supported asset prices |
| `fetch_orders` | All orders from the orderbook |
| `filter_orders` | Filter orders by asset/type/expiry |

#### Token & Option Data
| Tool | Description |
|------|-------------|
| `get_token_balance` | Token balance for address |
| `get_token_allowance` | Token allowance for spender |
| `get_token_info` | Token decimals and symbol |
| `get_option_info` | Option contract details |
| `calculate_option_payout` | Calculate payout for an option at settlement price |
| `preview_fill_order` | Preview order fill (dry-run) |

#### Utilities
| Tool | Description |
|------|-------------|
| `calculate_payout` | Calculate payoff for structures |
| `convert_decimals` | Convert to/from chain decimals |
| `get_order_fill_events` | Historical fill events |
| `get_chain_config` | Chain contracts and tokens |

#### MM Pricing
| Tool | Description |
|------|-------------|
| `get_mm_all_pricing` | Get all MM-adjusted pricing for an asset |
| `get_mm_ticker_pricing` | Get MM pricing for a specific ticker |
| `get_mm_position_pricing` | Get position-aware MM pricing with collateral cost |
| `get_mm_spread_pricing` | Get MM pricing for a two-leg spread |
| `get_mm_condor_pricing` | Get MM pricing for a four-leg condor |
| `get_mm_butterfly_pricing` | Get MM pricing for a three-leg butterfly |

#### RFQ Quotation Tools
| Tool | Description |
|------|-------------|
| `get_quotation` | Get detailed quotation info by ID including parameters and current state |
| `get_quotation_count` | Get total number of quotations created |
| `get_user_offers` | Get all RFQ offers made by a user |
| `get_user_options` | Get all options held by a user |
| `encode_settle_quotation` | Encode settlement transaction for an RFQ after reveal phase (returns tx data) |
| `encode_settle_quotation_early` | Encode early settlement to accept a specific offer before offer period ends |
| `encode_cancel_quotation` | Encode cancellation transaction for an RFQ (requester only) |
| `encode_cancel_offer` | Encode offer cancellation transaction (offeror only) |

#### RFQ Builder Tools
| Tool | Description |
|------|-------------|
| `build_rfq_request` | Build RFQ request parameters for vanilla PUT or CALL option |
| `build_spread_rfq` | Build RFQ request for a two-leg spread (put spread or call spread) |
| `build_butterfly_rfq` | Build RFQ request for a three-leg butterfly |
| `build_condor_rfq` | Build RFQ request for a four-leg condor (all calls or all puts) |
| `build_iron_condor_rfq` | Build RFQ request for a four-leg iron condor (put spread + call spread) |
| `build_physical_option_rfq` | Build RFQ request for physical-settled vanilla PUT or CALL |
| `encode_request_for_quotation` | Encode RFQ creation transaction from built request parameters |

#### Calculation Tools
| Tool | Description |
|------|-------------|
| `calculate_num_contracts` | Calculate number of contracts from trade amount |
| `calculate_collateral_required` | Calculate collateral required for a position in USDC |
| `calculate_premium_per_contract` | Calculate premium per contract in USD from MM price |
| `calculate_reserve_price` | Calculate total reserve price (minimum acceptable premium) |
| `calculate_delivery_amount` | Calculate delivery amount for physical options |
| `calculate_protocol_fee` | Calculate protocol fee for an RFQ trade |

#### Validation Tools
| Tool | Description |
|------|-------------|
| `validate_butterfly` | Validate butterfly option strike configuration (3 strikes with equal wing widths) |
| `validate_condor` | Validate condor option strike configuration (4 strikes with equal spread widths) |
| `validate_iron_condor` | Validate iron condor strike configuration (put spread below, call spread above) |
| `validate_ranger` | Validate ranger (range) option strike configuration (2 strikes) |

#### Chain Configuration Tools
| Tool | Description |
|------|-------------|
| `get_chain_config_by_id` | Get full chain configuration by chain ID |
| `get_token_config_by_id` | Get token configuration (address, decimals) by chain ID and symbol |
| `get_option_implementation_info` | Get all option implementation addresses and deployment status |
| `generate_example_keypair` | Generate an example ECDH keypair (for demonstration only) |

#### Option Query Tools
| Tool | Description |
|------|-------------|
| `get_full_option_info` | Get comprehensive option information including all strikes, positions, and settlement status |
| `get_position_info` | Get position information for buyer or seller of an option |
| `parse_ticker` | Parse an option ticker string (e.g., "ETH-16FEB26-1800-P") into its components |
| `build_ticker` | Build an option ticker string from components |

#### Event Query Tools
| Tool | Description |
|------|-------------|
| `get_option_created_events` | Get historical option creation events |
| `get_quotation_requested_events` | Get historical RFQ quotation requested events |
| `get_quotation_settled_events` | Get historical RFQ quotation settled events |
| `get_position_closed_events` | Get historical position closed events for a specific option contract |

#### Encoding Tools (Transaction Builders)
| Tool | Description |
|------|-------------|
| `encode_fill_order` | Encode a transaction to fill an order from the orderbook |
| `encode_approve` | Encode a token approval transaction |

**Note:** Encoding tools return transaction data for wallet signing - they do NOT execute transactions.

## Manual installation (Claude Desktop / others)

If your client isn't covered by [Quick install](#quick-install) above, paste the following JSON into your client's MCP config file. No clone or local build needed — `npx` fetches the package on first run.

> **Note:** This is a community build published as `@shawnchee/thetanuts-mcp-server`. An official `@thetanuts-finance/mcp-server` may supersede it in the future.

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "thetanuts": {
      "command": "npx",
      "args": ["-y", "@shawnchee/thetanuts-mcp-server"],
      "env": {
        "THETANUTS_RPC_URL": "https://mainnet.base.org"
      }
    }
  }
}
```

Restart Claude Desktop. A `thetanuts` indicator should appear once the server connects.

### Cline / Continue / other MCP clients

Same JSON snippet as above, placed in each client's MCP config file (typically `.mcp.json` at the workspace root, or the client's settings UI).

### Local development (contributors)

```bash
git clone https://github.com/Shawnchee/thetanuts-mcp-server.git
cd thetanuts-mcp-server
npm install
npm run dev   # Run with tsx (no build needed)
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `THETANUTS_RPC_URL` | `https://mainnet.base.org` | RPC endpoint |

## Examples

### Get Market Data
```
Tool: get_market_data
Result: { prices: { BTC: 95000, ETH: 3200, ... } }
```

### Filter ETH Calls
```
Tool: filter_orders
Args: { asset: "ETH", type: "call" }
Result: { count: 5, orders: [...] }
```

### Get MM Pricing
```
Tool: get_mm_ticker_pricing
Args: { ticker: "ETH-28FEB26-2800-C" }
Result: {
  ticker: "ETH-28FEB26-2800-C",
  rawBidPrice: 0.0245,
  rawAskPrice: 0.0255,
  feeAdjustedBid: 0.0241,
  feeAdjustedAsk: 0.0259,
  strike: 2800,
  expiry: 1740700800,
  isCall: true,
  byCollateral: { ... }
}
```

### Get RFQ Quotation
```
Tool: get_quotation
Args: { quotationId: "744" }
Result: {
  quotationId: "744",
  requester: "0x...",
  status: "SETTLED",
  parameters: { ... },
  offers: [ ... ]
}
```

### Encode Settlement Transaction
```
Tool: encode_settle_quotation
Args: { quotationId: "744" }
Result: {
  to: "0x...",           // OptionFactory address
  data: "0x...",         // Encoded calldata
  description: "Settle quotation 744"
}
```

**Note:** Use the returned `to` and `data` with your wallet to sign and send the transaction.

## License

MIT
