# Logic Encoder Crypto App Store

![Telegram catalogue, cart, payment progress and completed delivery](assets/Le-CAS-Admin9.png)

Logic Encoder Crypto App Store sells digital products through a Telegram Mini App and browser storefront. Buyers browse a catalogue, build a basket, pay in cryptocurrency and receive purchased files automatically. A live operator dashboard follows each purchase from creation through blockchain confirmation to delivery. Open the [Telegram store](https://t.me/v52cfgr5452dbot/cryptoshop) or [browser storefront](https://logicencoder.com/app/).

## Tech stack

| Layer | Technologies |
|---|---|
| Server | JavaScript, Node.js, Fastify 5 |
| Interfaces | HTML, CSS, JavaScript, responsive single-page dashboards |
| Live updates | WebSockets and authenticated HTTP status recovery |
| Blockchain | Ethereum, Sepolia, ethers v6, ERC-20 logs, JSON-RPC |
| Provider resilience | PublicNode, Alchemy, Infura, configurable HTTP and WebSocket endpoints |
| Telegram | Telegram Mini Apps, Telegraf, document delivery |
| Persistence | DuckDB |
| Catalogue publishing | WordPress REST API and authenticated webhooks |
| File verification | SHA-256 registration and server-side availability checks |
| Regression checks | Node.js test runner, Playwright Chromium, isolated Anvil chain |

## Telegram catalogue and basket

Product cards bring together the application image, name, description, version, price and optional status badge. Buyers use **Add** to assemble a basket, review the total, remove individual items and return to browsing before proceeding to payment.

Product deep links can open a selected item from an announcement. Operators copy these entry points from the warehouse, making the same catalogue useful for both browsing and product-specific Telegram posts. The browser storefront offers the same catalogue and order flow outside the Telegram shell.

The catalogue and operator order detail can be inspected together. Buyers choose products in the Mini App while the operator follows basket contents, the transaction reference and the delivery audit.

![Telegram product catalogue beside the operator order detail](assets/Le-CAS-Admin6.png)

## Checkout and live payment progress

**Proceed to Payment** creates an order and opens its payment screen. The buyer sees the order ID, purchased items, exact amount, destination wallet and expiry countdown. Copy controls help transfer the amount and address into a wallet; QR and wallet integration controls support payment without manually retyping the destination.

Configured payment modes include USDC and native ETH. The screen identifies the selected network, including Sepolia in test mode. A manual transaction-hash control lets a buyer identify a payment sent through another wallet or service.

Payment state progresses from watching for a transaction to detection, confirmations and delivery. Confirmation progress comes from observed blockchain blocks. WebSocket messages carry live updates, while authenticated HTTP recovery reconciles the current order after missed events or a connection interruption.

- Checkout validates product information and file availability against the selected catalogue.
- The countdown exposes the order's payment window.
- Buyer cancellation is available while the order remains pending.
- Detection starts confirmation handling before delivery.
- Ambiguous payment matches request verification rather than selecting an arbitrary order.

## Automatic file delivery

The bot sends purchased documents directly into the buyer's Telegram conversation after payment verification. Messages identify the order and delivery outcome. The Mini App switches to **Files Delivered!** and offers **Back to Store**, while the operator sees the corresponding completed order.

Browser purchases receive expiring download links associated with their buyer session. Individual file delivery records let retries account for work already handled. Missing, oversized or integrity-invalid files follow explicit error handling instead of being silently counted as a complete purchase.

The combined view below connects buyer acknowledgement, document receipt and the stored audit trail. An operator can verify that the same order completed across the chat, storefront and dashboard.

![Completed purchase, Telegram document and delivery audit](assets/Le-CAS-Admin10.png)

## Orders and audit trails

The **Orders** area combines summary figures, status filters and a paginated purchase queue. Rows show the buyer, basket, value, status and transaction reference. Filter chips isolate pending payments, orders being confirmed, completed deliveries and purchases requiring attention.

Expanding a row reveals order information, purchased product details and timestamped audit events. Creation, payment detection, payment acceptance and delivery outcomes remain associated with the purchase. Explorer links allow the operator to inspect the underlying transfer, while permitted cancellation actions support eligible interventions.

Live patches update order state and dashboard figures while the panel remains open. Status colours distinguish waiting, confirming, paid, delivered and unsuccessful outcomes; the expanded detail explains the history behind each badge.

![Orders dashboard with expanded details and history](assets/Le-CAS-Admin7.png)

## Warehouse and deliverable management

The **Warehouse** controls the product catalogue and its files. Operators select WordPress or the local warehouse as the source, import products, export catalogue metadata, add applications and edit existing entries.

Product fields cover name, price, version, file reference, status, requirements, badge and presentation metadata. Availability and file size appear alongside the catalogue row, making it easier to verify a deliverable before publishing. Removing a product entry does not require deleting its ZIP.

File registry and checker panels compare registered files with disk availability. SHA-256 and size information support integrity checks, and shop/item links provide copyable Telegram entry points. A typical publishing workflow is to prepare the file, verify its registration, publish or import the product, and check availability before announcing it.

![Warehouse product list and file verification panels](assets/Le-CAS-Admin1.png)

## Wallet and reconciliation

The **Wallet** area places balances and payment history together. Summary cards show the configured wallet, native asset balance, token balance and received value. Incoming payments retain order references, statuses and explorer links for reconciliation.

Incoming and outgoing records have separate paginated lists. Outgoing transfers include recipient, amount, note and transaction reference. When signing is configured, an authorised operator can enter a destination and amount in the send form.

Wallet actions are permission controlled. The payment history gives support and accounting work a direct connection between an order and its on-chain transfer.

![Wallet balances, incoming payments and outgoing transfers](assets/Le-CAS-Admin2.png)

## Blockchain watcher

The **Chain** area shows the payment watcher's network, current block, availability, configured token and receiving wallet. Active confirmation cards expose the detected block, target block and progress of purchases awaiting confirmation.

Incoming payment records link activity back to the sales queue. Outgoing history adds the corresponding treasury view. The confirmation card helps an operator distinguish normal waiting for blocks from a watcher connection problem.

WebSocket subscriptions supply new blocks and relevant token transfer events. HTTP recovery scans overlap persisted checkpoints to recover missed activity. Configurable provider priority, chain identity checks and fallback handling support continuity when an endpoint becomes unavailable.

![Chain watcher with active confirmation progress](assets/Le-CAS-Admin8.png)

## Live logs

The **Logs** area offers a terminal-style stream within the dashboard. Filters separate order activity, backend events, watcher output, errors and block events, helping the operator follow a purchase or investigate a delivery failure.

Autoscroll and buffer controls support live observation and incident review. Preferences control retained lines and block-event visibility. IP masking can be configured separately for terminal output and surrounding panels.

The terminal provides runtime context across the shop; an order's structured audit supplies the history of a specific purchase.

## System health

The **System** area brings runtime health and order distribution together. Cards expose uptime, process memory, order counts, audit volume and sales totals. The status distribution display helps identify a build-up of purchases in waiting, confirming or unsuccessful states.

Runtime and stack panels provide an operational configuration summary. The persistent header also shows WebSocket connectivity and the current block as the operator moves between tabs.

Use the overview to establish whether the application is running and processing purchases, then move into Orders, Chain or Logs for the specific issue.

![System health, runtime metrics and order distribution](assets/Le-CAS-Admin3.png)

## Database inspection

The **DB** workspace supports authorised inspection of application records. Table selectors and quick-query buttons help locate order, audit and delivery information; the SQL input supports more specific support questions.

Results appear in a paginated grid with configurable display size. The query API applies role checks and a statement-type restriction. Normal order and catalogue changes use the dedicated application controls.

This screen provides the stored detail behind a dashboard status or transaction reference, alongside the other operational tools.

![Database query workspace and paginated results](assets/Le-CAS-Admin4.png)

## Settings and access management

**Settings** separates local dashboard preferences from backend configuration. Display controls include log retention, autoscroll, block-event visibility, order fetch limits, pagination, row tint and the initially expanded warehouse panels.

Privacy controls mask IP addresses in panels and terminal output. Backend controls configure accepted payment modes, confirmation requirements, basket limits, download use limits and the bot link.

**RPC priority** allows authorised operators to reorder available named providers and apply the choice through **Save Runtime Settings**. The saved order updates RPC connections without restarting the service. Explicit endpoint configuration is indicated separately.

Owner-managed accounts support roles, account creation, disabling access and password resets. Session controls provide logout; actions elsewhere in the dashboard follow the authenticated account's permissions.

The capture illustrates the display, privacy, backend and account-management areas. Provider controls depend on the deployed configuration.

![Display preferences, backend configuration and administrator accounts](assets/Le-CAS-Admin5.png)

## WordPress catalogue companion

The [LE Shop plugin](https://github.com/logicencoder/le-shop-plugin-overview) supplies an editorial workflow for application descriptions, images, versions, prices and file references. Publication changes notify the store, which reads current canonical information before updating its catalogue projection.

The warehouse source selector chooses the publishing model. WordPress mode follows the editorial catalogue; local mode follows products maintained in the operator panel. Checkout validates against the selected source and reports source failures rather than trusting buyer-supplied prices.

Product editing can therefore remain in WordPress while the store handles order state, blockchain payment checks, buyer updates and digital delivery.

## Buyer ownership and operational safeguards

Status access, cancellation and transaction submission require proof of buyer ownership. The Mini App authenticates its WebSocket order subscription after connecting, and status recovery respects the same ownership boundary. Telegram identity validation and buyer session capabilities support access to the appropriate order.

The server owns order state. Conditional updates protect competing payment claims, and receipt checks validate payments before delivery. The interface ignores obsolete recovery replies after accepting newer live state.

Operator accounts use password hashing, expiring sessions and permission checks. Catalogue webhooks have a separate authentication boundary. File registration, per-file delivery tracking and limited browser download links provide controls around fulfilment.

---

Private code: [le-crypto-app-store](https://github.com/logicencoder/le-crypto-app-store)

WordPress companion: [le-shop-plugin-overview](https://github.com/logicencoder/le-shop-plugin-overview)

See [REPOS.md](REPOS.md) for related repositories.

**[Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
