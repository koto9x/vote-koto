# Quadratic voting tools for KOTOPIA's anti-whale governance

**Realms with its native quadratic voting plugin is KOTOPIA's best option — it's the only production-ready, Solana-native QV implementation and can go live within hours.** Snapshot, the dominant off-chain voting platform, does not support Solana at all. Among dozens of tools researched, only Realms delivers quadratic voting on Solana with built-in Sybil resistance and a specific mechanism for neutralizing disproportionate whale holdings. The setup is no-code, free, and battle-tested across 97% of Solana DAOs managing **$1.5B+ in treasury value**.

---

## Snapshot cannot be used — it's EVM-only

Snapshot.org is the gold standard for off-chain DAO voting (**96% of major DAO votes** flow through it), but it is fundamentally incompatible with Solana. Every voting strategy reads balances from EVM smart contracts via `balanceOf()` calls. Creating a space requires an **ENS domain on Ethereum mainnet**. Wallets must be MetaMask or similar EVM wallets — Phantom and Solflare won't work.

This isn't a theoretical limitation. When **Render Network** migrated to Solana, they explicitly abandoned Snapshot for a Solana-native platform, noting "ETH voting through Snapshot will discontinue." The only workaround — exporting SPL token holders into a whitelist strategy requiring voters to also connect Ethereum wallets — adds so much friction it's impractical for a quick community vote.

**That said, Snapshot's QV mechanics are worth understanding** because they inform how Realms works. Snapshot treats Quadratic Voting as a *voting type* (how outcomes are calculated), separate from *voting strategies* (how raw voting power is determined). The formula is more sophisticated than simple `sqrt(tokens)`: for each choice, Snapshot sums the square roots of every individual voter's contribution to that choice, then squares the total. This means **three small voters with 1 token each collectively outweigh a single voter holding 9 tokens** — the number of supporters matters more than the size of their holdings. Snapshot also offers an `anti-whale` strategy that applies `inflectionPoint × (balance/inflectionPoint)^0.5`, which compresses whale power at the strategy level before QV even kicks in.

---

## Realms delivers exactly what KOTOPIA needs

Realms has a dedicated **Quadratic Voting plugin** with the formula `voting_power = a × √(tokens) + b × tokens + c`, where default coefficients (a=1, b=0, c=0) produce pure square-root weighting. A wallet holding **1,000,000 tokens gets 1,000 voting power** while a wallet with 100 tokens gets 10 — compressing the ratio from 10,000:1 down to 100:1.

Three features make Realms uniquely suited to KOTOPIA's whale problem:

**Circulating supply factor.** The QV plugin allows explicitly adjusting for known whale holdings. The docs state: *"If 25% of the total supply is held by a single entity that votes in a block, you can set the circulating supply factor to 75% + the square root of the number of tokens held by the entity."* This directly addresses a protocol-owned wallet holding disproportionate supply.

**Civic Pass requirement.** Quadratic voting on Realms **mandates Civic Pass** integration for Sybil resistance. Without it, a whale could simply split tokens across hundreds of wallets to game the square-root formula. Civic Pass verifies each voter is a unique individual — configurable from a simple CAPTCHA/liveness check up to full KYC. This is the critical anti-gaming layer that makes QV actually work.

**Treasury wallets are excluded by design.** Tokens held in a DAO's native treasury PDA are not associated with a token owner record and cannot vote. If KOTOPIA's protocol-owned tokens sit in the DAO treasury, they're automatically excluded — no configuration needed.

Setup takes **hours, not days**. Go to app.realms.today → Create DAO → Community Token DAO → Advanced Options → Quadratic → enable Civic Pass chaining → configure your SPL token → launch. The entire process is no-code. Transaction fees are negligible (~$0.0025 per vote). The SPL Governance program must be version ≥2.2.6, which is standard for new DAOs.

---

## How every other platform stacks up

No other tool matches Realms for this specific use case, but several deserve consideration depending on KOTOPIA's priorities:

**MetaDAO (Solana-native futarchy)** offers the most radical anti-whale approach. Instead of voting, decisions are made through prediction markets — traders bet on whether a proposal will increase or decrease the token's value. If PASS tokens outperform FAIL tokens by ≥3%, the proposal passes. Whale manipulation creates profitable counter-trading opportunities, making it **inherently self-correcting**. Used by Sanctum, Drift, and Jito. The tradeoff: it's conceptually complex, requires market liquidity, and may confuse a community expecting a simple vote.

