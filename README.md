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

**A launch that doesn't land doesn't charge.** If the bundle fails to land, the response comes back `service_unavailable` with nothing charged — the creator gets a fresh quote to retry, not a stuck payment.

**A bad quote asset is caught even earlier.** An ineligible or unusable quote asset is rejected at the very first step, before any transaction exists to sign — with a plain `invalid_request` error, not `service_unavailable`.

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

### Identify by mint address, not symbol

A ticker symbol (`SOL`, `ALON`, `USDC`) is just a label — nothing stops two different tokens from picking the same one. A mint address is the actual identifier: a long, unique string that can never collide.

Roughly 1 in 9 pairs on the list shares its symbol with at least one other pair. Two different tokens both trade as `ALON`. Separately, `xBTC` and `WBTC` both represent wrapped Bitcoin, just from different bridges — same idea, different tokens.

If code looks up a quote asset by its symbol instead of its mint address, it can silently grab the wrong one — no error, just the wrong token used. Always identify a quote asset by its mint address. The symbol is fine to show a person; it's the wrong thing for code to match on.

### How pricing works

Every quote asset is priced independently: its own live USD rate, converted using its own decimals, is what sizes a launch's curve. There's no single shared calculation across the list — each quote asset runs through this on its own, which is what lets StonkFun price a launch against anything from a stablecoin to a tokenized stock using the same underlying method.

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

## Tokenomics

| Field | Standard mode | Reward mode |
|---|---|---|
| **Starting supply** | 1,000,000,000 | 1,000,000,000 |
| **Decimals** | 6 | 9 |
| **Token standard** | SPL Token | Token-2022 |
| **Mint authority** | Revoked | Revoked |
| **Freeze authority** | Revoked | Revoked |
| **Supply can increase** | No | No |
| **Supply can decrease** | Yes — burns | Yes — burns |

### $STONK is an exception to this, not an example of it

| Field | $STONK |
|---|---|
| **Supply** | 852,067,953 |
| **Decimals** | 9 |
| **Token standard** | SPL Token (Standard mode) |
| **Mint authority** | Revoked |
| **Freeze authority** | Revoked |

$STONK's supply and decimals don't match the Standard-mode pattern above — it should not be read as a typical example of one.

## Launch venue

Every launch runs through LaunchLab. It costs network rent plus a minimal platform fee. The token starts on a bonding curve with no upfront liquidity: it trades against that curve until enough of the quote asset has been raised, then graduates automatically into a real Raydium pool.

A LaunchLab launch can also be built directly against Raydium's program without going through StonkFun's API at all — StonkFun scans the chain for pools carrying its platform id and adopts anything it finds within a minute or two.

## Graduation

Graduation is what happens once a bonding curve raises enough to stop being a bonding curve at all.

Every LaunchLab token starts trading purely against its own curve, with no upfront liquidity. Once its market cap crosses **$40,000**, it graduates: the platform migrates it into a real Raydium pool. Tokens get flagged "about to graduate" earlier, at **$32,000**, as an early signal before it actually happens. Progress toward that threshold is tracked continuously — every token carries a live progress value between 0 and 1 showing exactly how close it is.

Graduation is a real, permanent state change, not just a label. On-chain, the pool's own status flips the moment it happens — confirmed directly by comparing a still-bonding pool against a graduated one. The migration itself moves the curve's accumulated liquidity into a brand new Raydium pool, and the LP tokens that liquidity produces get locked permanently as part of that same step — not held by the creator, not held by the platform, not withdrawable by anyone, ever.

## Economics

Platform revenue is quote-token trading fees claimed to the treasury. Fees paid directly to creators or to reward-token holders don't count toward this — those are claimed separately, straight from Raydium, and never touch the treasury at all.

### $STONK buyback and burn

Most of that treasury goes toward one thing: buying $STONK back on the open market and burning it permanently. What's left over is kept as protocol revenue.

There's one clean exception to the buy-then-burn pattern. When a launch happens to be quoted against $STONK itself, there's no open-market purchase to make — the fee already arrives in $STONK — so it skips straight to burning instead of buying first.

