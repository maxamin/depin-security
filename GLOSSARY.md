# Glossary

- **DePIN:** decentralised physical infrastructure network; pays tokens for operating real-world hardware.
- **Proof-of-coverage (PoC):** scheme that rewards devices for showing radio coverage, typically through beacons and witnesses.
- **Beacon:** packet a device transmits on request so that neighbours can witness it.
- **Witness:** a device's signed report that it heard a beacon, with signal measurements.
- **Verifier oracle:** off-chain service that checks proofs and outputs rewardable and invalid receipts.
- **Denylist:** list of devices or links denied rewards because of suspected spoofing or malfunction.
- **Reciprocity:** ratio of how often a device is heard by others to how often it hears others.
- **Free-space path loss (FSPL):** minimum signal loss over distance with no obstacles; signals stronger than FSPL allows are implausible.
- **Distance bounding:** challenge-response timing that bounds how far away a prover can be.
- **Verifiable multilateration:** secure positioning using several distance-bounding verifiers around a claimant.
- **Wormhole attack:** tunnelling packets over a fast link so distant nodes appear close.
- **GNSS spoofing:** transmitting fake satellite navigation signals to falsify position or time.
- **Root of trust:** minimal trusted hardware/firmware that holds device identity and produces attestation evidence.
- **DICE:** Device Identifier Composition Engine; derives device identity from a unique device secret and firmware measurement.
- **Attestation:** signed evidence of a device's identity and software state, checked by a verifier.
- **Self-dealing:** an operator paying itself for usage to earn rewards exceeding the fees.
- **Recurring fee:** ongoing cost per identity used to make Sybil identities expensive to maintain.
