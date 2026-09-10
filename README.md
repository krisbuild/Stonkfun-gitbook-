# Overview

StonkFun is a permissionless token launchpad on Solana built around one defining idea:

> Any token, paired against anything — a stock, a currency, a meme, or any other token.

The same launch mechanics and non-custodial guarantees apply regardless of what's on the other side of the pair — though the quote asset chosen directly determines the token's price behavior, since its value is permanently denominated in that asset.

## How a launch works

**1. Choose what the token is priced against, and how it pays.**

The quote asset comes from StonkFun's approved list of 450+ tokens. StonkFun controls that list; there is no way to add one yourself. The mode — Standard or Reward — is chosen at the same time and is permanent.

**2. The curve gets sized in the quote asset.**

Solana launchpads normally size a raise in SOL. StonkFun cannot, because the quote asset is arbitrary, so it converts: it looks up the quote asset's USD price and works out the amount that makes this launch worth the same as the platform's default 85 SOL raise.

A real quote, taken against SPYX (tokenized S&P 500) at $768.39:

| | |
|---|---|
| Raise | 11.4455166 SPYX |
| Starting market cap | $2,893 |
| Graduation market cap | $42,507 at the quoted price |
| Multiple to graduation | 14.7x |
| Supply | 1,000,000,000 (793,100,000 sold on the curve) |
| Curve | `ConstantCurve`, migrating to a Raydium CPMM pool |

This step is where launches fail. If the pricing service cannot value the quote asset, or the asset's supply is so large that the derived raise overflows, the launch is rejected before anything is signed. That is a property of the quote asset, not of the token being launched.

**3. The creator signs.**

On the paid Raydium path, StonkFun's API builds the transaction and the creator signs a fee payment from their own wallet. On LaunchLab there is no platform fee, and the creator can skip StonkFun's API entirely and build the instruction against Raydium's LaunchLab program directly, baking StonkFun's platform id into the pool. That id is the whole membership test: nothing else registers a token as a StonkFun launch.

There is no account, no API key and no signup anywhere in this. The platform never holds or signs with a key on anyone's behalf.

**4. The opening buy lands in the same bundle.**

A dev buy of up to 50% of supply is submitted in the same atomic Jito bundle as the transaction that makes the pool tradeable. Either the pool opens with the buy already filled, or the whole launch fails and nothing is charged. An airdrop snapshot, if used, is frozen before the mint exists — up to 50% of supply, weighted by holding and capped at 1.5% per wallet — so nobody can see a new token coming and buy in to qualify.

**5. The launch is adopted, and fees start flowing.**

If it came through the API it is already on record. If it was built independently, StonkFun reads the chain every minute for pools carrying its platform id and adopts it within a minute or two. An adopted launch is indistinguishable from one created through the API: token page, chart, volume, fee ledger, holder rewards. From there, every trade pays a fee, and those fees fund what follows.

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

{% hint style="warning" %}
These thresholds were provided directly by the StonkFun team and are not published in the public API or on the site. They are current as of September 2026; the team has indicated they plan to revise this system, so treat this table as a snapshot rather than a permanent specification.
{% endhint %}

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
