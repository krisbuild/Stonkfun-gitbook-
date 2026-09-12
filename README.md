# Overview

StonkFun is a permissionless token launchpad on Solana built around one defining idea:

> Any token, paired against anything — a stock, a currency, a meme, or any other token.

The same launch mechanics and non-custodial guarantees apply regardless of what's on the other side of the pair — though the quote asset chosen directly determines the token's price behavior, since its value is permanently denominated in that asset.

## How launches work

Creating a launch mints the token and sizes its curve in the same flow, priced against whatever quote asset the creator picks — not just SOL. The creator sets the name, symbol, image, description and links at creation, and chooses a launch mode at the same time: **Standard**, which carries no tax and pays the creator a share of trading fees, or **Reward**, a Token-2022 mint with a permanent 1% or 3% transfer tax and no creator fee position. Both the quote asset and the mode are locked in at creation — neither can be changed afterward.

***

**01 Choose**
Pick a quote asset from StonkFun's ever-growing curated list, and a launch mode. Both are permanent.

**02 Price**
The curve is sized in the quote asset itself, using its live USD price, so the launch is worth the same regardless of what it's priced against.

**03 Sign**
The creator signs a single fee payment from their own wallet. No account, no API key, nothing held in custody.

**04 Land**
The creator's opening buy, if any, is submitted as part of the same atomic Jito bundle as the payment, mint, pool and liquidity — everything lands together in a single block, or the whole launch fails with nothing charged.

**05 Trade**
The pool goes live and every trade pays a fee from that point on.

***

### Launch protection

**The opening buy can't be front-run.** It's executed as the pool's literal first trade, bundled into the same atomic Jito bundle as the mint, pool and liquidity that make it tradeable at all — there is no block in which anyone else could trade ahead of it.

**A launch that doesn't land doesn't charge.** If the bundle fails to land, the response comes back `service_unavailable` with nothing charged — the creator gets a fresh quote to retry, not a stuck payment. And because that quote is priced from the quote asset's live rate at signing time, an asset the pricing service can't currently value fails the launch before anything is signed, rather than launching at a wrong price.

## Platform at a glance

| Field | Value |
|---|---|
| **Chain** | Solana (mainnet-beta) |
| **Base API URL** | `https://www.stonkfun.xyz/api/public/v1` |
| **API key** | None required — every endpoint is open, rate-limited per IP |
| **Tokens launched** | Growing continuously, around the clock |
| **Quote assets** | An ever-growing curated list, spanning 9 categories |
| **Graduation threshold** | $40,000 market cap (configured); tokens are flagged "about to graduate" at $32,000 |
| **Platform token** | $STONK — fixed supply, mint and freeze authority both permanently revoked |

## Quote Assets

A quote asset is what a new token is priced against — the other side of the pair, which can be almost anything, not just SOL.

### The list is curated

New quote assets are vetted and added to the approved list by StonkFun. Subsequently, users will be able to add their own custom quote asset by burning $STONK.

Before building a launch, the API's own guidance is to call `/pairs` first: `quoteMint` must be one of the assets it returns.

### The categories

The list is spread across various categories:

| Category | Label shown | Examples |
|---|---|---|
| `custom` | Custom | Ordinary memecoins and established Solana tokens |
| `xstock` | xStock | SPYX (S&P 500), NVDAX (NVIDIA), TSLAX (Tesla), GOOGLX (Google), COINX (Coinbase), MSTRX (MicroStrategy) |
| `backpack` | **Sunrise** | MU (Micron), TTWO (Take-Two), SNDK (SanDisk), NBIS, SILVER |
| `prestock` | PreStock | ANTHROPIC, ANDURIL, NEURALINK, POLYMARKET, FIGUREAI, OPENAI, KALSHI |
| `currency` | Currency | USDC, USDT, EURC, ONYC, JLUSDC |
| `leverage` | Leverage | xSOL, XBTC |
| `solana` | Solana | SOL (Wrapped), SKR |
| `collectible` | Collectibles | SV151, HEEBOO |

### What it takes for a pair to be usable

Clearing curation isn't the whole story — a pair has to clear three separate requirements before a launch against it will actually go through.

**First, the asset needs at least $50,000 in liquidity seeded on Raydium.** Below that threshold, a quote asset isn't eligible to be added to the list — a market that thin can't be priced or reliably traded against, so this comes before anything else is even considered.

**Second, the team has to approve it.** A pair doesn't get added just by clearing the liquidity bar — it has to be reviewed and whitelisted by StonkFun.

**Third, if the launch runs on LaunchLab, Raydium has to have separately provisioned that asset on-chain.** Raydium's own on-chain `GlobalConfig` for that specific quote asset has to already exist. The API reports this as `launchLabReady`.

### Symbols collide — match by mint address

A ticker symbol (`SOL`, `ALON`, `USDC`) is just a label — nothing stops two different tokens from picking the same one. A mint address is the actual identifier: a long, unique string that can never collide.

Roughly 1 in 9 pairs on the list shares its symbol with at least one other pair. Two different tokens both trade as `ALON`. Separately, `xBTC` and `WBTC` both represent wrapped Bitcoin, just from different bridges — same idea, different tokens.

