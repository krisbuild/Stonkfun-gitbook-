# Overview

StonkFun is a permissionless token launchpad on Solana built around one defining idea: every token is priced against another token, not just SOL. A launch can be paired with a meme, a tokenized stock, a stablecoin, a leveraged asset, or almost anything else already on Solana.

## What StonkFun is

StonkFun is three things stacked on top of each other:

1. **A launch venue abstraction.** New tokens can launch through a paid Raydium CLMM pool, a free Raydium LaunchLab bonding curve, or (legacy, currently disabled) pump.fun. All three are presented identically: one token page, one chart, one fee ledger, one holder-rewards system, regardless of which venue executed the launch.
2. **A curated market for quote assets.** Launches can be paired against any of 450+ existing Solana tokens — tokenized stocks, pre-IPO equity, stablecoins, leveraged tokens, and ordinary memecoins. This list is maintained by StonkFun; see [Quote Assets](how-it-works/quote-assets.md) for how a token qualifies.
3. **A self-sustaining token economy.** Platform trading-fee revenue funds an automatic buyback-and-burn of $STONK, StonkFun's own token. A second, independent mechanism — the Ecosystem Flywheel — continuously buys back and burns the platform's own top tokens by market cap.

## Quick facts

| | |
|---|---|
| **Chain** | Solana (mainnet-beta) |
| **Base API URL** | `https://www.stonkfun.xyz/api/public/v1` |
| **API key** | None required — every endpoint is open, rate-limited per IP |
| **Tokens launched** | 29,000+, growing at roughly 6–7 launches per minute |
| **Quote assets** | 450+ across 9 categories |
| **Platform token** | $STONK — fixed supply, no mint or freeze authority |

## What makes it distinct

**Pairing against anything, not just SOL.** The pairs list spans tokenized equities (xStock, PreStock, Tessera, Sunrise), stablecoins, leveraged tokens, and hundreds of ordinary memecoins, all selectable from the same launch flow. Equity-flavored pairs are a minority of actual usage — roughly 12% of listed pairs — the rest are everyday memecoins.

**Two launch venues, one of them free.** Raydium CLMM is the paid path. **LaunchLab** is a bonding-curve venue with no platform fee — only Solana's own network rent. LaunchLab launches don't require StonkFun's API at all: a transaction built directly against Raydium's program is adopted automatically within a minute or two, receiving the same token page, fee forwarding, and holder rewards as a launch created through the API.

**Non-custodial by construction.** No API key, no signup — a launch is authorized by the creator's own wallet signing the fee payment, and the platform never holds or signs with a private key on a user's behalf. Fee claims work the same way: a claim transaction is only valid when signed by the wallet that owns it.

**Two reward models, structurally different:**

- **Standard tokens** carry no tax; the creator earns a share of trading fees instead.
- **Reward tokens** pay holders through a Token-2022 transfer tax — 1% or 3%, fixed permanently at launch — collected on every transfer, on any venue. Reward tokens have no creator fee position.

The transfer tax model exists because a rewards mechanism funded by a share of trading fees has two weaknesses: the fee is attached to a single pool, and a competing pool for the same pair can undercut it elsewhere, routing volume — and the reward stream — away. A transfer tax is a property of the token's mint rather than any one pool. It applies on every transfer regardless of venue, so there is no cheaper pool to route around it, and no live price oracle is required to apply it.

**Atomic dev buys and airdrops.** A creator's opening buy is submitted in the same bundle as the transaction that makes the pool tradeable — either the pool opens with the buy filled, or the launch fails and nothing is charged. Airdrop recipient snapshots are frozen before the mint exists, so nobody can see a new token coming and buy in early to qualify.

**Two independent burn mechanisms.** Platform trading-fee revenue funds a buyback-and-burn of $STONK. Separately, the Ecosystem Flywheel takes a share of reward-token trading fees and uses it to buy back and burn the platform's own top 15 tokens by market cap.

**$STONK was launched on StonkFun itself**, through the same standard-mode flow available to anyone, paired against SPYX (a tokenized S&P 500 index). Its supply is fixed, with both mint and freeze authority permanently revoked.

## What StonkFun is not

StonkFun does not vet, audit, or endorse tokens launched on it. A tokenized-stock pairing does not make a token a stock, a derivative of stock, or an investment product — it only describes what the token trades against. Distributions to reward-token holders are a mechanical property of the token, not a dividend or yield.

## Next

- [How StonkFun Works](how-it-works/README.md) — launch venues, launch modes, fees, and economics
- [Getting Started](getting-started/README.md) — launching a first token
- [FAQ & Troubleshooting](faq/README.md) — common questions and error messages
