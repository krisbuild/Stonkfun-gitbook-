# About StonkFun

StonkFun is a permissionless token launchpad on Solana with one defining idea: **every token is priced against another token, not just SOL.** You can launch a coin paired with a meme, a tokenized stock, a stablecoin, a leveraged asset, or almost anything else already on Solana.

That single design choice is why StonkFun looks different from a typical launchpad, and it's the thread running through this whole guide.

{% hint style="info" %}
Everything in this documentation is sourced directly from StonkFun's public API, its live on-chain data, and its own site — verified, not guessed. Where something couldn't be confirmed, it's marked as such rather than assumed.
{% endhint %}

## What StonkFun actually is

At its core, StonkFun is three things stacked on top of each other:

1. **A launch venue abstraction.** New tokens can launch through a paid Raydium CLMM pool, a free Raydium LaunchLab bonding curve, or (legacy, currently disabled) pump.fun — and the platform presents all three the same way: one token page, one chart, one fee ledger, one holder-rewards system, regardless of which venue actually executed the launch.
2. **A curated market for quote assets.** Anyone can pick from over 450 existing Solana tokens to pair a new launch against — spanning tokenized stocks, pre-IPO equity, stablecoins, leveraged tokens, and ordinary memecoins. This list is maintained by StonkFun itself; it isn't something you can add to yourself (more on that in [Quote Assets](how-it-works/quote-assets.md)).
3. **A self-sustaining token economy.** Trading fees collected across the platform fund an automatic buyback-and-burn of StonkFun's own token, **$STONK**, plus a second, independent mechanism — the Ecosystem Flywheel — that continuously buys back and burns the platform's own top tokens by market cap.

## Quick facts

| | |
|---|---|
| **Chain** | Solana (mainnet-beta) |
| **Base API URL** | `https://www.stonkfun.xyz/api/public/v1` |
| **API key required** | None — every endpoint is open, rate-limited per IP |
| **Tokens launched to date** | 29,000+ and climbing at roughly 6–7 new launches per minute |
| **Quote assets available** | 450+ across 9 categories |
| **Platform token** | $STONK — fixed supply, no mint or freeze authority |

## What makes it unique

**Pairing against anything, not just SOL.** Most launchpads give you one choice of quote asset. StonkFun's pairs list spans real tokenized equities (xStock, PreStock, Tessera, Sunrise), stablecoins, leveraged tokens, and hundreds of ordinary memecoins — all selectable from the same launch flow. The "trade against a stock" angle gets most of the attention, but it's a minority of actual usage: roughly 12% of listed pairs are equity-flavored, the rest are everyday memecoins.

**Two ways to launch, and one of them is free.** A launch can go through StonkFun's paid Raydium CLMM flow, or through **LaunchLab**, a bonding-curve venue with **no platform fee at all** — you only pay Solana's own network rent. LaunchLab launches also don't require going through StonkFun's API in the first place: you can build the transaction yourself against Raydium's program directly, and StonkFun's own scanner adopts it automatically within a minute or two, granting it the exact same token page, fee forwarding, and holder rewards as a launch created through the API.

**Non-custodial by construction, not just by claim.** There's no API key and no signup, because a launch is authorized by your own wallet signing the fee payment — the platform never asks for, holds, or signs with a private key on your behalf. This extends to fee claiming too: a claim transaction is only ever valid when signed by the wallet that actually owns it.

**Two fundamentally different reward models exist**, and mixing them up is the single easiest mistake to make:

- **Standard tokens** carry no tax at all; the creator earns a share of trading fees instead.
- **Reward tokens** pay holders automatically through a **Token-2022 transfer tax** — 1% or 3%, fixed permanently at launch — collected on every transfer, on any venue, not just trades on StonkFun. There's no creator fee position on a reward token at all.

**A front-runproof dev buy and airdrop.** A creator's opening buy is submitted in the same atomic bundle as the transaction that makes the pool tradeable — either the pool opens with the buy already filled, or the whole launch fails and nothing is charged. Airdrops work the same way: the recipient snapshot is frozen *before the mint even exists*, so nobody can see a new token coming and buy in early to qualify for the drop.

**Two independent burn mechanisms, not one.** Platform trading-fee revenue funds a buyback-and-burn of $STONK (roughly 60% of revenue, historically). Separately, the **Ecosystem Flywheel** takes a slice of every reward-token's trading fees and uses it to buy back and burn the platform's own top 15 tokens by market cap, on a rolling basis — a mechanism most competing platforms don't have at all.

**$STONK was launched on StonkFun itself.** The platform's own token isn't a special pre-mine — it was created through the same standard-mode launch flow anyone can use, paired against SPYX (a tokenized S&P 500 index), with a fixed supply and both mint and freeze authority permanently revoked.

## What StonkFun is not

Worth stating plainly, because the platform's own Terms of Service are unusually direct about this: StonkFun does not vet, audit, or endorse any token launched on it. A tokenized-stock pairing does not make a token stock, a derivative of stock, or an investment product of any kind — it only describes what the token trades against. Distributions to reward-token holders are a mechanical property of the token, not a dividend or yield. If you're building on top of StonkFun or explaining it to someone else, these distinctions are worth carrying forward accurately rather than smoothing over.

## Where to go next

- **[How StonkFun Works](how-it-works/README.md)** — launch venues, launch modes, fees, and the economics in full detail
- **[Getting Started](getting-started/README.md)** — a walkthrough of launching your first token
- **[FAQ & Troubleshooting](faq/README.md)** — real, verified answers to the questions people actually ask
