# Drip Shop Live GraphQL API

- generated: '2026-07-18'
- method: searched
- source: https://api.dripshop.live/graphql (live introspection)

## Overview

Drip Shop Live (legal entity **Flourish Cares, Inc.** DBA Drip Shop Live) operates a
single public GraphQL API that powers its live-shopping mobile and web clients. The
endpoint at `https://api.dripshop.live/graphql` had **introspection enabled** at the time
of capture, so this schema was harvested verbatim from the live server rather than
reconstructed. There is no separately published developer portal or OpenAPI document; this
GraphQL schema is the authoritative machine-readable description of the surface.

## Endpoint

- **GraphQL endpoint:** `https://api.dripshop.live/graphql`
- **Transport:** HTTP POST (queries/mutations); GraphQL subscriptions over WebSocket for realtime
- **Health probe:** `query { health }` returns `true` unauthenticated
- **Schema file:** [`dripshop-schema.graphql`](dripshop-schema.graphql) (SDL) and
  [`dripshop-introspection.json`](dripshop-introspection.json) (raw introspection result)

## Authentication

Most operations require an authenticated user. An unauthenticated `query { me { id } }`
returns an `errors[]` payload, while `query { health }` succeeds. Authentication is a
**bearer token** presented on the `Authorization` header, minted through the app's login
flows: wallet / Farcaster sign-in (`connectOrLoginWithWallet`, `createFarcasterUser`) and
linked social identities (Facebook, Twitch, YouTube OAuth). See
[`../authentication/dripshop-authentication.yml`](../authentication/dripshop-authentication.yml).

## Shape and scale

| Root | Fields |
|---|---|
| Query | 268 |
| Mutation | 283 |
| Subscription | 56 |
| Named types | 1,112 (739 objects, 204 unions, 133 enums, 27 input objects, 7 scalars, 2 interfaces) |

The schema is built with a Pothos/Prisma-style code-first server: it uses Relay-style
cursor **connections** (`*Connection` / `*ConnectionEdge` with `PageInfo`) for list
queries and tagged **union result types** (`*Success` / error members) for many operations.

## Domain coverage

- **Live streaming & simulcast** — `Stream`, `Simulcast`, RTMP key management, moderators,
  raids, chat, presence, announcements, shout-outs (`stream`, `streamsLiveNow`,
  `streamRtmpKeyCreateV2`, `simulcastStartAll`).
- **Auctions** — realtime bidding with pre-authorization holds (`auction`, `auctionPlaceBid`,
  `auctionMaxBid`, `auctionCreateBidPreAuth`, `AuctionBid`).
- **Box breaks** — the collectibles "break" format: spots, templates, randomized spot
  assignment, reveals, buybacks (`boxBreak*`, `BoxBreakSpot`).
- **Giveaways & drops** — entries, spins, winners, multi-entry giveaways, referral coupons
  (`giveaway*`, `drop`, `enterDrop`).
- **Pull games / instant packs** — provably-fair pulls with client/server seeds
  (`pullGame*`, `instantPack*`, `PullGameServerSeed`).
- **Catalog & products** — products, collections, categories, tags, card pricing, comments
  (`product*`, `productCollections`, `tag*`, `CardPricing`).
- **Commerce** — carts, checkout, orders, invoices, transactions, bulk offers, wholesale
  (`cart*`, `checkoutCart*`, `order*`, `invoice*`, `transaction*`, `bulkOffer*`).
- **Payments** — Stripe (`cartStripeCreatePaymentIntentId`), PayPal
  (`cartPaypalCreateOrder`), and crypto/wallet rails including Solana, Base and Farcaster
  (`checkoutCartPrepareCryptoPayment`, `cartWalletPurchase`, `usdToETH`).
- **Fulfilment & shipping** — shipments, parcels, rates, labels, store pickups, fulfilment
  partners (`shipment*`, `storePickup`, `fulfilmentPartners`).
- **Social & messaging** — follows, conversations, direct messages, posts, blocking
  (`userFollowers`, `userConversations`, `createMessage`, `posts`).
- **Rewards & subscriptions** — Drip Coin / driplets leaderboards, plans and creator
  subscriptions (`dripletEarnedLeaderBoard`, `subscribeToPlanV2`, `mySubscription`).

## Realtime (subscriptions)

56 GraphQL subscriptions provide the live surface: stream events, auction bid counts,
giveaway events, chat/new-message notifications, cart updates, and per-user events
(`streamEvents`, `auctionBidCount`, `newMessageNotification`, `userOutBid`, `cartV2`).

## Notes for consumers

- This is a **first-party, app-facing** GraphQL API. Drip Shop does not publish a partner
  developer program, SDKs, or rate-limit documentation; treat the schema as descriptive.
- Pagination follows the Relay Connections spec — see
  [`../conventions/dripshop-conventions.yml`](../conventions/dripshop-conventions.yml).
- Entity relationships are mapped in
  [`../data-model/dripshop-data-model.yml`](../data-model/dripshop-data-model.yml).