$STONK itself isn't a special, separately-deployed asset — it went through StonkFun's own Standard-mode launch flow like anything else, paired against SPYX (a tokenized S&P 500 index) rather than SOL. Its supply is fixed permanently, and both its mint and freeze authority have been revoked for good — nobody can create more of it, and nobody can freeze anyone's holdings, StonkFun included.

### Ecosystem Flywheel

The Flywheel is a second, independent burn engine, and it doesn't touch the treasury above at all. Instead, it runs on a fixed 5% cut of trading fees taken directly from every Reward-mode pool — a stream the STONK buyback never sees.

What it does with that revenue: it continuously buys back and burns whichever tokens currently sit in the platform's own top 10 by market cap, weighting each buyback by size — a bigger token in that top 10 gets a bigger share of every round. And it doesn't run occasionally; it ticks every few minutes, all day.

The ranking is alive, not fixed. A token only gets bought back while it's actually sitting inside that top 10. Fall out of it, and the Flywheel simply stops touching that token — but nothing about its history is lost. Climb back in later, and it picks up again exactly where it left off, as if it never left.

### Reward-token payouts

This one doesn't draw from platform revenue at all. It's funded entirely by the token's own transfer tax — and critically, that tax rate isn't something StonkFun sets or imposes. It's a choice the creator makes at the moment of launch: 1% or 3%, locked in permanently the instant the token exists. Pick reward mode, and you're deciding right then how hard that token will tax its own transfers to pay its holders.

Once that tax starts accruing, one wallet — operated by StonkFun, and it alone — holds the authority to pull it out of every Reward mint on the platform. It harvests whatever's accrued, sells it on the open market for that token's own quote asset, and pays the proceeds out to holders in batches.

## Launching Your First Token

Quote asset and mode are already covered above — both are locked in the moment a launch is built. Name, symbol, logo, and a quote asset are all required; social links are optional, and skipping the website links the launch back to StonkFun by default.

**Dev buy.** Optionally buy into your own launch as part of the same landing — a target share of supply, or a SOL amount, never both. The cap is 75% of supply. It executes as the pool's literal first trade, so there's no window for anyone else to trade ahead of it. On a non-SOL quote asset, the SOL cost is worked out against that asset's live market at build time; on a Reward launch, what actually lands in the wallet is net of the transfer tax.

**Airdrop Mode (Reward launches only).** Optionally set aside a slice of supply — held out of the pool entirely — to distribute to existing holders of whatever asset is being paired against. The recipient list is built from that asset's own holder rankings (the top 100 by default), with exchange, custody, and program-owned wallets stripped out and backfilled so the drop still reaches its full size. That list is frozen the instant the launch is built — nobody can buy into the quote asset afterward hoping to catch the drop.

Once signed, the payment, mint, pool, liquidity, and any dev buy all land together as one atomic bundle. If it lands, the token is tradable immediately. If it doesn't, nothing is charged.

## Using the Public API

Everything on StonkFun — every token, every launch, every burn — is readable through one open REST API. No account, no signup, and for reading data, no key at all.

```
Base URL: https://www.stonkfun.xyz/api/public/v1
```

