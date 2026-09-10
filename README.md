# Introduction

A launchpad is usually judged on one thing: how fast it gets a new coin trading against SOL. That's the wrong question. The more interesting one is what a coin gets to trade against at all.

StonkFun starts from a different premise — that a new token doesn't have to be priced against SOL to exist. It can be priced against a tokenized stock, a stablecoin, a leveraged asset, an index, or another coin entirely, using the same launch mechanics and the same guarantees, on the same chain.

> Every token is priced against another token. Not just SOL.

That one decision shapes everything else about how the protocol is built. Supporting hundreds of quote assets instead of one means solving problems a single-asset launchpad never has to face — pricing curves correctly across wildly different token decimals, keeping non-custodial guarantees intact no matter which venue a launch runs through, and paying holders in a way that holds up at scale instead of quietly decaying. Each of these has a specific, deliberate answer built into the protocol.

**Three ideas hold it together:**

**Launch anywhere, look the same everywhere.** A token can launch through a paid Raydium pool, a free bonding curve, or be built independently and adopted automatically after the fact — and it ends up with the same token page, the same fee ledger, and the same holder-reward mechanics regardless of which path it took.

**Nothing is held, nothing is asked for.** No account. No API key. No signup. A launch is authorized by a wallet signature on a fee payment; a fee claim is authorized by the wallet that owns it. The protocol never custodies a key and never signs on anyone's behalf.

**Rewards that don't decay.** A holder-reward model funded by a share of trading fees erodes as a token grows and can be routed around by a competing pool elsewhere. StonkFun pays holders through a transfer tax instead — a property of the token itself, applied on every transfer, on any venue, that can't be undercut and doesn't depend on a live price oracle to keep working.

The [Overview](overview.md) covers the concrete mechanics, numbers, and structure behind each of these.
