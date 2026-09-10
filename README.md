# Overview

StonkFun is a permissionless token launchpad on Solana built around one defining idea:

> Any token, paired against anything — a stock, a currency, a meme, or any other token.

The same launch mechanics and non-custodial guarantees apply regardless of what's on the other side of the pair — though the quote asset chosen directly determines the token's price behavior, since its value is permanently denominated in that asset.

## Architecture

StonkFun is three things stacked on top of each other:

1. **A launch venue abstraction.** New tokens can launch through a paid Raydium CLMM pool or a free Raydium LaunchLab bonding curve. A third venue, pump.fun, exists in the platform's history but is currently disabled. Every venue is presented identically — one token page, one chart, one fee ledger, one holder-rewards system — regardless of which one ran the launch.
2. **A curated market for quote assets.** Launches can be paired against any token on StonkFun's own list of 450+ Solana assets. That list is whitelisted by StonkFun, not self-service — see [Quote Assets](how-it-works/quote-assets.md) for what determines inclusion.
3. **A self-sustaining token economy.** Platform trading-fee revenue funds an automatic buyback-and-burn of $STONK, the platform's own token. A second, independent mechanism — the Ecosystem Flywheel — continuously buys back and burns the platform's own top tokens by market cap, funded separately from reward-token trading fees.

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

## Non-custodial design

There is no account, no API key, and no signup anywhere on the platform. A launch is authorized entirely by the creator's own wallet signing a fee payment; the platform never holds or signs with a private key on anyone's behalf. Fee claims work the same way — a claim transaction is only ever valid when signed by the wallet that actually owns it.

This extends to the launch mechanics themselves. A creator's opening buy (up to 50% of supply) is submitted in the same atomic bundle as the transaction that makes the pool tradeable — either the pool opens with the buy already filled, or the whole launch fails and nothing is charged. Airdrops work the same way: the recipient snapshot (up to 50% of supply, weighted by holding and capped at 1.5% per wallet) is frozen before the mint even exists, so nobody can see a new token coming and buy in early to qualify.

## Economics

Platform trading-fee revenue funds two independent burn mechanisms, not one:

- A **buyback-and-burn of $STONK**, historically around 60% of platform revenue, with the remainder retained.
- The **Ecosystem Flywheel**, funded separately from reward-token trading fees, which continuously buys back and burns the platform's own top 15 tokens by market cap.

$STONK itself was launched through StonkFun's own standard-mode flow, paired against SPYX (a tokenized S&P 500 index) rather than SOL. Its supply is fixed, and both mint and freeze authority are permanently revoked.

## What StonkFun is not

StonkFun does not vet, audit, or endorse tokens launched on it. A tokenized-stock pairing does not make a token a stock, a derivative of stock, or an investment product — it only describes what the token trades against. Distributions to reward-token holders are a mechanical property of the token, not a dividend or yield.

## Next

- [How StonkFun Works](how-it-works/README.md) — launch venues, launch modes, fees, and economics in full detail
- [Getting Started](getting-started/README.md) — launching a first token
- [FAQ & Troubleshooting](faq/README.md) — common questions and error messages
