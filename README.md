# Logic Encoder Crypto App Store

Telegram Mini App shop for LogicEncoder digital tools: browse apps, pay with ERC20 stablecoins, automatic delivery, and an operator admin dashboard.

**Live stack:** Telegram → Node backend on SOL → WordPress catalogue on logicencoder.com.

---

## What it does

- **Telegram WebApp storefront** — basket, unique payment amounts, QR / wallet flow  
- **ERC20 payment detection** — listens on-chain, confirms orders, expires stale checkouts  
- **File delivery** — registered downloads to Telegram chat or signed web token  
- **WordPress catalogue** — products managed as `application` posts; changes sync to backend via webhook  
- **Admin Mission Control** — orders, warehouse, wallet sends, analytics, DuckDB tools  

---

## Repositories

| Repo | Contents |
|------|----------|
| [le-crypto-app-store](https://github.com/logicencoder/le-crypto-app-store) | Backend source (Fastify + DuckDB) |
| **This repo** | Public overview |
| [le-crypto-app-store-plugin](https://github.com/logicencoder/le-crypto-app-store-plugin) | WordPress `le-shop` + `le-settings` |
| [le-crypto-app-store-plugin-overview](https://github.com/logicencoder/le-crypto-app-store-plugin-overview) | Plugin overview |

Implementation details: [`ARCHITECTURE.md`](https://github.com/logicencoder/le-crypto-app-store/blob/main/ARCHITECTURE.md) in the code repo.

---

## Production map

| Layer | Where |
|-------|--------|
| Backend runtime | SOL `/home/sol/lojzo/le_crypto_app_store` |
| WordPress plugins | Hostinger `logicencoder.com` |
| Public site | https://logicencoder.com |

---

**Made by [logicencoder](https://github.com/logicencoder)**
