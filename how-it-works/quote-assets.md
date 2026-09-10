# Quote Assets & Categories

A quote asset is what a new token is priced against — the other side of the pair. StonkFun's central claim is that this can be almost anything: not just SOL, but a tokenized stock, a stablecoin, a commodity, or an ordinary memecoin. This page is the detail behind that claim: what the approved list actually contains, how it's organized, and where it breaks.

## The list is curated, not open

There is no way to add a quote asset yourself. Every asset on the list was added by StonkFun; a token you hold has no path onto the list just because you want to launch against it. The list is also not fixed — new assets are added on an ongoing basis — but growth only ever comes from StonkFun's side.

Before building a launch, the API's guidance is to call `/pairs` first: `quoteMint` must be one of the assets it returns.

## The nine categories

Based on a snapshot of the live `/pairs` response (453 quote assets, September 2026):

| Category | Label shown | Count | Share | Examples |
|---|---|---|---|---|
| `custom` | Custom | 387 | ~85% | Ordinary memecoins and established Solana tokens |
| `xstock` | xStock | 24 | ~5% | SPYX (S&P 500), NVDAX (NVIDIA), TSLAX (Tesla), GOOGLX (Google), COINX (Coinbase), MSTRX (MicroStrategy) |
| `backpack` | **Sunrise** | 22 | ~5% | MU (Micron), TTWO (Take-Two), SNDK (SanDisk), NBIS, SILVER |
| `prestock` | PreStock | 7 | ~1.5% | ANTHROPIC, ANDURIL, NEURALINK, POLYMARKET, FIGUREAI, OPENAI, KALSHI |
| `currency` | Currency | 5 | ~1% | USDC, USDT, EURC, ONYC, JLUSDC |
| `tessera` | Tessera | 2 | <1% | OPENAI, KALSHI |
| `leverage` | Leverage | 2 | <1% | xSOL, XBTC |
| `solana` | Solana | 2 | <1% | SOL (Wrapped), SKR |
| `collectible` | Collectibles | 2 | <1% | SV151, HEEBOO |

*Counts and shares are a point-in-time snapshot. The category breakdown shifts as the list grows — don't treat these as fixed.*

Two things worth noticing in the raw data:

**The category id and its display label don't always match.** The API returns `backpack` as the raw `category` value, but the site displays it as "Sunrise." Anyone reading the API directly rather than the UI needs to know this mapping, or a `backpack`-category pair will look unlabeled.

**Some symbols exist twice, in different categories, with different statuses.** OPENAI and KALSHI each appear as two separate quote-asset entries: once under `prestock` (active), and once under a now-retired `tessera` category. Both `tessera` entries are correctly flagged as no longer launchable — evidence the platform has consolidated categories over time rather than keeping every one live forever.

## The two gates a pair has to clear

The API tracks two independent flags per pair, and both have to be true for a launch to actually work:

**`launchable`** — StonkFun's own rule. A pair StonkFun has retired (like the `tessera` duplicates above) reports `false` here, permanently.

**`launchLabReady`** — whether Raydium itself has created the on-chain `GlobalConfig` a LaunchLab launch needs against that specific quote asset. A pair can be `launchable: true` and still not be LaunchLab-ready — StonkFun approves it, but Raydium's on-chain side isn't provisioned for it yet. Constructing a LaunchLab launch against one of those fails on-chain, not at the API layer.

In the same snapshot, three `custom`-category pairs (PENGUIN, PUMPCADE, BURNIE) were exactly in that state: approved by StonkFun, not yet usable on LaunchLab. If you're building a LaunchLab launch directly against Raydium rather than through StonkFun's API, checking `launchLabReady` first is the difference between a clean launch and a failed on-chain transaction.

## Symbols collide — always match by mint address

Roughly 1 in 9 pairs in the list shares its ticker symbol with at least one other pair. Some examples from the snapshot: two different tokens both trade as `ALON`, `KALSHI` appears under two categories, `xBTC` and `WBTC` both describe wrapped Bitcoin from different bridges, and `CATE` and `STONK` each have look-alike entries elsewhere in the list.

The API surfaces a `symbolAmbiguous` flag on pairs affected by this — undocumented in the published spec, but observably true exactly on entries with a duplicate-looking ticker. Whatever the field is officially for, the practical rule holds regardless: **never resolve a quote asset by symbol.** Always use the mint address. A UI or integration built around ticker text will occasionally pick the wrong token.

## Where pricing actually breaks

Every quote asset needs a live, reliable USD price for StonkFun to size a launch's curve (see [How launches work](../README.md#how-launches-work)). Two real failure classes have shown up on the platform:

**Supply overflow.** A quote or base asset with an extremely large raw supply can push the derived raise past what the underlying math can represent, and the launch is rejected with a pricing error before anything is signed. This isn't a liquidity problem — it's an arithmetic one, tied to the asset's decimals and total supply, not its market cap.

**Per-asset pricing outages.** The pricing service prices each quote asset independently. One asset can go down while every other asset on the list keeps working normally — this has happened to a single pairing while the rest of the platform launched without issue. When this happens it's a problem with that specific asset's price feed, not with the launch system as a whole.

Both failure modes are asset-specific, not systemic — they affect the one quote asset in question, not the platform's ability to price everything else.
