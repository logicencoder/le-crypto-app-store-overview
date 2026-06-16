# Logic Encoder Crypto App Store

A production software store where buyers discover Logic Encoder tools, pay with **USDC or ETH on Ethereum**, and receive files automatically after on-chain confirmation. The experience is built for **Telegram Mini App** users first, with a full **browser storefront** for the same checkout flow. Operators run the business from a single **admin dashboard** — orders, warehouse, wallet, chain activity, live logs, and configuration — without touching raw server files for day-to-day work.

**Made by [Logic Encoder](https://logicencoder.com)**

---

## What you can do with it

### As a buyer

- Browse a **product catalogue** (names, versions, prices, status badges) in a mobile-first shop UI.
- Add items to a **cart**, review the basket, and start checkout in one flow.
- Pay in **USDC (ERC-20)** or **ETH**, depending on what the shop has enabled.
- See a **payment screen** with order ID, exact amount, wallet address, and a **QR code** (EIP-681) for external wallets.
- Pay via **MetaMask in the browser** (network switch + token transfer) when a wallet extension is present.
- Submit a **manual transaction hash** if you paid from an exchange or hardware wallet — the backend picks it up for verification.
- Watch **live order status** over a WebSocket (pending, verifying, paying, confirming blocks, paid, delivered, expired, failed).
- See a **countdown** while the order is pending payment.
- **Cancel** a pending order and return to the store (before payment is detected on-chain).
- After delivery in the browser, download purchased files via **time-limited download links** tied to your browser session.

### As an operator

- Log into the **admin dashboard** with username and password (multi-user, role-based).
- Monitor every order from creation through delivery, filter by status, paginate large queues, and open **per-order audit trails**.
- Manage the **product warehouse**: add, edit, enable/disable, or delete items; import from WordPress; switch catalogue source; verify ZIP files on disk.
- Inspect **wallet balances** (ETH + shop token), incoming payments, and outgoing sends; initiate token sends from the dashboard when permitted.
- Follow **live operational logs** (order events, block ticks, watchdog status) in a terminal-style view.
- Review **system health**: uptime, memory, order status distribution, revenue rollups, and stack summary.
- Use the **chain tab** for recent incoming payments and outgoing sends alongside current block and watchdog state.
- Run **read-only SQL** against the analytics database for support and debugging.
- Tune **runtime settings** (accepted currencies, ETH rate for test pricing, confirmation depth, download limits, bot links) without redeploying code.
- Manage **admin users** (owner): create accounts, assign roles, disable users, reset passwords.

---

## Buyer storefront

The storefront is a single-page shop designed for thumb navigation and dark-theme readability.

**Catalogue and cart.** Products load from the active catalogue API. Each card shows name, price, version, and status. Buyers tap **Add** to build a basket; the bottom bar shows item count and total USD. The cart view lists line items with remove actions before checkout.

**Checkout.** Checkout creates an order on the backend with **server-validated prices** when a product catalogue file is present — client-submitted prices are not trusted in that mode. Each order receives a unique payment amount (derived from the order session) so multiple buyers paying the same wallet can still be matched deterministically on-chain.

**Payment methods.**

| Method | Behaviour |
|--------|-----------|
| **USDC** | ERC-20 transfer to the shop wallet; amount shown in token decimals with contract-aware QR encoding. |
| **ETH** | Native transfer when ETH checkout is enabled; conversion uses a configurable ETH/USD rate (used on testnets and when ETH is offered). |
| **MetaMask** | In-browser wallet connect, chain switch, and `transfer()` call; transaction hash is forwarded to the backend as a backup to the chain watcher. |
| **Manual TXID** | Buyer pastes a 66-character `0x` hash; order moves to manual verification and the watchdog validates the transfer. |

**Status and confirmations.** While payment is in flight, the UI shows human-readable status text (watching, verifying, paying, confirming). During block confirmations, a **step progress** UI advances with real block numbers from the backend WebSocket — not a fake timer. Terminal states include delivered, payment failed, expired, and cancelled.

**Web vs Telegram.** In Telegram, validated `initData` proves the buyer identity. In a normal browser, a persistent **web session** binds the order to that tab so download links and status APIs cannot be fetched by someone who only guesses an order ID.

---

## Order lifecycle

Orders move through explicit states that operators and buyers both see:

| Status | Meaning |
|--------|---------|
| **PENDING** | Order created; waiting for payment before expiry. |
| **MANUAL_VERIFY** | Buyer submitted a TXID; backend is checking it on-chain. |
| **PAYING** | Transfer detected; waiting for confirmations. |
| **PAID** | Payment confirmed; delivery starting. |
| **DELIVERED** | Files sent (Telegram and/or web download tokens). |
| **EXPIRED** | Payment window closed without valid payment. |
| **PAYMENT_FAILED** | On-chain receipt failed or amount mismatch. |
| **CANCELLED** | Buyer cancelled while still pending. |
| **ADMIN_CANCELLED** | Operator cancelled from the dashboard. |

Late payments after expiry are flagged for operator attention rather than silently dropped. Amount mismatches against the expected wei value are rejected and logged.

---

## Delivery

**Telegram.** After `DELIVERED`, the bot sends documents to the buyer’s chat. Delivery is guarded so the same order/file pair is not sent twice.

**Web downloads.** For browser orders, the backend issues **UUID download tokens** per file with expiry and a use limit (configurable). Tokens are only returned to clients that prove the original web session.

**Files on disk.** Products reference ZIP files in a managed downloads area. A registry tracks SHA-256 hashes and on-disk presence so the warehouse UI can show whether each product’s file exists and matches expectations.

---

## Admin dashboard

Eight primary areas, available on desktop and mobile (bottom navigation on small screens).

### Orders

- **Stats strip** at the top: totals for revenue-related counts across active payment states (visible on the Orders tab).
- **Filter chips** for every major status (All, Pending, Paying, Manual verify, Paid, Delivered, Expired, Failed, Cancelled, Admin cancel) with live counts.
- **Paginated order list** with user, items, status badge, transaction link, and timestamps.
- **Expandable audit trail** per order: timestamped actions (created, paying, paid, delivered, etc.) with detail text and explorer links for addresses and transaction hashes.
- **Admin cancel** for orders that need operator intervention.
- **Real-time updates** via WebSocket — new orders, patches, and stats refresh without full page reload.

### Warehouse

- **Source toggle**: serve products from **WordPress API** or a **local warehouse** table.
- **Product table**: ID, name, price, version, file ID, on-disk ZIP indicator, size, hash check, status (live, beta, unstable, inactive).
- **CRUD actions**: add item form, edit, enable/disable, delete (ZIP files remain on disk).
- **Import from WordPress** to sync catalogue into the local warehouse.
- **Export `products.json`** on the server for price-validation mode at checkout.
- **Shop & item deep links** for Telegram bot sharing (copy-friendly URLs per product).
- **File Checker**: scan downloads folder, list registry vs disk, re-scan on demand.
- **Debug file registry** (collapsible): raw SHA-256 table for support.

### Wallet

- **Balance cards**: ETH balance, configured token balance (e.g. USDC), contract address, decimals, network label.
- **Incoming transactions**: orders with on-chain payments — order ID, user, USD, tokens, status, explorer link, time.
- **Outgoing sends** history from operator-initiated transfers.
- **Send form** (role-gated): recipient address, amount, note — signs and broadcasts ERC-20 transfer when wallet key is configured.

### Logs

- **Live terminal** streaming backend events: order lifecycle, watchdog blocks, wallet events, system messages.
- **Level filters** (ALL, GOOD, WARN, BAD, blocks, etc.) and autoscroll control.
- **WebSocket connection indicator** in the top bar.

### System

- **Metric cards**: uptime, memory, order counts by status, audit log volume, revenue-style rollups.
- **Status distribution** chart for quick visual balance of queue health.
- **Architecture flow diagram** (desktop) showing how storefront, backend, chain watcher, and delivery connect — conceptual, for onboarding new operators.
- **Stack info** panel listing runtime technologies.

### Chain

- **Status cards**: network, current block, watchdog online/offline, token symbol, wallet address (truncated), explorer links.
- **Incoming payments** table with pagination.
- **Outgoing sends** table with pagination.
- **Live chain event log** mirroring payment-related WebSocket events.

### DB Browser

- **Table list** with row counts.
- **Read-only SQL** console: `SELECT` / `SHOW` / `DESCRIBE` only — mutations are rejected.
- Quick-query buttons for common support queries.
- Paginated result grid.

### Settings

- **Privacy**: mask IP addresses in audit/terminal views (panel and logs separately).
- **Terminal display**: max lines, block-event visibility, autoscroll default.
- **Orders**: fetch limit for bulk load, optional row background tint.
- **Admin users** (owner): list users, roles (owner, admin, operator, viewer), enable/disable, reset password, add user form.
- **Session**: logout control.
- **Pagination default** shared across Orders, DB, Wallet, and Chain tabs.
- **Warehouse section defaults**: which collapsible panels start open (file registry, shop links, file checker).
- **Backend config**: runtime-editable payment and delivery settings (accepted currencies, ETH rate, confirmation count, basket size cap, download max uses, bot link) with save feedback.

---

## Access control and security (product-facing)

The admin surface is built for a small team with different responsibilities:

| Role | Typical use |
|------|-------------|
| **Owner** | Full access including user management and database browser. |
| **Admin** | Orders, warehouse write, wallet read/send, settings, files, chain, audit. |
| **Operator** | Orders (read + cancel), warehouse read, system/chain/audit read — no wallet send or settings write. |
| **Viewer** | Read-only orders, system, and chain. |

**Session login** replaces shared static API keys for daily use. Sessions expire after twelve hours. Failed login attempts trigger a temporary lockout. Optional legacy API key mode exists for break-glass automation but is disabled by default.

**Buyer-facing APIs** require a web session token for sensitive actions (download tokens, cancel, manual TXID) so order IDs alone are insufficient.

**Checkout integrity:** Telegram purchases validate signed `initData`. When a server-side product catalogue is loaded, prices and file IDs come from the server, not the client cart.

**Download tokens** are single-purpose, expiring, and usage-limited.

**Debug “force paid”** endpoint is hidden unless explicitly enabled in server configuration and restricted to the owner role — not part of normal operations.

---

## WordPress catalogue integration

WordPress remains the marketing and catalogue editor for many deployments:

- **Webhook** on product changes triggers backend refresh.
- **REST import** pulls products into the local warehouse on demand.
- **Runtime switch** chooses whether buyers see WordPress-sourced or warehouse-sourced products without code changes.

WordPress owns presentation and editorial content; this backend owns **orders, payments, chain matching, and delivery**.

---

## Tech stack

| Layer | Technologies |
|-------|----------------|
| Backend | Node.js, Fastify |
| Database | DuckDB (`analytics.db` — orders, audit, warehouse, sessions, file registry) |
| Chain | ethers v6, JSON-RPC (ERC-20 Transfer logs + native ETH), configurable confirmations |
| Realtime | WebSocket hubs for admin logs and buyer order status |
| Bot | Telegraf (notifications + file delivery) |
| Storefront | Single-page HTML/JS (Telegram WebApp + browser); MetaMask; QR (EIP-681) |
| Admin UI | Single-page HTML dashboard with role-aware UI |
| Integrations | WordPress REST + webhook; optional Cloudflare tunnel for public HTTPS |

---

## Related repositories

| Repository | Role |
|------------|------|
| [le-crypto-app-store](https://github.com/logicencoder/le-crypto-app-store) | Private application code |
| [le-crypto-app-store-overview](https://github.com/logicencoder/le-crypto-app-store-overview) | This product overview |
| [le-shop-plugin-overview](https://github.com/logicencoder/le-shop-plugin-overview) | WordPress catalogue companion |

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