**Tally** recently launched MultiGov with Solana support (Wormhole DAO is the first adopter), but it **does not support quadratic voting** — only standard Governor-style token-weighted voting with delegation. Setup takes days to weeks and is enterprise-oriented.

**Commonwealth** supports Solana and works well as a discussion forum with governance integration, but has **no native quadratic voting**. It's best used as a community hub that links out to Realms for actual votes.

**JokeRace** allows custom voting power via CSV allowlists (manually compute square-root weights and upload them), but is **EVM-only** — no Solana support.

**Vocdoni** (Aragon-affiliated) supports quadratic voting with strong privacy features (zk-proof anonymous voting), but is **not Solana-compatible** and takes weeks to deploy.

| Platform | Quadratic voting | Solana support | Setup speed | Wallet exclusion | Cost |
|----------|:---:|:---:|:---:|:---:|:---:|
| **Realms** | Native plugin | Native | Hours | Treasury auto-excluded | Free |
| **Snapshot** | Native type | EVM only | Hours | Via strategies | Free |
| **MetaDAO** | N/A (futarchy) | Native | Days | Market-driven | Free |
| **Tally** | No | MultiGov | Weeks | No | Enterprise |
| **Commonwealth** | No | Yes | Hours | No | Free |
| **JokeRace** | No (manual CSV) | EVM only | Minutes | Allowlist | Free |
| **Vocdoni** | Yes | No | Weeks | Yes | Unclear |

---

## The Civic Pass friction is worth it

The main objection to Realms QV is that **every voter must obtain a Civic Pass** before casting a ballot. This adds a verification step — typically a liveness check taking 2-5 minutes per person. For a DAO trying to launch a vote within days, this feels like friction.

But it's essential friction. Without Sybil resistance, quadratic voting is trivially gameable. A whale holding 1,000,000 tokens gets √1,000,000 = 1,000 voting power in a single wallet. Split across 1,000 wallets of 1,000 tokens each, that becomes 1,000 × √1,000 = **31,623 voting power** — a 31x amplification. Civic Pass prevents this by ensuring each wallet maps to one real human. Skipping Sybil resistance doesn't just weaken QV — it can make the whale *more* powerful than simple token-weighted voting.

For KOTOPIA's community, the messaging is straightforward: "Verify once, vote fair. This 2-minute step ensures no single wallet can dominate the outcome."

---

## Recommended deployment plan for KOTOPIA

**Day 1: Create the DAO and configure QV.** Use Realms' no-code wizard to create a Community Token DAO with the quadratic voting plugin and Civic Pass. Set the circulating supply factor to account for the protocol-owned wallet's share. Test on Solana devnet first, then deploy to mainnet. Total active work: **2-4 hours**.

**Day 1-2: Voter onboarding.** Announce the vote to the community with clear instructions: connect Phantom wallet → obtain Civic Pass → vote. Provide a step-by-step guide. Allow 24-48 hours for the community to verify before the voting period begins.

**Day 2-3: Launch the vote.** Create the proposal on Realms with defined choices and a voting period of 3-7 days. Share the direct link across all community channels. Votes are on-chain, transparent, and immutable.

**Optional enhancement:** Run a **Polis (pol.is)** conversation in parallel for open-ended sentiment discovery. Polis uses ML to cluster opinions and surface consensus statements — useful for identifying *what* to vote on before running the formal Realms vote. It's free, embeddable, and takes minutes to set up, though it has no blockchain integration and treats all participants equally regardless of token holdings.

---

## Conclusion

The Solana governance tooling landscape has matured enough that KOTOPIA's problem is solvable today with off-the-shelf tools. **Realms' QV plugin with Civic Pass is the clear winner** — it's the only solution that combines native Solana support, production-ready quadratic voting, built-in Sybil resistance, and automatic treasury wallet exclusion. The circulating supply factor was literally designed for the scenario where a single entity holds a disproportionate share. Snapshot's superior UX and ecosystem don't help here because it simply doesn't speak Solana. MetaDAO's futarchy is the most intellectually compelling anti-whale mechanism on Solana but adds complexity inappropriate for a fast community sentiment vote. For a vote that needs to be live this week, Realms is the answer.
