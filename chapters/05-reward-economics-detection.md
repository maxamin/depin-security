# Reward-Economics Attacks and Detection

Every DePIN reward formula is an incentive that attackers optimise against. Many of the worst DePIN failures were not exploits of code but of economics: formulas that paid more for faking than for building. This chapter covers economic attack patterns, how to model them before launch, and how to detect them after launch without crushing honest operators.

## Economic attack patterns

| Pattern | Mechanism | Design countermeasure |
|---|---|---|
| Capacity farming | Rewards for claimed coverage, uptime or storage with no demand | Weight rewards towards paid usage; cap proof rewards per area |
| Self-dealing | Operator pays itself for usage; rewards exceed fees | Rewards per unit of usage below its fee; independent-customer checks |
| Density arbitrage | Assert devices into empty reward zones | Penalties for implausible placement; customer-demand weighting |
| Witness rings | Groups that only witness each other | Reciprocity and graph-structure checks; random challenger assignment |
| Snapshot gaming | Spin up devices just before a reward epoch or airdrop snapshot | Minimum tenure; rewards vest over time |
| Early-emission capture | Farm the high early emissions, sell, leave | Vesting; staking requirements that are slashed on fraud |
| Oracle capture | Influence the operators who run verification | Multiple independent verifiers; published methodology; audit trails |

Two of these interact badly. If emissions are high and verification is weak, farmed tokens fund more farms. Margolin and Levine's FC 2008 analysis of Sybil resistance proposes a **recurring fee** as a deterrent: identities that must pay continuously are expensive to keep in bulk. In DePIN terms this is staking or a recurring registration cost that scales with the number of devices, combined with slashing when fraud is found.

## Modelling before launch

Before launch, simulate the reward formula with adversarial agents:

1. Define honest operator types (home user, small business, fleet) with costs and realistic placement.
2. Define attacker strategies: co-located clusters, emulated devices with key extraction, self-dealing, snapshot timing.
3. For each strategy, compute monthly profit = rewards − (hardware + hosting + staking cost + expected slashing).
4. Find the parameter region where every attacker strategy loses money while honest types profit. If no region exists, the formula needs a stronger verification signal, not tuning.

This is the same discipline as DeFi economic simulation; see **defi-economic-attack-simulation** for agent-based tools.

## Detection after launch

Detection combines **rules** (physics and protocol checks from Chapters 2–3) with **statistics** over the population.

| Signal family | Examples | Typical false positives |
|---|---|---|
| Physics | Signal too strong for distance; impossible speed; terrain blocked | Good antennas, multipath, GNSS errors |
| Graph | Low reciprocity; dense isolated witness clusters; star-shaped funding of operator wallets | Rural devices with few neighbours |
| Timing | Many devices onboarded within minutes; synchronised uptime; late beacon arrivals | Fleet deployments, shared power outages |
| Ownership | Shared funding wallets; shared IP ranges; reward wallets consolidating to one exchange deposit | Hosting providers, families, small businesses |
| Behaviour | Identical sensor readings; perfect traces; no customer traffic | Devices in quiet areas |

Wireless sensor network research lists the same adversaries. Boukerche's *Algorithms and Protocols for Wireless Sensor Networks* (2008) names Sybil nodes, wormholes and sinkholes, and pairs network-layer attacks with defences such as malicious-node detection and isolation, key management and secure routing. DePIN verifiers are a specialised case of malicious-node detection.

## Fairness and appeals

Statistical detection will flag honest operators. Helium's 2023 denylist posts are a public example of the trade-off: a weekly list derived from statistical metrics, an appeals route where owners submitted evidence, and FAQ advice that the most common cause of inclusion was a hotspot placed away from its asserted location. A later proposal moved from denying whole hotspots to denying individual improbable witness links, to reduce collateral damage from beacon re-broadcasting.

Good practice:

- **Publish reason codes** for each denial, so operators can fix setups.
- **Separate "invalid this epoch" from "banned"**: most signals should reduce rewards for specific events, not remove devices.
- **Offer appeals with evidence** (photos, installation data, RF surveys) and track appeal outcomes as a measure of false positives.
- **Keep an audit trail** of detection-rule changes and their effect on rewards.
- **Measure** precision and recall on labelled cases (confirmed farms, verified honest installs) before rolling out a rule.

## Incident handling

When a farm is found, decide in advance:

1. Do you claw back past rewards, stop future rewards, or only fix the rule? Clawbacks need on-chain mechanisms (vesting, slashing) that exist *before* the incident.
2. How do you communicate without giving farmers a tuning guide? Publish methodology and outcomes, not exact thresholds in real time.
3. How do you protect honest operators caught in the sweep? Expedited appeals.

The io.net case in April 2024 shows a further point: when the fix (re-authentication) overlapped with a reward snapshot, honest suppliers who failed to re-authenticate in time were reportedly affected too. Plan remediation timing around reward events.

## Worked example

*Execution status: Not executed in this book's CI; verify locally. Uses only synthetic data.*

Build a toy reward simulator:

1. In Python, create 500 honest devices placed randomly on a 50 × 50 km grid and one farm of 40 devices in a single 100 m square that asserts spread-out locations.
2. Implement a reward of 1 unit per valid witness, scaled down by local asserted density.
3. Run one epoch without checks and record the farm's share of rewards.
4. Add the free-space plausibility check from Chapter 2 (`witness_check.py`) and rerun. Record the farm's share and the number of honest witness links rejected.
5. Add a staking cost per device and find the stake at which the farm's net profit is negative while a typical honest device is still positive.

## Common pitfalls

- Tuning thresholds in public, in real time, which turns detection into a feedback loop for farmers.
- Banning devices on one weak signal.
- No clawback or vesting mechanism when fraud is found.
- Measuring success by "devices denied" instead of precision and recall.
- Running remediation that overlaps with reward snapshots.

## Exercises

1. Implement the simulator and plot farm share against stake size.
2. Add a "self-dealing" agent to the simulator and find the reward-to-fee ratio at which it profits.
3. Draft an appeals policy with evidence requirements and a target response time.

## Further reading

- N. B. Margolin and B. N. Levine, "Quantifying Resistance to the Sybil Attack", FC 2008 (recurring fees).
- A. Boukerche (ed.), *Algorithms and Protocols for Wireless Sensor Networks* (Wiley, 2008), security chapters.
- Cointelegraph, "io.net responds to GPU metadata attack" (April 2024): <https://cointelegraph.com/news/io-net-responds-to-gpu-metadata-attack>
- See **airdrop-sybil-detection** for graph and clustering detection methods.
