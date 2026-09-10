# Overview

StonkFun is a permissionless token launchpad on Solana built around one defining idea:

> Any token, paired against anything — a stock, a currency, a meme, or any other token.

The same launch mechanics and non-custodial guarantees apply regardless of what's on the other side of the pair — though the quote asset chosen directly determines the token's price behavior, since its value is permanently denominated in that asset.

## Architecture

A launch moves through three stages, in this order.

**1. A quote asset is chosen.** Before anything else, a creator picks a token from StonkFun's own curated list of 450+ Solana assets — whitelisted by StonkFun, with no public process to add to it. Roughly 80% of what's on that list carries a visible provenance tag: already verified on Jupiter's token list, already launched on pump.fun, or already launched on StonkFun itself. This choice isn't cosmetic — it fixes the currency the new token is priced in for the rest of its life, and its own volatility becomes the new token's volatility by construction.

**2. The launch executes.** The chosen quote asset then has to survive real curve math. Execution runs through one of two live venues — a paid Raydium CLMM pool, or a free LaunchLab bonding curve — and both have to size themselves correctly against the quote asset's actual decimals and price. This is where stage 1's choice can fail outright: a small number of extreme-supply or oracle-fragile quote assets break this step regardless of which venue is used, independent of anything the creator does right. A launch doesn't need StonkFun's own API to reach this stage, either — a transaction built independently against Raydium's LaunchLab program is detected and adopted automatically within a minute or two, ending up with the same token page, fee ledger, and reward mechanics as one created through the API.

**3. Trading generates economics.** Every trade through either venue produces fee revenue — covered in full under [Economics](#economics).

## By the numbers

| Field | Value |
|---|---|
| **Chain** | Solana (mainnet-beta) |
| **Base API URL** | `https://www.stonkfun.xyz/api/public/v1` |
| **API key** | None required — every endpoint is open, rate-limited per IP |
| **Tokens launched** | 29,000+, growing at roughly 6–7 launches per minute |
| **Quote assets** | 450+ across 9 categories |
| **Graduation threshold** | $40,000 market cap (configured); tokens are flagged "about to graduate" at $32,000 |
| **Platform token** | $STONK — fixed supply, mint and freeze authority both permanently revoked |

**Quote asset categories**, by approximate share of the pairs list:

| Category | What it is | Share |
|---|---|---|
| Custom | Ordinary memecoins and established Solana tokens | ~85% |
| xStock | Tokenized public-company stocks | ~5% |
| Sunrise | A second tokenized-equity category | ~5% |
| PreStock | Tokenized pre-IPO / private companies | ~1.5% |
| Currency | Stablecoins and fiat-pegged tokens | ~1% |
| Leverage, Collectibles, Solana, Tessera | Amplified-exposure tokens, tokenized collectibles, SOL and related core assets, and a second pre-IPO issuer | remainder |

Ordinary memecoins dominate the pairs list. The tokenized-equity categories combined are a small minority of what's actually launchable — the "trade against a stock" framing is a small slice of real usage.

## Launch venues

| | Raydium CLMM | LaunchLab |
|---|---|---|
| **Cost** | A flat SOL fee, shown live before signing | Network rent only (~0.012–0.013 SOL) — no platform fee |
| **Liquidity model** | Locked position from launch | Bonding curve, no upfront liquidity, graduates into a Raydium pool |
| **Requires StonkFun's API** | Yes | No — a transaction built directly against Raydium's program is adopted automatically within a minute or two |
| **Fee claiming** | Fee Key NFT + manual claim | Forwarded automatically, nothing to claim |

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

{% hint style="warning" %}
These thresholds were provided directly by the StonkFun team and are not published in the public API or on the site. They are current as of September 2026; the team has indicated they plan to revise this system, so treat this table as a snapshot rather than a permanent specification.
{% endhint %}

## Non-custodial design

There is no account, no API key, and no signup anywhere on the platform. A launch is authorized entirely by the creator's own wallet signing a fee payment; the platform never holds or signs with a private key on anyone's behalf. Fee claims work the same way — a claim transaction is only ever valid when signed by the wallet that actually owns it.

This extends to the launch mechanics themselves. A creator's opening buy (up to 50% of supply) is submitted in the same atomic bundle as the transaction that makes the pool tradeable — either the pool opens with the buy already filled, or the whole launch fails and nothing is charged. Airdrops work the same way: the recipient snapshot (up to 50% of supply, weighted by holding and capped at 1.5% per wallet) is frozen before the mint even exists, so nobody can see a new token coming and buy in early to qualify.

## Economics

Platform trading-fee revenue funds two independent burn mechanisms, not one:

- A **buyback-and-burn of $STONK**, historically around 60% of platform revenue, with the remainder retained.
- The **Ecosystem Flywheel**, funded separately from reward-token trading fees, which continuously buys back and burns the platform's own top 15 tokens by market cap.

$STONK itself was launched through StonkFun's own standard-mode flow, paired against SPYX (a tokenized S&P 500 index) rather than SOL. Its supply is fixed, and both mint and freeze authority are permanently revoked.

Reward-token payouts run on a separate, third mechanism: a single wallet, operated by StonkFun, holds the authority to withdraw accrued transfer tax off every reward-mode mint on the platform. It harvests each token's accumulated tax, sells it on the open market for the token's quote asset, and batches the proceeds out to holders — running continuously, across every reward token on the platform, at once, rather than as a per-token process each creator sets up individually.

## What StonkFun is not

StonkFun does not vet, audit, or endorse tokens launched on it. A tokenized-stock pairing does not make a token a stock, a derivative of stock, or an investment product — it only describes what the token trades against. Distributions to reward-token holders are a mechanical property of the token, not a dividend or yield.

## Next

- [How StonkFun Works](how-it-works/README.md) — launch venues, launch modes, fees, and economics in full detail
- [Getting Started](getting-started/README.md) — launching a first token
- [FAQ & Troubleshooting](faq/README.md) — common questions and error messages
