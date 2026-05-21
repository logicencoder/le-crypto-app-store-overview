# Logic Encoder Crypto App Store — public overview

Telegram-first software shop for LogicEncoder: discover tools, pay with cryptocurrency, receive files automatically — with a WordPress-backed catalogue on [logicencoder.com](https://logicencoder.com).

**Implementation (private):** [le-crypto-app-store](https://github.com/logicencoder/le-crypto-app-store)

---

## The problem

Selling small digital tools to a crypto-native audience has friction:

- Card processors and traditional checkout are a poor fit.  
- Buyers want to pay from a wallet inside Telegram, not a multi-page WooCommerce flow.  
- Exact payment amounts must be unique so the server knows **which** order was paid without trusting a memo field.  
- Operators still want to edit product copy in **WordPress**, not redeploy Node on every price change.

This system splits responsibilities: **WordPress = catalogue**, **Node service = money + files + bot**.

---

## Who benefits

| Audience | Outcome |
|----------|---------|
| **Buyer** | Opens Mini App, sees live apps, pays USDC/ETH, receives zips in Telegram or via time-limited download link |
| **Operator** | Edits apps in wp-admin; monitors orders, warehouse disk status, and wallet activity in admin dashboard |
| **Public site visitor** | Reads application pages on logicencoder.com (fed by same catalogue metadata) |

---

## Ecosystem (do not merge with other plugins)

| Component | Role |
|-----------|------|
| **le-crypto-app-store** | This overview — checkout, chain matching, delivery |
| **le-shop-plugin** | WordPress catalogue + webhook when products change |
| **le-settings-plugin** | Site security/SEO — unrelated to payments |

---

## Capabilities (detailed)

### Telegram Mini App storefront

**What:** Full-screen shop UI served at `/` on the backend URL, embedded in Telegram. Shows product cards (price, badge, version), basket, payment QR / wallet flow, order status polling.

**Why:** Meets users where they already are (Telegram) with native WebApp UX instead of forcing a desktop checkout.

**Who benefits:** Mobile-first buyers; higher conversion for small impulse-priced tools.

### Unique payment amounts (anti-collision)

**What:** Each checkout session generates a slightly unique ERC20 amount (derived from a session hash) so incoming transfers map to exactly one open order.

**Why:** Shared deposit addresses cannot rely on user-entered order IDs in chain transfers; amount fingerprinting is the reconciliation key.

**Who benefits:** Operator automation; buyers only send one precise number.

### Automatic on-chain confirmation

**What:** Service listens for `Transfer` events (USDC/USDT) and native ETH payments, waits N confirmations, then marks order paid.

**Why:** Manual tx checking does not scale and adds hours of delay.

**Who benefits:** Buyers get files in minutes; operator sleeps.

### Timed checkout expiry

**What:** Orders in `PENDING` expire after ~15 minutes; late payments trigger admin Telegram alerts instead of silent wrong deliveries.

**Why:** Prevents old quotes being paid after price changes; surfaces accounting edge cases.

**Who benefits:** Operator accounting integrity; buyers prompted to start a fresh checkout.

### Automatic file delivery

**What:** After payment, bot sends each purchased zip to the buyer’s Telegram chat; tracks deliveries to avoid duplicates on retry.

**Why:** Buyer paid for automation — manual email delivery would defeat the product.

**Who benefits:** Buyer instant gratification; operator not in the loop per sale.

### Web checkout path (download tokens)

**What:** For buyers without Telegram chat id, system issues limited-use HTTPS download tokens after delivery.

**Why:** Supports hybrid flows where marketing page points to web session instead of bot-only.

**Who benefits:** Buyers who start on web but still pay on-chain.

### WordPress catalogue sync

**What:** When operator publishes an application in WordPress, webhook notifies backend; admin can also bulk `sync-from-wp` from REST.

**Why:** Marketing site and shop stay one catalogue without duplicate data entry in Node.

**Who benefits:** Operator editorial workflow unchanged.

### Operator admin dashboard

**What:** Browser UI at `/admin`: live orders, stats, warehouse editor, disk missing flags, wallet send tool, DB inspection, runtime toggles, WebSocket live feed.

**Why:** Production operations (refunds, missing zip uploads, stuck orders) need a control plane beyond Telegram alerts.

**Who benefits:** Operator and support during incidents.

### Analytics logging

**What:** Basket lines and audit entries stored in DuckDB for later reporting.

**Why:** Understand what users attempt to buy and where funnel drops.

**Who benefits:** Operator improving catalogue and pricing.

---

## Live operation note

Runtime runs on private infrastructure (SOL) exposed via secure tunnel to Telegram and WordPress — specifics are in the private repo, not here.

---

## Related repositories

[REPOS.md](REPOS.md)

---

**Made by [logicencoder](https://github.com/logicencoder)**
