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

**Second, it has to be approved.** Clearing the liquidity bar makes a token *eligible* — it doesn't put it on the list automatically. Approval is a separate decision: StonkFun choosing that specific token for the list. That decision is exposed in the API as a single field, `launchable`. `true` means it's on the list and currently usable; `false` marks a retired pair — one that was approved at some point and has since been taken off the list. The Tessera duplicates covered above are exactly this case: approved once, then permanently switched off.

So the sequence runs: eligible (clears the liquidity bar) → chosen (StonkFun approves it) → technically ready (Raydium has provisioned it on-chain, covered next). A token can clear the first and third and still never make the list, because the middle step is a deliberate choice, not an automatic pass.

**Third, if the launch runs on LaunchLab, Raydium has to have separately provisioned that asset on-chain.** Approval alone doesn't create this — Raydium's own on-chain `GlobalConfig` for that specific quote asset has to already exist, independent of the approval decision itself. The API reports this as `launchLabReady`.

A pair can clear approval without clearing provisioning: approved, but not yet ready on Raydium's side for LaunchLab specifically. In one snapshot, three approved pairs — PENGUIN, PUMPCADE, and BURNIE — were caught exactly there: listed and approved, but a LaunchLab launch against any of them would fail on-chain rather than at the API level.

### Symbols collide — match by mint address

Roughly 1 in 9 pairs shares its ticker symbol with at least one other — two different tokens both trade as `ALON`, `xBTC` and `WBTC` both describe wrapped Bitcoin from different bridges. Never resolve a quote asset by symbol; always use the mint address, or an integration will occasionally pick the wrong token.

### Where pricing actually breaks

Every quote asset needs a live, reliable USD price to size a launch's curve. Two real failure classes have shown up on the platform: **supply overflow**, where an asset's raw supply is large enough to push the derived raise past what the math can represent, rejecting the launch before anything is signed; and **per-asset pricing outages**, where the pricing service goes down for one specific asset while every other asset on the list keeps working normally. Both are asset-specific, not systemic.

## Launch venues

| | Raydium CLMM | LaunchLab |
|---|---|---|
| **Cost** | A flat SOL fee, shown live before signing | Network rent only (~0.012–0.013 SOL) — no platform fee |
| **Liquidity model** | Locked position from launch | Bonding curve, no upfront liquidity, graduates into a Raydium pool |
| **Requires StonkFun's API** | Yes | No — a transaction built directly against Raydium's program is adopted automatically within a minute or two |
| **Fee claiming** | Fee Key NFT + manual claim | Forwarded automatically, nothing to claim |

Where there is something to claim, the claim is only valid when signed by the wallet that owns the position. A claim transaction prepared for a token you do not own cannot be used.

## Launch modes

Every launch is either **Standard** or **Reward**, independent of which venue it runs through.

**Standard tokens** carry no tax. The creator earns a share of trading fees instead. On LaunchLab, the total trading fee is 1.25% — 0.25% to Raydium's own protocol fee, 1% collected by StonkFun, half of which (0.5%) is forwarded to the creator automatically. On the paid Raydium path, the creator chooses a 1% pool (50/50 split) or a 2% pool (75% to the creator) at launch, permanently.

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

Platform trading-fee revenue funds two independent burn mechanisms, not one:

- A **buyback-and-burn of $STONK**, historically around 60% of platform revenue, with the remainder retained.
- The **Ecosystem Flywheel**, funded separately from reward-token trading fees, which continuously buys back and burns the platform's own top 15 tokens by market cap.

$STONK itself was launched through StonkFun's own standard-mode flow, paired against SPYX (a tokenized S&P 500 index) rather than SOL. Its supply is fixed, and both mint and freeze authority are permanently revoked.

Reward-token payouts are funded by a third source again: the transfer tax itself, not trading-fee revenue. A single wallet operated by StonkFun holds the authority to withdraw accrued tax from every Reward mint on the platform; it harvests, sells into the quote asset, and batches the proceeds out to holders.

## What StonkFun is not

StonkFun does not vet, audit, or endorse tokens launched on it. A tokenized-stock pairing does not make a token a stock, a derivative of stock, or an investment product — it only describes what the token trades against. Distributions to reward-token holders are a mechanical property of the token, not a dividend or yield.

## Next

- [How StonkFun Works](how-it-works/README.md) — launch venues, launch modes, fees, and economics in full detail
- [Getting Started](getting-started/README.md) — launching a first token
- [FAQ & Troubleshooting](faq/README.md) — common questions and error messages
