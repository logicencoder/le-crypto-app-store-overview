# Logic Encoder Crypto App Store

**A real-time Telegram commerce platform — crypto in, files out, automatically — with a full operator control panel.**

Logic Encoder Crypto App Store (LE CAS) is a purpose-built system for selling digital products **inside Telegram** and in the browser. Buyers browse a Mini App shop, pay **USDC or ETH on Ethereum**, watch payment progress **live on screen**, and receive ZIP files **without anyone clicking Send**. Operators see the same events in real time: new orders, chain confirmations, deliveries, and failures — in the **admin dashboard**, in the **live log terminal**, and in **Telegram operator alerts**.

No payment gateway. No manual inbox. One automated loop: **shop → chain → confirm → deliver → notify**.

**Made by [Logic Encoder](https://logicencoder.com)**

---

## What the platform does

| Layer | Role |
|-------|------|
| **Telegram Mini App** | Mobile-first storefront inside the bot — catalogue, cart, checkout, live status |
| **Chain watcher** | Embedded in the backend — matches transfers to orders within seconds |
| **Delivery engine** | Sends files to buyer chat (Telegram) or issues expiring download links (web) |
| **Admin dashboard** | Real-time control plane — orders, warehouse, wallet, logs, chain, DB, settings |
| **WordPress (LE Shop)** | Optional catalogue editor — publish in WP, buyers see updates in Telegram |

Checkout, payment matching, and file delivery **never** go through WordPress. The Node backend owns money and files.

---

## Real-time automation — what happens when someone buys

This is the core product. Every step is observable — by the buyer in the Mini App and by the operator in the dashboard.

```
1. Buyer opens Telegram bot → Mini App loads product catalogue
2. Adds items to cart → taps Checkout
3. Backend creates order with UNIQUE payment amount (USDC wei or ETH wei)
4. Buyer sees wallet address + QR + countdown (15 min)
5. Buyer sends on-chain (wallet app, MetaMask, or exchange + manual TXID)
6. Watcher sees ERC-20 Transfer or native ETH within seconds
7. Order → PAYING → live block confirmations (1..N) pushed via WebSocket
8. Receipt verified → PAID → deliverFiles() runs automatically
9. Telegram: bot sends ZIP documents to buyer chat
   Web: buyer gets time-limited download token URLs
10. Order → DELIVERED
11. Operator Telegram: "✅ DELIVERED" summary (optional — sale already completed)
```

**Late payments** after expiry are flagged — not silently credited. **Amount mismatches** are rejected and logged. **Double delivery** is prevented by an idempotency table (one send per order/file pair).

---

## Telegram buyer experience

### Shop inside the bot

The storefront is a **Telegram Mini App**: full-screen, dark theme, thumb navigation. Product cards show name, version, price, and status badge (live, beta, unstable). Buyers tap **Add** to build a basket; the bottom bar shows item count and USD total.

Catalogue source is **runtime-switchable**: WordPress API or local warehouse — no redeploy.

### Checkout and payment

At checkout the buyer receives:

- **Order ID** and **exact token amount** (unique per order — critical for matching many buyers to one wallet)
- **Shop wallet address** and **EIP-681 QR code** for mobile wallets
- **MetaMask** pay button in browser mode (connect → switch network → transfer)
- **Manual TXID** field (66-char `0x` hash) for exchange/hardware wallet payers

When a server-side `products.json` / warehouse catalogue is active, **prices and file IDs are validated on the server** — the client cannot tamper with cart totals.

### Live status WebSocket

After checkout, status is **not** poll-and-pray. A WebSocket streams every state change:

| Buyer UI | Backend state |
|----------|---------------|
| Watching for payment… | `PENDING` — expiry countdown running |
| Verifying your transaction… | `MANUAL_VERIFY` or transfer spotted |
| Transaction detected — confirming… | `PAYING` |
| Block 3 of 12 confirmed… | Real block numbers from watcher |
| Payment confirmed — delivering… | `PAID` |
| Delivered — check your chat | `DELIVERED` |

Confirmation progress uses **actual chain blocks**, not a fake timer. Failures (`PAYMENT_FAILED`, `EXPIRED`, `CANCELLED`) surface immediately.

### Auto-delivery

On `DELIVERED`:

- **Telegram buyers** — bot `sendDocument` for each purchased file in the same chat; guarded against duplicate sends on retry
- **Web buyers** — UUID download URLs with configurable max uses and expiry; only returned when the browser proves its `web_session`

### Buyer controls

- **15-minute payment window** with visible countdown
- **Cancel** while `PENDING` (before on-chain payment detected)
- **Cannot cancel** once payment is in flight — protects against abuse mid-confirmation

---

## Browser storefront

The same backend powers a standalone web shop (`le_crypto_app_store_v2.html`):

- Identical catalogue, cart, checkout, chain watcher, and delivery logic
- Persistent **web session** token binds order status and downloads to that browser tab
- MetaMask, QR, and manual TXID flows behave the same as Telegram

Telegram is the primary channel; web is the same engine without the Mini App shell.

---

## Order lifecycle

| Status | Meaning |
|--------|---------|
| `PENDING` | Order created; waiting for payment before expiry |
| `MANUAL_VERIFY` | Buyer submitted TXID; backend checking on-chain |
| `PAYING` | Transfer matched; waiting for N confirmations |
| `PAID` | Payment confirmed; delivery starting |
| `DELIVERED` | Files sent (Telegram and/or web download tokens) |
| `EXPIRED` | Payment window closed without valid payment |
| `PAYMENT_FAILED` | Receipt failed or amount mismatch |
| `CANCELLED` | Buyer cancelled while pending |
| `ADMIN_CANCELLED` | Operator cancelled from dashboard |
| `PAID_NO_FILES` | Paid but no deliverable files on disk — needs operator action |

Every transition is written to the **audit log** and broadcast on the admin WebSocket.

---

## Operator Telegram alerts

The bot notifies the operator chat (`ADMIN_CHAT_ID`) in parallel with dashboard updates:

| Event | Telegram message |
|-------|------------------|
| New checkout | `🛒 NEW ORDER` — user, items, USD total, exact token amount, device, 15 min expiry |
| Payment detected | Transfer spotted — order moving to confirmations |
| Delivered | `✅ DELIVERED` — order ID, file count, buyer, amount |
| Paid, no files | `⚠️ PAID – NO FILES` — manual intervention needed |
| Expired late payment | `⏰ EXPIRED` — TXID arrived after window |
| Payment failed on-chain | Failed receipt notification |

Routine sales complete without operator action. Alerts are **awareness**, not a manual delivery step.

---

## Admin dashboard — real-time control plane

The admin UI is a single-page dashboard (`/admin`) with **eight sections**, desktop top tabs and **mobile bottom navigation** (compact cards on small screens). A persistent **WebSocket** connection drives live updates — order patches, stats, log lines, block ticks, and active confirmation cards — without full page reloads.

**Top bar indicators:** WebSocket connected/disconnected, current block number (live from watcher), logged-in user, last refresh, manual reload.

**Login:** username + password (scrypt-hashed accounts, 12-hour sessions). Role-based UI hides actions you cannot perform. Failed logins lock out after 5 attempts for 15 minutes.

---

### Orders tab

The operational heart of the panel — see every sale from creation to delivery in real time.

**Stats strip** (visible on Orders tab): total orders, revenue rollup, pending count, paying count, delivered count, average order value — updates live via WebSocket.

**Status filter chips** with live counts: All, Pending, Paying, Manual verify, Paid, Delivered, Expired, Failed, Cancelled, Admin cancel. Tap a chip to filter the queue instantly.

**Order table:** sortable columns (Order ID, User, Items, USD, Status, TXID, Created). Paginated (10/25/50/100 per page). Each row shows status badge, explorer link for transaction hash, and item summary.

**Expandable audit trail** per order: timestamped events (CREATED, PAYING, PAID, DELIVERED, EXPIRED, etc.) with detail text, addresses, and Etherscan links. On mobile, orders render as **compact cards** with inline label/value layout; audit opens as a nested panel.

**Admin cancel** — operators with permission can cancel stuck or fraudulent orders (`ADMIN_CANCELLED`).

**Real-time:** new orders appear without refresh; status patches update rows in place; filter counts and stats strip sync on every event.

---

### Warehouse tab

Manage what the shop sells — products, files, and catalogue source.

**Source toggle:** serve buyers from **WordPress API** or **local warehouse** table — switch at runtime.

**Product table:** ID, name, price, version, file ID, ZIP-on-disk indicator, size, status (active/beta/inactive), actions (edit, enable/disable, delete). ZIP files stay on disk when a product row is deleted.

**Add / Edit form:** name, price, version, file ID (maps to `downloads/{id}.zip`), status, size, requirements, badge label, image URL, sort order.

**Import from WordPress** — one-click sync of WP `application` posts into the local warehouse.

**Export `products.json`** on the server — enables strict server-side price validation at checkout.

**Shop & Item Links** (collapsible): full bot shop URL (`BOT_LINK`) and per-product **deep links** (`?startapp=`) for Telegram channel campaigns — copy-friendly.

**File Checker:** scan downloads folder, compare registry vs disk, highlight missing ZIPs.

**Debug file registry** (collapsible): raw SHA-256 table, re-scan button, list of files on disk not in registry.

---

### Wallet tab

On-chain treasury view for the shop wallet.

**Balance cards:** wallet address, network, ETH balance, configured token balance (e.g. USDC), contract address, decimals, total USD received from paid orders.

**Incoming transactions table:** paginated list of orders with on-chain payments — order ID, user, USD, token amount, status, TXID, time, explorer links.

**Outgoing sends history:** operator-initiated ERC-20 transfers with time, recipient, amount, note, TXID.

**Send tokens form** (admin role, `wallet.send`): recipient `0x…`, amount, optional note — signs and broadcasts when `WALLET_PRIVATE_KEY` is configured. Irreversible — UI warns clearly.

---

### Logs tab

Live **terminal-style** view of everything the backend emits.

**Filters:** ALL, INFO, ORDERS, BACKEND, WATCHDOG, ERROR, BLOCKS.

**Controls:** autoscroll toggle, clear buffer.

**Stream source:** admin WebSocket — same events as server console: order lifecycle, watchdog block ticks, wallet events, delivery results, errors.

Use this to watch a sale happen **second by second** without tailing SSH logs.

---

### System tab

Health and architecture overview.

**Metric cards:** process uptime, memory usage, order counts by status, audit log volume, revenue-style rollups, status distribution chart.

**Architecture flow diagram:** visual path Store → API → DuckDB → Watcher → Chain RPC → Bot → Buyer (conceptual onboarding aid).

**Stack info panels:** backend runtime, blockchain config, bot, database — at a glance.

---

### Chain tab

Dedicated blockchain watcher view — what the automation sees on-chain.

**Status cards:** network name, current block, watchdog ONLINE/OFFLINE, token symbol, wallet address (truncated), explorer links.

**Active confirmations** (live): when orders are in `PAYING`, cards show per-order block progress (populated by WebSocket `CONFIRMING` events) — same data the buyer sees, from the operator side.

**Incoming payments table** — paginated on-chain matched orders.

**Outgoing sends table** — paginated operator transfers.

**Live chain event log:** scrolling feed of payment-related WebSocket events; optional "hide block spam" filter.

---

### DB Browser tab

Read-only SQL console against the analytics DuckDB (`orders`, `audit`, `warehouse`, `sessions`, `deliveries`, `download_tokens`, etc.).

**Table list** with row counts — click to preview.

**SQL input:** `SELECT`, `SHOW`, `DESCRIBE` only — mutations rejected server-side.

**Quick-query buttons** for common support queries.

**Paginated result grid** with configurable page size (up to 250 rows).

Owner and admin roles only (`db.read` permission).

---

### Settings tab

Dashboard preferences (localStorage) plus live backend configuration.

**Privacy — IP masking:** toggle masking in order panels vs terminal log lines.

**Terminal display:** max log lines (100–5000), show/hide block events, autoscroll default.

**Orders:** fetch limit (bulk load cap), optional row background tint. Synced with Orders tab "Per page" dropdown.

**Admin users** (owner only): list accounts with roles; create user (username, password 8+, role); enable/disable; reset password. Compact card layout on mobile.

**Session:** logout — ends 12-hour server session.

**Pagination default:** unified rows-per-page for Orders, DB, Wallet, and Chain tabs.

**Warehouse section defaults:** which collapsible panels start open (file registry, shop links, file checker).

**Backend config** (runtime-editable, role-gated): accepted currencies (USDC/ETH), ETH/USD rate for test pricing, confirmation block count, basket size cap, download token max uses, bot link — save without redeploying code.

---

## Access control and security

| Role | Capabilities |
|------|--------------|
| **Owner** | Full access including user management and DB browser |
| **Admin** | Orders, warehouse write, wallet read/send, settings, files, chain, audit |
| **Operator** | Orders read + cancel, warehouse read, system/chain/audit read — no wallet send or settings write |
| **Viewer** | Read-only orders, system, chain, audit |

**Sessions** replace shared static API keys for daily use (12-hour TTL, hashed tokens server-side). Optional legacy `x-admin-key` mode exists for break-glass automation — **disabled by default**.

**Buyer APIs:** `order-status`, `cancel-order`, `manual-txid`, and download tokens require `X-Web-Session` when the order has a web session — order ID alone is insufficient (IDOR protection).

**Telegram checkout:** HMAC-validated `initData` ties identity to Telegram user.

**Admin WebSocket:** token sent in post-connect auth message — not in URL query string.

**Download tokens:** expiring, usage-limited, atomic decrement on fetch.

**Debug `verify-payment`:** hidden unless `DEBUG_MODE=true` and owner role.

---

## WordPress catalogue (LE Shop plugin)

WordPress is the **editorial layer** for many deployments — product copy, thumbnails, and structured metadata live on logicencoder.com (or any WP site with LE Shop installed).

- **`application` custom post type** with price, version, status, file ID, badge, requirements
- **Webhook** on publish/update/delete/status change → `POST /webhook/wp-changed` (fire-and-forget)
- **REST API** enriches `wp/v2/application` with normalized `metadata` for Mini App and backend import
- **One-click import** in admin dashboard warehouse tab syncs WP → local warehouse

See **[le-shop-plugin-overview](https://github.com/logicencoder/le-shop-plugin-overview)** for the WordPress side — dashboard, items/warehouse comparison, settings, and operator workflow.

---

## Tech stack

| Layer | Technologies |
|-------|----------------|
| Realtime commerce | Telegram Mini App + buyer WebSocket status hub |
| Backend | Node.js, Fastify |
| Database | DuckDB — orders, audit, warehouse, sessions, deliveries, file registry |
| Chain | ethers v6, ERC-20 Transfer logs + native ETH, configurable confirmations |
| Delivery | Telegraf bot (`delivery.js`) + tokenized web downloads |
| Storefront | HTML/JS, MetaMask, EIP-681 QR |
| Admin UI | Single-page dashboard, role-aware, mobile bottom nav |
| Catalogue sync | WordPress REST + webhook (LE Shop plugin) |
| Operator alerts | Telegraf → admin Telegram chat |

---

## Related repositories

| Repository | Role |
|------------|------|
| [le-crypto-app-store](https://github.com/logicencoder/le-crypto-app-store) | Private application code |
| [le-crypto-app-store-overview](https://github.com/logicencoder/le-crypto-app-store-overview) | This product overview |
| [le-shop-plugin](https://github.com/logicencoder/le-shop-plugin) | Private WordPress catalogue plugin |
| [le-shop-plugin-overview](https://github.com/logicencoder/le-shop-plugin-overview) | WordPress catalogue companion overview |

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
