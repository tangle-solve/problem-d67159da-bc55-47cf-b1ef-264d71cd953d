<!-- solve-problem:d67159da-bc55-47cf-b1ef-264d71cd953d -->
# Separating liquidity depegs from insolvency

A [Tangle Solve problem](https://solve.tangle.tools/problems/d67159da-bc55-47cf-b1ef-264d71cd953d).

## What is the problem?

**Problem:** Given real-time observables from a stablecoin market, causally decompose a depeg (deviation of the stablecoin's secondary-market price from its peg, e.g., $1) into distinct causes:

1. **Insolvency / reserve losses:** backing assets have fallen below liabilities (fundamental value < peg);
2. **Redemption frictions:** the issuer honors redemptions only with delay, gating, or haircuts, so arbitrage cannot enforce the peg even if solvent;
3. **Flow imbalance:** transient net selling pressure exceeding arbitrage capital (price impact without solvency concerns);
4. **Exchange segmentation:** the stablecoin trades at different prices across venues with limited cross-venue arbitrage.

**Formal setting:** Observed time series (tick or minute frequency) of {exchange prices p_e(t) across venues e, volumes, order-book depth/bid-ask spreads, redemption queue data or on-chain burn/mint flows, issuer reserve attestations (low frequency), prices of reserve assets}. The goal is an online (real-time) classifier/attribution: at each time t after a depeg begins, output posterior probabilities over the four causes, with guarantees on detection delay and false attribution rates. This is an *identification + online change-point detection* problem: the map from cause to observable signature must be shown to be invertible (or not), and a detection rule built on it.

**Ambiguity to note:** "insolvency" is often unobservable in real time (attestations are monthly/weekly), so the problem inherently involves inference under partial reserve observability — likely the crux.

## Why does it matters?

Misattribution is costly in both directions: treating a pure liquidity depeg (e.g., USDC's March 2023 SVR-driven episode) as insolvency triggers unnecessary redemptions that *cause* a run; treating creeping insolvency as a liquidity event delays intervention. A validated real-time attribution method would inform issuer circuit-breakers, redemption-gating policy, exchange listing decisions, and regulator/central-bank backstop decisions. It would also settle empirical debates over whether 2023-style depegs were self-fulfilling runs or fundamental repricing.

## What's already known?

- **Lyons and Viswanath-Natraj (2023)** document that arbitrage activity (primary-market mint/burn against the issuer) stabilizes secondary-market prices in normal times — the baseline mechanism the four causes above disrupt. This is the key empirical anchor: deviations from their arbitrage-stabilization baseline are the signal to be explained.
- Empirical case studies exist for major episodes (USDT June 2021, UST May 2022 collapse, USDC March 2023) showing depegs with different subsequent dynamics (recovery vs. death spiral), but these are *ex post* narratives, not real-time causal identification.
- **Methodological literatures to import:** structural market-microstructure identification (sign/restriction-based identification of flow vs. inventory shocks; Hasbrouck-style information shares for segmentation) and online change-point detection (CUSUM, Bayesian online changepoint detection, Shiryaev). No published work is known to us that unifies these for stablecoin depeg attribution — that is the gap.
- Theoretical run models (Diamond–Dybvig-style stablecoin papers, e.g., Klages-Mundt and Buterin (verify); also models by Liu, Makarov, and Schoar on Tether (verify)) characterize equilibria but do not deliver real-time estimators.

## What would count as solving it?

Being an experimental problem, any of:

1. **Identification result:** a demonstration (structural model + testable restrictions) that the four causes induce distinguishable observable signatures (e.g., cross-venue price dispersion ↑ only under segmentation; redemption-queue growth + peg restoration upon issuer announcement only under liquidity causes), including proof/argument of when they are *not* distinguishable.
2. **Estimator + validation:** an online classifier fed by real-time data, validated on historical episodes (UST 2022, USDC 2023, USDT 2021, DAI events) with correct attribution, quantified detection delay, and false-alarm rates; ideally with backtests showing earlier/more accurate classification than price-threshold baselines.
3. **Counterfactual robustness:** show the method is not fooled by adversarial episodes (insolvency disguised as liquidity, or vice versa), e.g., via simulation from a calibrated run model.

A referee would want out-of-sample validation and honest uncertainty quantification, not just fit to famous episodes.

## How to start

1. **Read Lyons and Viswanath-Natraj (2023) first** — their arbitrage-band framework gives you the null model of peg stability; depeg causes are then modeled as deviations from it. Then skim a survey of online change-point detection (e.g., Adams & MacKay 2007, Bayesian online changepoint detection) and Hasbrouck's information-share methodology for the segmentation component.
2. **Build an episode dataset:** reconstruct tick/minute data for 5–10 known depeg episodes with hand-labeled ground truth from post-hoc disclosures (reserve attestation dates, issuer statements, redemption freezes). Labels are the scarce resource — this is the main practical obstacle.
3. **Derive signatures from a structural model:** write down a simple model with an issuer, arbitrageurs with capital constraints, and segmented venues; derive what each cause predicts for (cross-venue dispersion, redemption flows, bid-ask spreads, reserve-asset prices). Insolvency is the hard case — you only see reserve marks with lag, so you must infer it from correlated moves in reserve assets.
4. **Main obstacle:** causes co-occur (March 2023 USDC was arguably simultaneous reserve-mark loss and run dynamics). Design for joint attribution, not exclusive classification.

## Key references

- Lyons, R. K., & Viswanath-Natraj, G. (2023). "What keeps stablecoins stable?" *Journal of International Money and Finance*, 132, 102777. https://doi.org/10.1016/j.jimonfin.2022.102777 — Empirically shows arbitrage via issuer mint/burn keeps secondary-market prices near peg across USDT/USDC/PAXG, and characterizes depegs as arbitrage-capital disruptions.
- Adams, R. P., & MacKay, D. J. C. (2007). "Bayesian online changepoint detection." arXiv:0710.3742 — The standard online changepoint framework to adapt for real-time depeg detection. (verify relevance to multivariate signatures)
- Hasbrouck, J. (1995). "One security, many markets: Determining the contributions to price discovery." *Journal of Finance* 50(4) — Information-share decomposition for measuring exchange segmentation. (verify)
- Klages-Mundt, A., & Buterin, D. (2022). "When money kills: stablecoins and bank runs." (verify existence/venue) — Models overcollateralization and run dynamics in stablecoins; useful for simulating insolvency-vs-liquidity episodes.

## Warnings

- **Ground truth is the bottleneck:** most episodes lack clean labels; insolvency is only confirmed weeks later by attestations. Be explicit about label uncertainty.
- **Do not assume depegs are single-cause** — the March 2023 USDC episode mixed reserve repricing and run dynamics; exclusive classification will fail.
- **Survivorship bias:** datasets of stablecoins are dominated by coins that survived; dead coins (UST) have different data availability (on-chain only).
- **Attestations are point-in-time and unaudited in some cases**; treating them as truth risks circularity when testing the insolvency hypothesis.
- The census record is vague on deliverable format; both structural-identification papers and pure ML detectors would count, but referees in this field will want an economic model behind the classifier, not a black box.

---
*`BCGT-000337` · flagged for review*

## Acceptance guidance

Being an experimental problem, any of:

1. **Identification result:** a demonstration (structural model + testable restrictions) that the four causes induce distinguishable observable signatures (e.g., cross-venue price dispersion ↑ only under segmentation; redemption-queue growth + peg restoration upon issuer announcement only under liquidity causes), including proof/argument of when they are *not* distinguishable.
2. **Estimator + validation:** an online classifier fed by real-time data, validated on historical episodes (UST 2022, USDC 2023, USDT 2021, DAI events) with correct attribution, quantified detection delay, and false-alarm rates; ideally with backtests showing earlier/more accurate classification than price-threshold baselines.
3. **Counterfactual robustness:** show the method is not fooled by adversarial episodes (insolvency disguised as liquidity, or vice versa), e.g., via simulation from a calibrated run model.

A referee would want out-of-sample validation and honest uncertainty quantification, not just fit to famous episodes.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/d67159da-bc55-47cf-b1ef-264d71cd953d?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000337.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000337)
