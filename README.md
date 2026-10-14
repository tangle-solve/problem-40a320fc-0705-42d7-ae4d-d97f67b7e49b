<!-- solve-problem:40a320fc-0705-42d7-ae4d-d97f67b7e49b -->
# Contribution-based searcher rewards beyond copied-code Sybils

A [Tangle Solve problem](https://solve.tangle.tools/problems/40a320fc-0705-42d7-ae4d-d97f67b7e49b).

## What is the problem?

In MEV (Maximal Extractable Value) markets, searchers compete to capture value from transaction ordering (e.g., arbitrage, liquidations) by submitting bundles to builders. A protocol designer wants to reward searchers *according to their contribution* — the incremental value each searcher's strategy adds to the total extractable surplus — rather than pay-per-bundle. The known hard failure mode is **Sybil attacks**: a searcher splits one strategy into many fake identities (literally copying the same code/bundle under multiple keys) to multiply its apparent contribution share.

The current state of the art (per the census, attributable to Sarenche and Liu 2026) provides contribution-based reward guarantees (Shapley-value-style allocations with caps) only against a *modeled* adversary who Sybils by **exact code copying**, possibly in coalitions, with guarantees expiring/holding up to October 2026 formulations. The open problem: **design and prove Sybil-resilience (or characterize achievable reward rules) for contribution-based searcher rewards when identity manipulation goes beyond literal copying**, specifically:

1. **Semantic variants**: one strategy re-implemented in different code, different bundling, or slightly perturbed parameters so it is not textually identical but functionally duplicates another submission.
2. **Interacting opportunities**: strategies that partially overlap in the opportunities (arb cycles, liquidation targets) they touch, so "duplicates" are not cleanly separable and Shapley values interact nontrivially.
3. **Searcher–builder coalitions**: builders (who order transactions) colluding with searchers to inflate measured contribution.

Formal setting (most reasonable interpretation): define a value game where players are searcher identities, a characteristic function v(S) gives extractable surplus realizable by identity set S, and rewards are an allocation rule φ. The property demanded is a **Sybil-resilience / false-nameproofness condition** — e.g., a searcher cannot gain by splitting one strategy into k functionally-equivalent identities: φ across the split ≤ φ as one identity — plus standard properties (individual rationality, budget balance, incentive for truthful revelation). Ambiguity: "semantic equivalence" of strategies needs a formal definition (behavioral equivalence on the opportunity set? ε-close extraction profiles?); the solver must fix one.

## Why does it matter?

MEV extraction is dominated by a handful of sophisticated searchers; contribution-based rewards are the main proposed lever for decentralizing searcher competition fairly (e.g., in protocols like MEV-Share/MEV-blocker-style rebates and proposer-builder designs). If reward rules only resist literal code copying, any motivated searcher defeats them with a rewrite — the guarantees are vacuous in practice. Conversely, over-aggressive duplicate detection punishes genuinely independent searchers who converge on the same arbitrage, chilling participation. A solution would let reward-sharing protocols pay real economic contribution robustly, which is a precondition for credible on-chain fairness claims and for decentralized builder designs that don't collapse into collusion.

## What's already known?

- The anchor result (census): Shapley-capped contribution rewards with proven guarantees against *modeled copied-code Sybils* and simple coalitions (Sarenche & Liu 2026). The model presumably detects duplicates via literal code/bundle identity and caps the aggregate Shapley share of clones. **(verify exact model — the paper is the mandatory first read.)**
- From the false-nameproof mechanism design literature (the "related" thread): false-name-proofness without money is essentially impossible in general combinatorial settings (Yokoo's line of work); false-name-proofness with monetary transfers is possible but yields weak efficiency guarantees. Applied intuition: full Shapley Sybil-proofness and efficiency cannot both hold in general, so the achievable target is a *capped/bounded-gain* rule, exactly the shape the anchor result takes.
- From MEV literature: empirical studies document searcher concentration and repeated identical-strategy competition on the same opportunities (congestion/duplicate arb bundles). Ordering-auction theory (e.g., ordering auctions, channel auctions) addresses builder-side incentives but not searcher identity manipulation.
- **Gap**: no published reward rule handles (a) non-identical code implementing the same strategy, (b) partially overlapping opportunity footprints, (c) builder-side collusion inflating contribution measurement. That triad is the problem.

## What would count as solving it?

Any of the following, in decreasing strength:

1. **Strong (full characterization)**: A formal model of strategy space and semantic equivalence, plus a theorem characterizing all allocation rules satisfying Sybil-resilience (against semantic variants and coalitions), budget balance, and individual rationality — including an impossibility boundary showing what must be sacrificed.
2. **Construction**: An explicit allocation rule (e.g., Shapley value computed over *opportunity-footprint clusters* rather than identities, with cluster caps) with a proof that no adversary using semantic re-implementations, overlapping strategies, or builder collusion gains more than a stated additive/multiplicative bound.
3. **Lower bound / impossibility**: A proof that any budget-balanced contribution reward rule admits a profitable semantic-Sybil or builder-collusion strategy, quantifying the unavoidable welfare/revenue loss — this delimits the design space and is itself a publishable resolution.
4. **Experimental**: A simulation on realistic MEV opportunity data (or a fork of a reward-sharing protocol) demonstrating the rule's robustness against adaptive Sybil adversaries using re-implemented strategies.

## How to start

1. **Read Sarenche & Liu (2026) first, especially the model section and the copied-code Sybil theorem.** Identify precisely which assumptions break under semantic variants (this defines your delta). Since it's the single anchor paper, extract its allocation rule and adversary model verbatim.
2. **Formalize "semantic duplicate"**: the cleanest route is to define strategies by their *footprint* — the set of MEV opportunities (arb cycles, liquidations) they extract from and their extraction profile — and call two identities duplicates if footprints (nearly) coincide. This converts code-similarity (fragile) into set-similarity (game-theoretic). The main obstacle: near-coincident footprints also arise from genuinely independent searchers; you must bound the false-positive penalty.
3. **Import machinery**: read the false-name-proof mechanism design literature (Yokoo et al.) and coalitional cost-sharing Sybil-resistance results; the Shapley-capped structure suggests a weighted/clan Shapley approach where weights derive from footprint overlap.
4. **Handle builder collusion last** — it's the hardest layer; a natural start is assuming the builder's measurement channel is auditable (bids are observable) and asking whether contribution can be computed from public data only.

## Key references

- Sarenche, H., & Liu, [initials unknown] (2026). *Towards Decentralized Searcher Competition in MEV Markets*. DOI:10.4230/LIPIcs.AFT.2026.21. — Establishes Shapley-capped contribution-based searcher rewards with guarantees against copied-code Sybils and simple coalitions; the direct predecessor whose model this problem extends. (Anchor; verify author initials and exact results from the paper.)
- Yokoo, M., Sakurai, Y., & Matsubara, S. (2004). *The effect of false-name bids in combinatorial auctions*. Games and Economic Behavior. — Proves core impossibility results for false-name-proofness, supplying the lower-bound toolkit. (Verify exact citation; standard in the field.)
- Roughgarden, T. (2021). *Transaction Fee Mechanism Design*. EC'21 / JACM. — Models MEV-adjacent auction incentives; background on what allocation rules can achieve in blockchain settings. (Standard reference; verify.)

## Warnings

- The anchor paper (Sarenche & Liu 2026) is the *entire* cited literature; the census description is secondhand. Do not assume details of its theorem — read the actual paper before building on it, and treat the "October 2026" framing as census metadata, not a paper claim.
- "Semantic equivalence" of strategies is undefined in the source. Any definition you choose changes the problem; state yours explicitly and check it's robust to trivial obfuscation (reordering, dead code, parameter jitter).
- Common dead end: trying to detect similarity at the code level (hashing, static analysis). Adversaries defeat this trivially; ground equivalence in behavior on the opportunity set, not source.
- Don't prove Sybil-resistance against a fixed adversary model and claim generality — the census's whole point is that literal-copy models are too weak. Your adversary should be allowed arbitrary re-implementations.
- Watch the tension with false positives: over-merging identities punishes honest independent searchers who find the same arbitrage; a rule that's only "safe" by merging everyone is vacuous. Report false-positive costs.

---
*`BCGT-000300` · flagged for review*

## Acceptance guidance

Any of the following, in decreasing strength:

1. **Strong (full characterization)**: A formal model of strategy space and semantic equivalence, plus a theorem characterizing all allocation rules satisfying Sybil-resilience (against semantic variants and coalitions), budget balance, and individual rationality — including an impossibility boundary showing what must be sacrificed.
2. **Construction**: An explicit allocation rule (e.g., Shapley value computed over *opportunity-footprint clusters* rather than identities, with cluster caps) with a proof that no adversary using semantic re-implementations, overlapping strategies, or builder collusion gains more than a stated additive/multiplicative bound.
3. **Lower bound / impossibility**: A proof that any budget-balanced contribution reward rule admits a profitable semantic-Sybil or builder-collusion strategy, quantifying the unavoidable welfare/revenue loss — this delimits the design space and is itself a publishable resolution.
4. **Experimental**: A simulation on realistic MEV opportunity data (or a fork of a reward-sharing protocol) demonstrating the rule's robustness against adaptive Sybil adversaries using re-implemented strategies.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/40a320fc-0705-42d7-ae4d-d97f67b7e49b?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000300.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000300)