Every response is JSON. Every error follows the same shape:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "…",
    "retryable": false,
    "retryAfterSeconds": null
  }
}
```

`code` is always one of: `invalid_request`, `forbidden`, `not_found`, `method_not_allowed`, `conflict`, `rate_limited`, `internal`, `service_unavailable`. Requests are rate-limited per IP; a `rate_limited` response carries `retryAfterSeconds` telling you exactly how long to wait.

### Reading platform data

**`GET /tokens`** — every token with a live pool, with market data attached. Search by name, symbol, or mint with `q`; filter by `mode`, `status`, `quoteMint`, or `category`; sort with `sort` (defaults to market cap). Paginated with `page` and `pageSize` (up to 100 per page).

**`GET /tokens/{mint}`** — the same market data for one token, plus its original launch record.

**`GET /pairs`** — the full list of quote assets a launch can be built against. This is the one endpoint to check before building a launch: `quoteMint` has to be something this returns. Filter with `launchable` (excludes retired assets) or `launchLabReady` (only assets Raydium has actually provisioned on-chain).

**`GET /launches`** — the launch ledger, newest first. Filter by `creator` to pull up everything one wallet has launched, or by `mode`. Launches built directly against the chain, without going through this API at all, show up here too once adopted, under `launchpad: "launchlab"`.

**`GET /stats`** — aggregate platform totals, plus the config flags a client actually needs: `config.paidLaunchesEnabled` and `config.launchLabEnabled` say which launch path is currently live. Worth checking before showing a "Launch" button at all.

**`GET /revenue`** — fee revenue, buybacks, and burns.

**`GET /revenue/history`** — the whole daily revenue history in one call, keyed by UTC day. No date-range parameters by design — fetch it once and index it locally rather than re-querying by date.

### Reading one token in depth

A handful of endpoints go deeper than what `/tokens/{mint}` returns:

**`GET /tokens/{mint}/burns`** — totals and recent burns of this specific token by the platform's fee sweep.

**`GET /tokens/{mint}/rewards`** — for a Reward-mode token, lifetime payout totals and holder count. Standard-mode tokens answer with `mode: "standard"` and a null rewards object — not an error, just nothing to report.

**`GET /tokens/{mint}/airdrop`** — the airdrop a token launched with, if it had one. This reads the frozen snapshot taken at launch time, not a live balance scan, so it reflects exactly what was actually paid for. Most tokens have no airdrop at all, which comes back as `airdrop: null`.

**`GET /tokens/{mint}/backing`** — USD value permanently locked behind a token. This doesn't apply to any current launches — calling it on a real token returns `400 invalid_request` rather than a zero, since the concept doesn't apply to how tokens are launched today.

**`GET /tokens/{mint}/fees`** — trading fees a creator can currently claim, without needing a connected wallet to ask. Whether anything is claimable, and why, depends entirely on how the token was launched: a Standard-mode LaunchLab launch has its creator share forwarded automatically, so there's nothing to claim by signature at all; a Reward-mode launch pays holders through the transfer tax instead and gives its creator nothing. Both cases answer `claimable: null` with a reason, not an error.

### Launching a token

**`POST /launches/prepare`** — validates everything and returns a signed quote plus an unsigned payment transaction.

**`POST /launches/submit`** — takes the signed transaction back and lands the whole launch as one atomic bundle.

**`GET /launches/{paymentSignature}`** — poll this if `/submit` came back `processing`, until it flips to `completed`.

## Network & Program IDs

| Field | Value |
|---|---|
| **Network** | Solana (mainnet-beta) |
| **LaunchLab program** (bonding curve, every launch) | `LanMV9sAd7wArD4vJFi2qDdfnVhFxYSUg6eADduJ3uj` |
| **Raydium CPMM program** (graduation destination) | `CPMMoo8L3F4NbTegBCKVNunggL7H1ZpdTHKxQB5qKP1C` |
| **SPL Token program** (Standard-mode mints) | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` |
| **Token-2022 program** (Reward-mode mints) | `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` |
| **StonkFun platform config — Standard** | `4E876qZTE9FJMrBzgVtBrSrzz2TLivB5Y5QXPjB4gZL7` |
| **StonkFun platform config — Reward** | `6BwHHDg3u1854jC8PDLXvR4spTcLNaoBxLJNGC4nTESt` |
| **Reward-tax withdraw authority** | `5KXDF6QnqhBj72hDtJNkkpFaQVUfbFXNybMsp3DiK6tD` |

Every value here is live and verifiable — the LaunchLab and platform config IDs come straight from `GET /launchlab/pricing`, and the withdraw authority is the wallet actually observed on-chain harvesting transfer tax from Reward-mode mints.

## What StonkFun is not

StonkFun does not vet, audit, or endorse tokens launched on it. A tokenized-stock pairing does not make a token a stock, a derivative of stock, or an investment product — it only describes what the token trades against. Distributions to reward-token holders are a mechanical property of the token, not a dividend or yield.
