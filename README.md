# Thetanuts MCP Server

[![npm version](https://img.shields.io/npm/v/@thetanuts-finance/mcp-server.svg?style=flat-square)](https://www.npmjs.com/package/@thetanuts-finance/mcp-server)
[![npm downloads](https://img.shields.io/npm/dt/@thetanuts-finance/mcp-server.svg?style=flat-square&label=downloads)](https://www.npmjs.com/package/@thetanuts-finance/mcp-server)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg?style=flat-square)](./LICENSE)
[![Thetanuts SDK](https://img.shields.io/npm/v/@thetanuts-finance/thetanuts-client.svg?style=flat-square&label=SDK)](https://www.npmjs.com/package/@thetanuts-finance/thetanuts-client)
[![GitHub](https://img.shields.io/badge/source-GitHub-181717?style=flat-square&logo=github)](https://github.com/Shawnchee/thetanuts-mcp-server)

A read-only MCP (Model Context Protocol) server that exposes the Thetanuts SDK to MCP-compatible clients, including Claude Code, Cursor, VS Code, and Antigravity.

> **Note on installation.** This package is intended to be invoked by an MCP client via `npx`; it is not a runtime library. The `npm install` snippet shown in npm's sidebar does not apply here. See [Install methods compared](#install-methods-compared) for the full rationale.

## Install in your MCP client

[![Add to Cursor](https://img.shields.io/badge/Add_to-Cursor-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=thetanuts&config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsIkB0aGV0YW51dHMtZmluYW5jZS9tY3Atc2VydmVyIl19)
[![Install in VS Code](https://img.shields.io/badge/Install_in-VS_Code-0098FF?style=for-the-badge&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect?url=vscode%3Amcp%2Finstall%3F%7B%22name%22%3A%22thetanuts%22%2C%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40thetanuts-finance%2Fmcp-server%22%5D%7D)

**Cursor** and **VS Code** support one-click installation; click either badge to register the server with the IDE. The badges link to the vendor's HTTPS deep-link redirector, which then hands off to the IDE. Other clients require a single command or JSON snippet, documented below.

<details>
<summary><b>Claude Code</b> — single command (or commit <code>.mcp.json</code> for your team)</summary>

```bash
claude mcp add --transport stdio thetanuts -- npx -y @thetanuts-finance/mcp-server
```

For team installations, commit a `.mcp.json` file to your repository root and every developer receives the configuration automatically. See [Team setup](#team-setup) below.
</details>

<details>
<summary><b>Antigravity</b> — Agent Panel → ⋯ → MCP Servers → raw config</summary>

Open the **Agent Panel → ⋯ menu → MCP Servers → Manage MCP Servers → View raw config** and paste the configuration below. The config file is located at `~/.gemini/antigravity/mcp_config.json` on macOS and Linux.

```json
{
  "mcpServers": {
    "thetanuts": {
      "command": "npx",
      "args": ["-y", "@thetanuts-finance/mcp-server"]
    }
  }
}
```
</details>

After installation, restart your client. A `thetanuts` indicator will appear once the server has connected, typically within five seconds on first run while `npx` fetches the package.

## Quick install via an AI agent

For users who prefer not to select an installation method manually, the prompt below can be pasted into any MCP-compatible AI agent (Claude Code, Cursor's agent, Antigravity). The agent will detect the host client, install the server, verify the connection, and suggest example queries.

**Package:** [`@thetanuts-finance/mcp-server`](https://www.npmjs.com/package/@thetanuts-finance/mcp-server) on npm

````text
Install the Thetanuts MCP server for me.

Source of truth: https://www.npmjs.com/package/@thetanuts-finance/mcp-server
Package name: @thetanuts-finance/mcp-server

Pick the right install method for the client you're running in:

- Claude Code: run `claude mcp add --transport stdio thetanuts -- npx -y @thetanuts-finance/mcp-server`
- Cursor / VS Code: tell me to open https://github.com/Shawnchee/thetanuts-mcp-server and click the "Add to Cursor" or "Install in VS Code" badge — that's a one-click install.
- Antigravity: add this JSON to `~/.gemini/antigravity/mcp_config.json`:
  {"mcpServers":{"thetanuts":{"command":"npx","args":["-y","@thetanuts-finance/mcp-server"]}}}

After installing:
1. Tell me if I need to restart the client.
2. Verify the server is reachable by calling the `get_market_data` tool. Confirm live BTC and ETH prices come back.
3. Give me a 1-line summary of what I can now do, plus 3 example questions to try (covering market data, position queries, and RFQ encoding).

Don't ask me for my wallet address or any keys — this server is read-only and doesn't sign anything.
````

> An agent cannot install the server into the same session in which it is currently running. Expect it to perform the installation, then request a client restart. After the restart, the new tools become available.

## Use cases

- **Application developers integrating the Thetanuts SDK.** Use the MCP server alongside Cursor or Claude Code while writing application code. The agent invokes live tools to inspect actual return shapes, RFQ payloads, and encoded calldata while you draft your integration. Type definitions can be incomplete or out of date; the MCP server reflects current chain and API state.
- **Quants and researchers.** Query market data, MM pricing, the orderbook, and historical events in natural language, without writing scripts.
- **Active traders.** Explore positions, model strategies, and generate ready-to-sign transaction calldata for use with an external wallet.
- **New users.** Learn the protocol interactively by asking questions backed by current chain state.

## Scope and limitations

- **Not a trading bot.** Encoding tools (`encode_fill_order`, `encode_request_for_quotation`, and similar) return `{to, data}` payloads. Signing and submission are performed by the user in their own wallet.
- **Not a runtime substitute for the SDK.** Production applications should depend on `@thetanuts-finance/thetanuts-client` directly. The MCP server is a development-time aid, not a deployment dependency.

## Security model

The server **does not hold private keys, sign transactions, or submit transactions**. Its scope is limited to:

- Reading data via RPC, the Indexer API, and the RFQ API
- Encoding transaction calldata for client-side signing

State-changing actions are intentionally outside the server's scope.

## Team setup

Teams building on `@thetanuts-finance/thetanuts-client` are recommended to commit a `.mcp.json` file at the repository root. Every developer using **Claude Code** within the repository will receive the MCP server configuration automatically, with no per-developer setup.

```json
{
  "mcpServers": {
    "thetanuts": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@thetanuts-finance/mcp-server"]
    }
  }
}
```

On first invocation within the repository, Claude Code prompts each user to approve the workspace's MCP servers; subsequent loads are automatic. This is the recommended configuration for teams shipping on the SDK.

## Usage

Once the server is connected, interact with the agent in natural language. The agent selects appropriate tools from the 79 available based on the request.

### Traders and researchers

> *"What is the current ETH price?"*
> The agent calls `get_market_data` and returns a live price from Base mainnet.

> *"Show me all ETH calls expiring in February with MM-adjusted prices."*
> The agent chains `filter_orders` and `get_mm_ticker_pricing` to return a table of results.

> *"My address is 0xabc… — what option positions do I hold?"*
> The agent calls `get_user_positions` and returns the user's live position book.

> *"Encode a settlement transaction for RFQ 744. I will sign it in MetaMask."*
> The agent calls `encode_settle_quotation` and returns `{to, data}` for wallet submission.

> *"What lending opportunities exist for ETH right now?"*
> The agent calls `get_lending_opportunities`, returning unfilled limit orders with computed APR.

### Application developers

The MCP server is a **development-time companion**; it does not replace the SDK in an application's runtime. The recommended pattern:

```bash
# In your application repository (runtime dependency)
npm install @thetanuts-finance/thetanuts-client

# In your IDE (development-time helper)
# Cursor:      click the install badge above
# Claude Code: claude mcp add --transport stdio thetanuts -- npx -y @thetanuts-finance/mcp-server
# Or commit .mcp.json to the repo root (see Team setup above)
```

While coding in Cursor or Claude Code, ask the agent:

> *"Write a function that takes a strike and expiry and submits an RFQ for an ETH put. Use the SDK. Verify the calldata it produces matches `encode_request_for_quotation` from the MCP server."*

The agent will:
1. Call MCP tools (`get_market_data`, `build_rfq_request`, `encode_request_for_quotation`) to observe actual return shapes from live mainnet.
2. Write SDK code in the application that imports from `@thetanuts-finance/thetanuts-client`.
3. Compare the SDK call's encoded output against the MCP's reference `{to, data}` to detect parameter mismatches before any transaction is signed.

The result is a "ground truth while writing code" workflow: the SDK ships with your application; the MCP server does not.

## Available tools

#### Indexer API tools (OptionBook data)
| Tool | Description |
|------|-------------|
| `get_stats` | Protocol statistics (from Indexer API) |
| `get_user_positions` | User's option positions (from Indexer API) |
| `get_user_history` | User's trade history (from Indexer API) |
| `get_referrer_stats` | Aggregated stats for a referrer address (book side, from Indexer API) |
| `get_factory_referrer_stats` | Referrer stats scoped to factory/RFQ side (RFQs, referral IDs, protocol stats) |
| `get_fees` | Accumulated referrer fees for a specific token on OptionBook |
| `get_all_claimable_fees` | All claimable fees across every collateral token for a referrer |

#### State/RFQ API tools
| Tool | Description |
|------|-------------|
| `get_rfq` | Get specific RFQ by ID (from State/RFQ API) |
| `get_user_rfqs` | User's RFQ history (from State/RFQ API) |

#### Market and orders
| Tool | Description |
|------|-------------|
| `get_market_data` | Current prices for BTC, ETH, SOL, etc. |
| `get_market_prices` | All supported asset prices |
| `fetch_orders` | All orders from the orderbook |
| `filter_orders` | Filter orders by asset, type, or expiry |

#### Token and option data
| Tool | Description |
|------|-------------|
| `get_token_balance` | Token balance for an address |
| `get_token_allowance` | Token allowance for a spender |
| `get_token_info` | Token decimals and symbol |
| `get_option_info` | Option contract details |
| `calculate_option_payout` | Calculate payout for an option at a given settlement price |
| `preview_fill_order` | Preview an order fill (dry run) |

#### Utilities
| Tool | Description |
|------|-------------|
| `calculate_payout` | Calculate payoff for structures |
| `convert_decimals` | Convert to and from chain decimals |
| `get_order_fill_events` | Historical fill events |
| `get_chain_config` | Chain contracts and tokens |

#### MM pricing
| Tool | Description |
|------|-------------|
| `get_mm_all_pricing` | All MM-adjusted pricing for an asset |
| `get_mm_ticker_pricing` | MM pricing for a specific ticker |
| `get_mm_position_pricing` | Position-aware MM pricing with collateral cost |
| `get_mm_spread_pricing` | MM pricing for a two-leg spread |
| `get_mm_condor_pricing` | MM pricing for a four-leg condor |
| `get_mm_butterfly_pricing` | MM pricing for a three-leg butterfly |

#### RFQ quotation tools
| Tool | Description |
|------|-------------|
| `get_quotation` | Detailed quotation info by ID, including parameters and current state |
| `get_quotation_count` | Total number of quotations created |
| `get_user_offers` | All RFQ offers made by a user |
| `get_user_options` | All options held by a user |
| `encode_settle_quotation` | Encode settlement transaction for an RFQ after the reveal phase |
| `encode_settle_quotation_early` | Encode early settlement to accept a specific offer before the offer period ends |
| `encode_cancel_quotation` | Encode cancellation transaction for an RFQ (requester only) |
| `encode_cancel_offer` | Encode offer cancellation transaction (offeror only) |

#### RFQ builder tools
| Tool | Description |
|------|-------------|
| `build_rfq_request` | Build RFQ request parameters for a vanilla put or call option |
| `build_spread_rfq` | Build RFQ request for a two-leg spread (put spread or call spread) |
| `build_butterfly_rfq` | Build RFQ request for a three-leg butterfly |
| `build_condor_rfq` | Build RFQ request for a four-leg condor (all calls or all puts) |
| `build_iron_condor_rfq` | Build RFQ request for a four-leg iron condor (put spread plus call spread) |
| `build_physical_option_rfq` | Build RFQ request for a physically settled vanilla put or call |
| `encode_request_for_quotation` | Encode RFQ creation transaction from built request parameters |

#### Loan module (non-liquidatable lending)
| Tool | Description |
|------|-------------|
| `get_lending_opportunities` | Available lending opportunities with computed APR (lender-side discovery) |
| `get_loan_request` | On-chain state of a loan by quotation ID |
| `get_user_loans` | All loans (active and historical) for a borrower address |
| `get_loan_option_info` | Detailed info about a deployed loan option contract |
| `is_loan_option_itm` | Check if a loan option is ITM at the current TWAP |
| `get_loan_pricing` | Raw Deribit-style pricing for ETH and BTC loan options (cached for 30 seconds) |
| `get_loan_strike_options` | Available strike options grouped by expiry, with APR and effective borrowing cost |
| `calculate_loan` | Pure-calc loan costs (option premium, borrowing fee, protocol fee, promo detection) |
| `encode_request_loan` | Encode `requestLoan` transaction (borrower side); returns `{to, data}` |
| `encode_loan_accept_offer` | Encode `acceptOffer` transaction (borrower picks an MM offer early) |
| `encode_loan_cancel` | Encode `cancelLoan` transaction (borrower cancels a pending request) |

#### Calculation tools
| Tool | Description |
|------|-------------|
| `calculate_num_contracts` | Calculate number of contracts from a trade amount |
| `calculate_collateral_required` | Calculate collateral required for a position in USDC |
| `calculate_premium_per_contract` | Calculate premium per contract in USD from MM price |
| `calculate_reserve_price` | Calculate total reserve price (minimum acceptable premium) |
| `calculate_delivery_amount` | Calculate delivery amount for physical options |
| `calculate_protocol_fee` | Calculate protocol fee for an RFQ trade |

#### Validation tools
| Tool | Description |
|------|-------------|
| `validate_butterfly` | Validate butterfly option strike configuration (three strikes with equal wing widths) |
| `validate_condor` | Validate condor option strike configuration (four strikes with equal spread widths) |
| `validate_iron_condor` | Validate iron condor strike configuration (put spread below, call spread above) |
| `validate_ranger` | Validate ranger (range) option strike configuration (two strikes) |

#### Chain configuration tools
| Tool | Description |
|------|-------------|
| `get_chain_config_by_id` | Full chain configuration by chain ID |
| `get_token_config_by_id` | Token configuration (address, decimals) by chain ID and symbol |
| `get_option_implementation_info` | All option implementation addresses and deployment status |
| `generate_example_keypair` | Generate an example ECDH keypair (for demonstration only) |

#### Option query tools
| Tool | Description |
|------|-------------|
| `get_full_option_info` | Comprehensive option information, including all strikes, positions, and settlement status |
| `get_position_info` | Position information for the buyer or seller of an option |
| `parse_ticker` | Parse an option ticker string (for example, `ETH-16FEB26-1800-P`) into its components |
| `build_ticker` | Build an option ticker string from components |

#### Event query tools
| Tool | Description |
|------|-------------|
| `get_option_created_events` | Historical option creation events |
| `get_quotation_requested_events` | Historical RFQ quotation requested events |
| `get_quotation_settled_events` | Historical RFQ quotation settled events |
| `get_position_closed_events` | Historical position closed events for a specific option contract |

#### Encoding tools (transaction builders)
| Tool | Description |
|------|-------------|
| `encode_fill_order` | Encode a transaction to fill an order from the orderbook |
| `encode_approve` | Encode a token approval transaction |

**Note.** Encoding tools return transaction data for wallet signing; they do not execute transactions.

## Manual installation (other MCP clients)

If your client is not covered by [Install in your MCP client](#install-in-your-mcp-client) above, paste the JSON snippet below into your client's MCP configuration file. No clone or local build is required; `npx` fetches the package on first run.

### Antigravity

In Antigravity, navigate to **Agent Panel → ⋯ menu → MCP Servers → Manage MCP Servers → View raw config**. The config file is located at `~/.gemini/antigravity/mcp_config.json` on macOS and Linux, or `C:\Users\<user>\.gemini\antigravity\mcp_config.json` on Windows.

```json
{
  "mcpServers": {
    "thetanuts": {
      "command": "npx",
      "args": ["-y", "@thetanuts-finance/mcp-server"],
      "env": {
        "THETANUTS_RPC_URL": "https://mainnet.base.org"
      }
    }
  }
}
```

> Antigravity does not currently support per-workspace `.mcp.json` files; configuration is user-global only. Teams using Antigravity must install once per developer at the user level.

### Cline, Continue, and other MCP clients

The same JSON snippet shown above can be placed in each client's MCP configuration file, typically `.mcp.json` at the workspace root or via the client's settings UI.

### Local development (contributors)

```bash
git clone https://github.com/Shawnchee/thetanuts-mcp-server.git
cd thetanuts-mcp-server
npm install
npm run dev   # Run with tsx; no build step required
```

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `THETANUTS_RPC_URL` | `https://mainnet.base.org` | RPC endpoint |

## Examples

### Get market data
```
Tool: get_market_data
Result: { prices: { BTC: 95000, ETH: 3200, ... } }
```

### Filter ETH calls
```
Tool: filter_orders
Args: { asset: "ETH", type: "call" }
Result: { count: 5, orders: [...] }
```

### Get MM pricing
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

### Get RFQ quotation
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

### Encode settlement transaction
```
Tool: encode_settle_quotation
Args: { quotationId: "744" }
Result: {
  to: "0x...",           // OptionFactory address
  data: "0x...",         // Encoded calldata
  description: "Settle quotation 744"
}
```

**Note.** Use the returned `to` and `data` values with your wallet to sign and submit the transaction.

## Install methods compared

| Method | Setup | Config | Auto-updates | When to use |
|---|---|---|---|---|
| **`npx -y` (recommended)** | None | `command: "npx", args: ["-y", "@thetanuts-finance/mcp-server"]` | Yes, within `^0.2.3` (≥0.2.3, <0.3.0) | Default. The method used by every install path in this README. |
| `npm i -g` (global) | `npm i -g @thetanuts-finance/mcp-server` | `command: "thetanuts-mcp"` | No; manual `npm update -g` | Offline-heavy environments only. |
| `npm i` (local) | `mkdir`, `npm init`, then install | Absolute path to `node_modules/.bin/thetanuts-mcp` | No; manual | Contributors developing the server itself; not for normal use. |

**Summary.** Disregard the `npm i ...` snippet shown in npm's sidebar; it is a default that does not apply to MCP servers. Use `npx`.

## Related

- **[Thetanuts SDK on npm](https://www.npmjs.com/package/@thetanuts-finance/thetanuts-client)** — install in your application's repository to ship code that calls Thetanuts.
- **[Thetanuts SDK on GitHub](https://github.com/Thetanuts-Finance/thetanuts-sdk)** — source, examples, and reference documentation.
- **[Model Context Protocol](https://modelcontextprotocol.io/)** — the open standard this server implements.
- **[This server on GitHub](https://github.com/Shawnchee/thetanuts-mcp-server)** — issues, pull requests, and contributions are welcome.

## License

MIT
