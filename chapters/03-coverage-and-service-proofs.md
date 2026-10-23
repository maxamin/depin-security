# Coverage, Bandwidth and Compute Proofs

Location says where a device is. Coverage, bandwidth, storage and compute proofs say what it *does*. This chapter looks at how these proofs are built, how farmers fake them, and which design choices make faking expensive.

## Proof-of-coverage by beacons and witnesses

The best-known design is the beacon-and-witness scheme used by Helium's LoRaWAN IoT network for several years:

1. A hotspot is told (often using public randomness) to transmit a **beacon** packet.
2. Nearby hotspots that hear it submit signed **witness** reports with signal strength, signal-to-noise ratio, frequency and timestamp.
3. An off-chain **verifier oracle** correlates beacons with witnesses, rejects implausible ones, and outputs rewardable and invalid receipts.
4. A reward calculator pays beaconers and witnesses, often scaled by local hotspot density so that stacking many hotspots in one spot pays less.

The scheme assumes that radio links are hard to fake. In practice, the cheapest attacks are:

| Attack | How it works | Signal it leaves |
|---|---|---|
| Co-located cluster claiming spread-out locations | Several hotspots in one room assert positions kilometres apart and witness each other | Signal far too strong for the claimed distances; terrain ignored |
| Beacon relay / wormhole | A gateway or internet link re-broadcasts beacons elsewhere | Late arrival times; witnesses with no line of sight |
| Fake witness reports | Firmware or emulator signs reports for beacons never heard | Witness never heard by others; perfect, repeated signal values |
| Density gaming | Assert locations in empty hexes to avoid density penalties | Locations in water, forests, no customers |

Helium's 2023 developer blog posts describe how its **denylist** worked at that time: a list of hotspot addresses denied PoC rewards, regenerated weekly from two weeks of PoC data. Early metrics included terrain intersection (flagging hotspots more than three standard deviations above the network average) and witness reciprocity (the ratio of incoming to outgoing witnesses, where zero suggested spoofing or malfunction). A later post proposed denying individual improbable witness *edges* rather than whole hotspots, partly because an ordinary LoRaWAN gateway that re-broadcasts any PoC packet it hears could get honest neighbours flagged. Owners could appeal with evidence. These posts are from 2023, and the network has changed since (see the HIP 149 note in Chapter 1), but the trade-offs they describe apply to any witness-based design.

## Protocol weaknesses underneath

Coverage proofs ride on the radio protocol, so its weaknesses become proof weaknesses. Gildas Avoine and Loïc Ferreira's "Rescuing LoRaWAN 1.0" (Financial Cryptography 2018) analysed the then-deployed LoRaWAN 1.0 and presented attacks on "network availability, data integrity, and data confidentiality". These come from the protocol design itself, not from implementation bugs, and need no physical access. Their recommendations stay compatible with the specification. For DePIN reviewers the point is: check which LoRaWAN version and key-management setup devices use, and whether frame counters and join procedures resist replay.

## Timing as a relay detector

A relayed beacon arrives late. Radio travels about 300 km per millisecond, so honest witnesses within tens of kilometres should report arrival within a fraction of a millisecond of each other, after correcting for distance. A relay over the internet adds tens to hundreds of milliseconds. This only works if witness clocks are trustworthy (GNSS-disciplined clocks can themselves be spoofed, as Chapter 2 showed) and fine-grained timestamps are available, so treat it as one signal among several.

## Bandwidth, storage and compute

| Service | Proof approach | Main farming route |
|---|---|---|
| Bandwidth/relay | Challenger nodes send test traffic; clients sign receipts | Self-dealing: farmer is both client and provider; challengers colluding with providers |
| Storage | Challenge–response proofs that data is held (proofs of retrievability, replication) | Outsourcing storage to a cheap central store; generating data on demand if it is predictable |
| Compute (GPU) | Benchmarks, job results, attestation of hardware model | Spoofed device metadata; emulated workers; idle hardware claiming capacity |

**Self-dealing** deserves special attention. If an operator can pay themselves for "usage" and earn more in rewards than they pay in fees, usage rewards become proof rewards with extra steps. Check that rewards per unit of usage never exceed the fee for that usage, or that usage must come from verified independent customers.

**Spoofed metadata** is the compute equivalent of a false location. The April 2024 io.net incident (Chapter 1) involved fake GPU workers; whatever the exact mechanism, a worker declaring "I am an H100" must be checked by work only that hardware can do in the time allowed, or by attestation from the hardware vendor's root of trust (Chapter 4).

## Design principles

- Pay for **work that someone verifies independently**, ideally a paying customer.
- Make challenges **unpredictable** (public randomness), so precomputation does not help.
- **Cross-check physics**: distance, signal strength, terrain, timing.
- Prefer **edge-level penalties** to whole-device bans where possible, to limit collateral damage to honest operators.
- **Publish invalid-reason codes**, so honest operators can fix setups and appeal.

## Worked example

*Execution status: run locally with Python 3.13 (standard library only); output observed on synthetic data.*

Detect a relayed beacon from arrival times. Save as `replay_check.py`:

```python
import hashlib, statistics
obs = [(b"bcn-17", "W1", 1000.0, 3), (b"bcn-17", "W2", 1000.1, 8), (b"bcn-17", "W3", 1000.2, 12),
       (b"bcn-17", "W9", 1350.0, 2), (b"bcn-18", "W1", 5000.0, 4), (b"bcn-18", "W4", 5000.1, 9)]
C = 299.792  # km per ms
by = {}
for p, w, t, d in obs:
    by.setdefault(hashlib.sha256(p).hexdigest()[:8], []).append((w, t, d))
for h, rows in by.items():
    t0 = statistics.median(t - d / C for _, t, d in rows)
    for w, t, d in rows:
        lag = t - (t0 + d / C)
        print(h, w, f"lag={lag:7.2f} ms", "SUSPECT-RELAY" if lag > 50 else "ok")
```

Run `python3 -I replay_check.py`. Observed: all honest witnesses had a lag within ±0.13 ms; witness W9 had a lag of about 349.88 ms and was flagged `SUSPECT-RELAY`. The median makes the estimate robust to one bad witness, but if most witnesses of a beacon are in the farm, the median moves with them. Combine with the plausibility checks from Chapter 2.

## Common pitfalls

- Banning whole devices on one statistical signal and hitting honest neighbours of a relay.
- Allowing usage rewards to exceed usage fees, which invites self-dealing.
- Trusting device-declared hardware specs.
- Predictable challenges that let farms precompute answers.
- Forgetting the radio protocol version and its replay protections.

## Exercises

1. Add a farm of five witnesses that all report the relayed time. Does the median-based detector still work? Propose a fix.
2. Write a model of self-dealing for a bandwidth DePIN: at what reward-to-fee ratio does it become profitable?
3. Design a GPU proof that a consumer card cannot pass while claiming to be a data-centre card, and list its false-positive risks.

## Further reading

- G. Avoine and L. Ferreira, "Rescuing LoRaWAN 1.0", Financial Cryptography 2018.
- Helium developer blog, "Denylist evolution" (August 2023): <https://docs.helium.com/devblog/2023/08/07/denylist-evolution>
- Helium developer blog, "Denylist refine" (September 2023): <https://docs.helium.com/devblog/2023/09/14/denylist-refine>
- See **oracle-security-engineering** for oracle design and **defi-economic-attack-simulation** for reward modelling.