If code looks up a quote asset by its symbol instead of its mint address, it can silently grab the wrong one — no error, just the wrong token used. Always identify a quote asset by its mint address. The symbol is fine to show a person; it's the wrong thing for code to match on.

### How pricing works

Every quote asset is priced independently: its own live USD rate, converted using its own decimals, is what sizes a launch's curve. There's no single shared calculation across the list — each quote asset runs through this on its own, which is what lets StonkFun price a launch against anything from a stablecoin to a tokenized stock using the same underlying method.

## Launch venue

Every launch runs through LaunchLab. It costs network rent only (~0.012–0.013 SOL) — no platform fee. The token starts on a bonding curve with no upfront liquidity: it trades against that curve until enough of the quote asset has been raised, then graduates automatically into a real Raydium pool.

A LaunchLab launch can also be built directly against Raydium's program without going through StonkFun's API at all — StonkFun scans the chain for pools carrying its platform id and adopts anything it finds within a minute or two.

## Launch modes

Every launch is either **Standard** or **Reward**.

**Standard tokens** carry no tax. The creator earns a share of trading fees instead — the total trading fee is 1.25%: 0.25% to Raydium's own protocol fee, 1% collected by StonkFun, half of which (0.5%) is forwarded to the creator automatically.

**Reward tokens** pay holders through a Token-2022 transfer tax — 1% or 3%, fixed permanently at launch — collected on every transfer, on any venue, not just trades on StonkFun. Reward tokens have no creator fee position at all.

The transfer tax model exists because a rewards mechanism funded by a share of trading fees has two structural weaknesses: the fee is attached to a single pool, and a competing pool for the same pair can undercut it elsewhere, routing volume — and the reward stream — away. A transfer tax avoids both, because it's a property of the token's mint rather than any one pool: it applies on every transfer regardless of venue, so there is no cheaper pool to route around it, and no live price oracle is required to apply it.

Tax doesn't distribute on every individual transfer — it accrues into a pot until the pot crosses a threshold, then converts and pays out to holders in a batch. That threshold scales with the token's market cap:

| Market cap | Threshold to trigger a payout |
|---|---|
| Under $50,000 | $50 |
| $50,000 – $100,000 | $200 |
| $100,000 – $125,000 | $250 |
| $125,000 – ~$50,000,000 | 0.1% of market cap |
| $50,000,000+ | Capped at $50,000 |

## Economics

Platform revenue is quote-token trading fees claimed to the treasury. Fees paid directly to creators or to reward-token holders don't count toward this — those are claimed separately, straight from Raydium, and never touch the treasury at all.

### $STONK buyback and burn

Most of that treasury goes toward one thing: buying $STONK back on the open market and burning it permanently. What's left over is kept as protocol revenue.

There's one clean exception to the buy-then-burn pattern. When a launch happens to be quoted against $STONK itself, there's no open-market purchase to make — the fee already arrives in $STONK — so it skips straight to burning instead of buying first.

$STONK itself isn't a special, separately-deployed asset — it went through StonkFun's own Standard-mode launch flow like anything else, paired against SPYX (a tokenized S&P 500 index) rather than SOL. Its supply is fixed permanently, and both its mint and freeze authority have been revoked for good — nobody can create more of it, and nobody can freeze anyone's holdings, StonkFun included.

### Ecosystem Flywheel

The Flywheel is a second, independent burn engine, and it doesn't touch the treasury above at all. Instead, it runs on a fixed 5% cut of trading fees taken directly from every Reward-mode pool — a stream the STONK buyback never sees.

What it does with that revenue: it continuously buys back and burns whichever tokens currently sit in the platform's own top 15 by market cap, weighting each buyback by size — a bigger token in that top 15 gets a bigger share of every round. And it doesn't run occasionally; it ticks every few minutes, all day.

The ranking is alive, not fixed. A token only gets bought back while it's actually sitting inside that top 15. Fall out of it, and the Flywheel simply stops touching that token — but nothing about its history is lost. Climb back in later, and it picks up again exactly where it left off, as if it never left.

Past these two engines, the platform separately tracks four smaller burn categories — Quote-revenue, Reward, Auto, and Kickstart — each with its own running total, though the exact trigger behind each one isn't spelled out anywhere public.

### Reward-token payouts

This one doesn't draw from platform revenue at all. It's funded entirely by the token's own transfer tax — and critically, that tax rate isn't something StonkFun sets or imposes. It's a choice the creator makes at the moment of launch: 1% or 3%, locked in permanently the instant the token exists. Pick reward mode, and you're deciding right then how hard that token will tax its own transfers to pay its holders.

Once that tax starts accruing, one wallet — operated by StonkFun, and it alone — holds the authority to pull it out of every Reward mint on the platform. It harvests whatever's accrued, sells it on the open market for that token's own quote asset, and pays the proceeds out to holders in batches.

## What StonkFun is not

StonkFun does not vet, audit, or endorse tokens launched on it. A tokenized-stock pairing does not make a token a stock, a derivative of stock, or an investment product — it only describes what the token trades against. Distributions to reward-token holders are a mechanical property of the token, not a dividend or yield.

## Next

- [How StonkFun Works](how-it-works/README.md) — launch venues, launch modes, fees, and economics in full detail
- [Getting Started](getting-started/README.md) — launching a first token
- [FAQ & Troubleshooting](faq/README.md) — common questions and error messages
