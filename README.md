# Logic Encoder Crypto App Store

Telegram-first software shop: browse Logic Encoder tools, pay with **USDC or ETH**, receive zip files automatically. Product copy lives in **WordPress**; checkout, chain matching, and delivery run in a dedicated **Node/Fastify** backend.

Private source: [logicencoder/le-crypto-app-store](https://github.com/logicencoder/le-crypto-app-store). Catalogue plugin: [le-shop-plugin](https://github.com/logicencoder/le-shop-plugin) (private).

## The problem it solves

Crypto-native buyers do not want card checkout or multi-page WooCommerce flows inside Telegram. Operators still want to edit app descriptions in WordPress without redeploying Node on every price change.

This system splits roles: **WordPress = catalogue**, **Node service = money + files + bot**.

## Telegram Mini App storefront

Full-screen shop UI embedded in Telegram — product cards (price, badge, version), basket, wallet payment flow, and order status polling. Mobile-first WebApp UX instead of forcing desktop checkout.

## Unique payment amounts

Each checkout session generates a slightly unique ERC20 amount (derived from a session hash) so incoming transfers map to exactly one open order on a shared deposit address — no reliance on user-entered memo fields.

## On-chain confirmation and delivery

The service listens for USDC/USDT `Transfer` events and native ETH payments, waits for confirmations, then marks orders paid. After payment, the bot sends purchased zips to the buyer’s Telegram chat with duplicate-delivery guards. Orders in `PENDING` expire after ~15 minutes; late payments surface admin alerts instead of silent mismatches.

## Web checkout path

Buyers without a Telegram chat ID can receive **limited-use HTTPS download tokens** after payment — hybrid flows where marketing pages point to a web session instead of bot-only checkout.

## WordPress catalogue sync

When an operator publishes an application in WordPress, a webhook notifies the backend; admin can also bulk sync from REST. Marketing site and shop share one catalogue without duplicate data entry in Node.

## Operator admin dashboard

Browser UI at `/admin`: live orders, stats, warehouse editor, missing-file flags, wallet send tool, database inspection, runtime toggles, and WebSocket live feed. DuckDB stores basket lines and audit entries for funnel analysis.

## Order lifecycle

```
PENDING (~15 min TTL)
   ├─▶ PAYING (tx seen, confirming)
   ├─▶ PAID
   │     └─▶ DELIVERED (Telegram files / download tokens)
```

Stack: **Fastify 5**, **DuckDB** (`analytics.db`), **Telegraf**, **ethers v6**, merged ERC20 + ETH watchdog in one process.

See [REPOS.md](REPOS.md) for related repos.

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
