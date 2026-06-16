# Logic Encoder Crypto App Store

Telegram-first software store for Logic Encoder tools: browse products in a Mini App, pay with **USDC or ETH**, and receive delivery files automatically after chain confirmation. WordPress handles catalogue content, while this backend owns payment detection, order state, and file delivery.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Backend | Node.js + Fastify |
| Persistence | DuckDB (`analytics.db`) |
| Chain integration | ethers v6 (ERC20 + native ETH watchers) |
| Delivery | Telegraf bot + tokenized web downloads |
| Integrations | WordPress webhook + REST catalogue sync |

## Telegram storefront and checkout flow

The app runs as a Telegram Mini App with product cards, basket flow, payment instructions, and status polling in one mobile-first UI. Buyers can complete purchases without switching to card checkout plugins or leaving Telegram context.

Each order gets a unique payment amount derived from session data. This allows deterministic matching of incoming transfers to one order even when many buyers pay to the same destination wallet.

## Payment confirmation and delivery pipeline

The backend watches on-chain transfers, moves orders through states (`PENDING` → `PAYING` → `PAID` → `DELIVERED`), and sends files after confirmation checks. Delivery guards prevent duplicate sends for the same order/file pair.

Late or ambiguous payments are surfaced to operators instead of silently failing. This is critical when traffic spikes and several sessions are active at once.

## WordPress catalogue sync

Store content is synchronized from WordPress through webhook events and optional REST import. That keeps marketing pages and checkout inventory aligned without editing product metadata in two systems.

The model is simple: WordPress is catalogue authority; this service is order and payment authority.

## Operator control plane

An admin dashboard provides live operational visibility:

- order queues and payment state
- warehouse/file availability checks
- stats and audit inspection
- runtime toggles and diagnostics

This screen is used during incidents (missing file, delayed payment, stuck delivery) to resolve issues without SSH-only workflows.

## Delivery channels

After payment, users can receive files through:

- Telegram bot document delivery
- short-lived tokenized web download links (for non-bot flows)

This hybrid path supports both pure Telegram storefront usage and web-assisted purchase journeys.

Private code: [logicencoder/le-crypto-app-store](https://github.com/logicencoder/le-crypto-app-store)  
Catalogue source: [le-shop-plugin](https://github.com/logicencoder/le-shop-plugin)  
See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
