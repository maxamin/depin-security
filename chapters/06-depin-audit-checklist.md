# DePIN Security Review Checklist

A DePIN review covers more than smart contracts. The contracts usually just mint or distribute rewards; the security lives in device identity, off-chain verification and the reward formula. This checklist pulls the book together. Review only systems you are authorised to test, run experiments on local or test deployments with synthetic data, never transmit spoofed radio signals over the air, and report findings through the project's disclosure process.

## Scope the review

Collect these before you start:

- Architecture diagram from device to reward contract, with operators of each component.
- Reward formula and parameters, with emission schedule.
- Device onboarding flow and identity mechanism (level 0–4 from Chapter 4).
- Verifier/oracle code or, at least, written methodology and invalid-reason codes.
- Governance: who can change rules, allowlists, oracles and reward parameters.

Halborn's Nodle audit (2022) is a reminder that an on-chain `ensure_oracle` gate moves the trust to whoever holds the oracle origin. Ask for that part of the system even when it is out of the formal audit scope, and say so in the report if you cannot review it.

## Checklist

### A. Device identity and onboarding

- [ ] Device keys are non-exportable (secure element, TPM or DICE-derived), or the risk of extraction is documented.
- [ ] Attestation is checked with fresh nonces against a current firmware allowlist; old vulnerable versions are removed.
- [ ] One identity active in two places at once is detected.
- [ ] Onboarding APIs require strong authentication; tokens are scoped and rotate; registration rate limits exist.
- [ ] Manufacturer issuance numbers are reconciled against shipments.

### B. Location and physics

- [ ] Asserted locations are never trusted alone.
- [ ] Witness or challenger reports are checked for range, free-space signal plausibility and terrain.
- [ ] Mobile devices are checked for impossible speeds, jumps and replayed traces.
- [ ] Timing checks catch relayed beacons where timestamps allow it.
- [ ] Margins are calibrated on honest data, with measured false-positive rates.

### C. Service proofs

- [ ] Challenges are unpredictable (public randomness) and cannot be precomputed.
- [ ] Bandwidth and usage rewards cannot exceed the fees paid for that usage (no profitable self-dealing).
- [ ] Storage proofs bind to unique replicas where replication is rewarded.
- [ ] Compute capacity is verified by work or vendor attestation, not declared metadata.

### D. Verifier and oracle

- [ ] Verification code is reviewed, versioned and reproducible from public data where possible.
- [ ] More than one independent party can run verification, or the centralisation is disclosed.
- [ ] Invalid-reason codes are published to operators.
- [ ] Reward roots posted on chain can be recomputed by third parties.
- [ ] Oracle keys are in HSMs or threshold setups, with rotation.

### E. Reward economics

- [ ] Adversarial simulation shows farming strategies are unprofitable under current parameters.
- [ ] Proof rewards are capped or weighted towards real demand.
- [ ] Staking or recurring costs scale with device count; slashing exists for proven fraud.
- [ ] Rewards vest, so clawback is possible.
- [ ] Snapshot timing and minimum tenure prevent last-minute farming.

### F. Detection, appeals and response

- [ ] Detection rules are documented, with precision/recall measured on labelled cases.
- [ ] Penalties are graded (event-level invalidation before device bans).
- [ ] Appeals accept evidence and have a response target; outcomes are tracked.
- [ ] An incident plan covers clawback decisions, communication and timing relative to reward events.

### G. Contracts

- [ ] Reward distribution contracts are audited for standard issues (Merkle claim replay, double claims, access control). See **airdrop-sybil-detection** for claim-contract findings from audits.
- [ ] Parameter changes are time-locked and emit events.
- [ ] Emergency pause covers all reward paths, not just one. (Nodle's audit noted the emergency shutdown was used only in `allocate`, not in other critical functions.)

## Severity guidance

| Finding | Typical severity |
|---|---|
| Fake devices can be registered without hardware (API or key flaw) | Critical |
| Location can be falsified with no physical cross-check, and location drives rewards | High |
| Self-dealing profitable at current parameters | High |
| Verification run by a single undisclosed operator | Medium to High (centralisation) |
| No appeals or reason codes | Medium (fairness, operational) |
| Missing tenure or vesting | Medium |

## Using AI tools in a DePIN review

LLMs can help draft adversarial strategies, summarise verifier code or propose detection features. Validate every suggestion: a proposed attack must be shown profitable in your simulator, and a proposed detector must be measured on labelled or synthetic data before it goes into a report.

## Worked example

*Execution status: Not executed in this book's CI; desk review procedure.*

1. Pick a DePIN with public docs and reward formulas. Fill in the scope list above from public material only.
2. Go through sections A–G and mark each item as met, not met or unknown. "Unknown" is a finding in itself: it means the project has not published enough for users to judge.
3. For the top three gaps, build a small synthetic test (using the scripts from Chapters 2 and 3, or the simulator from Chapter 5) that demonstrates the impact.
4. Write the report: scope, trust assumptions (who runs the oracle), findings with evidence, residual risks and recommendations.

## Common pitfalls

- Auditing reward contracts only and calling the DePIN "audited".
- Treating "unknown" items as passes.
- Reporting attacks without a cost and profit estimate.
- Recommending detection rules without false-positive measurement.
- Testing against the live network instead of synthetic or test deployments.

## Exercises

1. Apply the checklist to two DePINs and compare which sections are most often "unknown".
2. Turn section E into a spreadsheet model that a project team could fill in with its parameters.
3. Write a responsible-disclosure email template for a DePIN finding that includes cost-to-exploit and suggested mitigations.

## Further reading

- Halborn, Nodle Substrate pallets audit report (2022).
- G. Avoine and L. Ferreira, "Rescuing LoRaWAN 1.0", FC 2018.
- S. Cheruvu et al., *Demystifying Internet of Things Security* (Apress, 2020).
- See **oracle-security-engineering**, **defi-economic-attack-simulation** and **bug-bounty-security-economics**.
