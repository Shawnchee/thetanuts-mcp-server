# Thetanuts MCP Server

[![npm version](https://img.shields.io/npm/v/@thetanuts-finance/mcp-server.svg?style=flat-square)](https://www.npmjs.com/package/@thetanuts-finance/mcp-server)
[![npm downloads](https://img.shields.io/npm/dt/@thetanuts-finance/mcp-server.svg?style=flat-square&label=downloads)](https://www.npmjs.com/package/@thetanuts-finance/mcp-server)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg?style=flat-square)](./LICENSE)
[![Thetanuts SDK](https://img.shields.io/npm/v/@thetanuts-finance/thetanuts-client.svg?style=flat-square&label=SDK)](https://www.npmjs.com/package/@thetanuts-finance/thetanuts-client)
[![GitHub](https://img.shields.io/badge/source-GitHub-181717?style=flat-square&logo=github)](https://github.com/Shawnchee/thetanuts-mcp-server)

A read-only MCP (Model Context Protocol) server that exposes the Thetanuts SDK to Claude Code, Cursor, Antigravity, and any other MCP-compatible client.

> ⚠️ **Don't `npm i` this.** The sidebar's `npm i @thetanuts-finance/mcp-server` snippet is misleading — this package is meant to be **spawned by your MCP client via `npx`**, not imported as a library. Use one of the install paths below. (Why? See [Install methods compared](#install-methods-compared) at the bottom.)

## Install in your MCP client

[![Add to Cursor](https://img.shields.io/badge/Add_to-Cursor-000000?style=for-the-badge&logo=cursor&logoColor=white)](cursor://anysphere.cursor-deeplink/mcp/install?name=thetanuts&config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsIkB0aGV0YW51dHMtZmluYW5jZS9tY3Atc2VydmVyIl19)
[![Install in VS Code](https://img.shields.io/badge/Install_in-VS_Code-0098FF?style=for-the-badge&logo=visualstudiocode&logoColor=white)](vscode:mcp/install?%7B%22name%22%3A%22thetanuts%22%2C%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40thetanuts-finance%2Fmcp-server%22%5D%7D)

**Cursor** and **VS Code** are the only clients with one-click install URL schemes — click a badge to open the IDE and confirm. Other clients (Claude Code, Antigravity) need a single command or JSON paste — see below.

> Heads-up: on **npmjs.com**, custom URL schemes (`cursor://`, `vscode:`) may be stripped — click the badges from this README on **GitHub** for the one-click install to fire correctly.

<details>
<summary><b>Claude Code</b> — one command (or commit <code>.mcp.json</code> for your team)</summary>

```bash
claude mcp add --transport stdio thetanuts -- npx -y @thetanuts-finance/mcp-server
```

For teams: commit a `.mcp.json` at your repo root and every dev gets it automatically. See [team setup](#building-a-team-app-on-the-sdk-drop-this-in-your-repo-root) below.
</details>

<details>
<summary><b>Antigravity</b> — Agent Panel → ⋯ → MCP Servers → raw config</summary>

Open Agent Panel → ⋯ menu → MCP Servers → Manage MCP Servers → View raw config. Paste this JSON. Config file: `~/.gemini/antigravity/mcp_config.json` (macOS/Linux).

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

After installing, restart your client. A `thetanuts` indicator should appear once the server connects (~5 s on first run while npx fetches the package).

## ⚡ Lazy install — paste this to your agent

Don't want to figure out which install method to use? Paste the prompt below into your AI agent (Claude Code, Cursor's agent, Antigravity). It'll detect your client, install the server, verify the connection, and suggest example questions.

📦 **Package:** [`@thetanuts-finance/mcp-server`](https://www.npmjs.com/package/@thetanuts-finance/mcp-server) on npm

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

> The agent won't be able to install the server *into the same session it's running in* without a restart. Expect it to install, then ask you to restart the client. After restart, it'll have the new tools live.

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

### Building a team app on the SDK? Drop this in your repo root

If you're a team building on `@thetanuts-finance/thetanuts-client`, commit a `.mcp.json` at the root of your project. Every teammate using **Claude Code** in the repo gets the MCP server automatically — no per-developer setup, no instructions to forward.

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

On first run inside the repo, Claude Code prompts each user to approve the workspace's MCP servers; after that, it loads automatically. This is the recommended setup for builders shipping on the SDK.

## How to use it

Once the server is connected, just talk to your agent in plain English. The agent picks the right tool from the 79 available based on your request.

### For traders / researchers (no code needed)

> *"What's the current ETH price?"*
> → agent calls `get_market_data`, returns live Base mainnet price.

> *"Show me all ETH calls expiring in February with MM-adjusted prices."*
> → agent chains `filter_orders` + `get_mm_ticker_pricing`, returns a table.

> *"My address is 0xabc… — what option positions do I have?"*
> → agent calls `get_user_positions`, returns your live book.

> *"Encode a settlement transaction for RFQ 744. I'll sign it in MetaMask."*
> → agent calls `encode_settle_quotation`, returns `{to, data}`. Paste into your wallet.

> *"What lending opportunities are there for ETH right now?"*
> → agent calls `get_lending_opportunities`, returns unfilled limit orders with computed APR.

### For builders shipping on the SDK

The MCP server is your **dev-time companion** — it doesn't replace the SDK in your app's runtime. The pattern:

```bash
# In your app's repo (runtime dependency — actually ships with your app)
npm i @thetanuts-finance/thetanuts-client

# In your IDE (dev-time helper — agent uses live tools while you code)
# Cursor:    click the badge above
# Claude Code: claude mcp add --transport stdio thetanuts -- npx -y @thetanuts-finance/mcp-server
# Or commit .mcp.json to your repo (see "team app" section above)
```

Then while coding in Cursor / Claude Code, ask things like:

> *"Write me a function that takes a strike + expiry and submits an RFQ for an ETH put. Use the SDK. Verify the calldata it produces matches `encode_request_for_quotation` from the MCP server."*

The agent will:
1. Call MCP tools (`get_market_data`, `build_rfq_request`, `encode_request_for_quotation`) to see *real* return shapes from live mainnet.
2. Write SDK code in your app that imports from `@thetanuts-finance/thetanuts-client`.
3. Compare the SDK call's encoded output against the MCP's reference `{to, data}` to catch param mismatches *before* you sign anything.

This is the "ground truth while writing code" workflow — the SDK ships in your app, the MCP ships nowhere.

## Available Tools

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

#### Loan Module (non-liquidatable lending)
| Tool | Description |
|------|-------------|
| `get_lending_opportunities` | Available lending opportunities with computed APR (lender-side discovery) |
| `get_loan_request` | On-chain state of a loan by quotation ID |
| `get_user_loans` | All loans (active + historical) for a borrower address |
| `get_loan_option_info` | Detailed info about a deployed loan option contract |
| `is_loan_option_itm` | Check if a loan option is ITM at current TWAP |
| `get_loan_pricing` | Raw Deribit-style pricing for ETH/BTC loan options (cached 30s) |
| `get_loan_strike_options` | Available strike options grouped by expiry, with APR and effective borrowing cost |
| `calculate_loan` | Pure-calc loan costs (option premium, borrowing fee, protocol fee, promo detection) |
| `encode_request_loan` | Encode `requestLoan` transaction (borrower side) — returns `{to, data}` |
| `encode_loan_accept_offer` | Encode `acceptOffer` transaction (borrower picks an MM offer early) |
| `encode_loan_cancel` | Encode `cancelLoan` transaction (borrower cancels pending request) |

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

## Manual installation (other MCP clients)

If your client isn't covered by [Install in your MCP client](#install-in-your-mcp-client) above, paste the following JSON into your client's MCP config file. No clone or local build needed — `npx` fetches the package on first run.

### Antigravity

In Antigravity: **Agent Panel** → **⋯** menu → **MCP Servers** → **Manage MCP Servers** → **View raw config**. Config file: `~/.gemini/antigravity/mcp_config.json` (macOS/Linux) or `C:\Users\<you>\.gemini\antigravity\mcp_config.json` (Windows).

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

> Antigravity does not yet support per-workspace `.mcp.json` files (user-global only for now). If your team is on Antigravity, each developer installs once at the user level.

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

## Install methods compared

| Method | Setup | Config | Auto-updates | When to use |
|---|---|---|---|---|
| **`npx -y` (recommended)** | None | `command: "npx", args: ["-y", "@thetanuts-finance/mcp-server"]` | ✅ within `^0.2.3` (≥0.2.3, <0.3.0) | Default. What every install path on this README uses. |
| `npm i -g` (global) | `npm i -g @thetanuts-finance/mcp-server` | `command: "thetanuts-mcp"` | ❌ manual `npm update -g` | Offline-heavy environments only. |
| `npm i` (local) | mkdir + npm init + install | absolute path to `node_modules/.bin/thetanuts-mcp` | ❌ manual | Contributors hacking on the server itself. Not for normal use. |

**TL;DR:** ignore the `npm i ...` snippet npm shows in its sidebar. It's a default that doesn't apply to MCP servers. Use `npx`.

## Related

- **[Thetanuts SDK on npm](https://www.npmjs.com/package/@thetanuts-finance/thetanuts-client)** — install in your app's repo to actually ship code that calls Thetanuts.
- **[Thetanuts SDK on GitHub](https://github.com/Thetanuts-Finance/thetanuts-sdk)** — source, examples, deeper docs.
- **[Model Context Protocol](https://modelcontextprotocol.io/)** — the open standard this server implements.
- **[This server on GitHub](https://github.com/Shawnchee/thetanuts-mcp-server)** — issues, PRs, contributions welcome.

## License

MIT
