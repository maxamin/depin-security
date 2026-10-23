# Proof-of-Location and GNSS/RF Spoofing

Many DePIN rewards depend on *where* a device is. Coverage networks pay more for hotspots in under-served areas. Mapping networks pay for roads driven. GNSS correction networks pay for reference stations spread over a region. Location is therefore the most valuable thing to lie about, and the hardest to verify from a distance.

## Why GNSS is not proof

A GNSS receiver computes position from unauthenticated civilian signals. Anyone with a software-defined radio can transmit fake ones. At DEF CON 23 (2015), Huang Lin and Yang Qing of Qihoo 360's Unicorn Team showed a "low-cost GPS simulator" built on SDR boards such as bladeRF, USRP and HackRF. At DEF CON 25 (2017), Dave "Karit" of ZX Security used GPS spoofing to shift the time on NTP servers that trusted a GPS clock. Karit's slides list the services that "rely on it", from taxis to ship tracking, and ask the audience whether they still trust GPS location.

DePIN farmers rarely need an SDR. They have easier options:

| Technique | Effort | What it defeats |
|---|---|---|
| Assert a false location in the app or on chain | Trivial | Networks that trust the asserted location |
| Mock-location apps or modified firmware reporting fake fixes | Low | Phone- or device-reported GNSS |
| Replay of recorded GNSS or sensor traces | Medium | Mobility/mapping proofs without liveness checks |
| SDR GNSS spoofing in a shielded box | Medium | Devices with genuine receivers |
| Relocating real devices to high-reward areas while claiming the old location | Low | Everything except RF-based cross-checks |

The key lesson: a location claim must be checked against something the claimant does not control.

## Physical checks that work

**Distance bounding.** Radio signals cannot travel faster than light. A verifier that measures round-trip time can bound how far away the claimant is. The claimant can delay its reply to appear *further* away, but cannot appear *closer*. Sahinoglu, Gezici and Guvenc's *Ultra-wideband Positioning Systems* (2008) describe challenge–response ranging that resists "distance enlargement and reduction attacks", and **verifiable multilateration**: with three or more verifiers around the claimant, any attempt to enlarge one distance would force another distance to shrink, which distance bounding prevents. This gives strong location proofs, but needs precise timing hardware (UWB, not LoRa or Wi-Fi beacons).

**RF plausibility.** Received signal strength falls with distance. Free-space path loss is the best case. A witness reporting a signal much *stronger* than free-space physics allows, at the claimed distance, is either lying about location or receiving a relayed signal. Terrain models add more: a hill between two hotspots should block line-of-sight links.

**Witness geometry.** Real devices are heard by neighbours in a pattern that matches geography. Fake clusters often show impossible patterns: devices "kilometres apart" that hear each other perfectly, or one device that hears others but is never heard (a receive-only listener).

**Kinematics.** For moving devices, two fixes imply a speed. Speeds above what a car or plane can do are impossible. Sudden jumps, perfectly straight paths and identical speed profiles across devices point to replay or simulation.

**Multiple sources.** Karit's NTP advice applies to location too: use at least three upstream sources of different types, so a single spoofed one can be detected and dropped. ZX Security's open-source GPSnitch tool watches for time offsets, signal-to-noise values and their spread, and stationary-location changes as spoofing signs.

## Secure localisation research

Wireless sensor network research faced the same problem before DePIN existed. Boukerche (ed.), *Algorithms and Protocols for Wireless Sensor Networks* (2008), has a chapter on secure localisation systems and lists the attacks a DePIN verifier faces: **Sybil nodes** that "appear at more than one place simultaneously", and **wormholes** that tunnel messages over a low-latency link so that distant nodes seem close. A wormhole is exactly how a beacon recorded in one city can be "witnessed" in another.

## Worked example

*Execution status: run locally with Python 3.13 (standard library only); the output below was observed on synthetic data.*

Save as `witness_check.py`. It checks synthetic witness reports for range and free-space signal plausibility, and checks one pair of fixes for impossible travel.

```python
import math
def km(a, b):
    la1, lo1, la2, lo2 = map(math.radians, (*a, *b))
    h = math.sin((la2-la1)/2)**2 + math.cos(la1)*math.cos(la2)*math.sin((lo2-lo1)/2)**2
    return 2*6371*math.asin(math.sqrt(h))
def fspl_max(tx_dbm, d_km, f_mhz=868.0):
    return tx_dbm - (20*math.log10(max(d_km, 0.001)) + 20*math.log10(f_mhz) + 32.44)
pos = {"A": (52.370, 4.890), "B": (52.380, 4.900), "C": (52.090, 5.120), "D": (51.920, 4.480)}
reports = [("A", "B", -95), ("A", "C", -118), ("B", "C", -60), ("A", "D", -61), ("C", "D", -62)]
for b, w, rssi in reports:
    d, best = km(pos[b], pos[w]), fspl_max(14, km(pos[b], pos[w]))
    flags = (["beyond-range"] if d > 50 else []) + ([f"too-strong(+{rssi-best:.0f}dB)"] if rssi > best + 3 else [])
    print(f"{b}->{w} d={d:5.1f}km rssi={rssi} max={best:6.1f}", " ".join(flags) or "OK")
v = km((52.370, 4.890), (51.920, 4.480)) / (60/3600)
print(f"implied speed {v:.0f} km/h", "IMPOSSIBLE" if v > 300 else "ok")
```

Run `python3 -I witness_check.py`. Observed: A→B and A→C passed. B→C, A→D and C→D were flagged as about 48–51 dB stronger than free space allows at 36–57 km, and A→D was also beyond the 50 km range limit. The two fixes 60 seconds apart implied about 3 440 km/h and were flagged impossible. In a real verifier you would add terrain, antenna gain and measurement error, and tune the margins to keep false positives low.

## Common pitfalls

- Trusting asserted or device-reported location without any independent check.
- Checking only maximum range, not signal plausibility; relays and wormholes pass range checks.
- Using margins so tight that honest devices with good antennas get flagged.
- Assuming SDR spoofing is the main threat when simple relocation is cheaper.
- Running spoofing experiments over the air: GNSS transmission is illegal without a licence in most countries. Use cabled setups or shielded enclosures, and only with authorisation.

## Exercises

1. Extend `witness_check.py` with an antenna-gain parameter and measure how many synthetic honest links are flagged as you vary the margin.
2. Add a reciprocity check: flag devices that witness others but are never witnessed. Generate synthetic data with one receive-only listener.
3. Read GPSnitch's source and list which of its checks could run on a DePIN device to self-report suspected spoofing.

## Further reading

- Huang Lin and Yang Qing, "GPS Spoofing: Low-cost GPS simulator", DEF CON 23 (2015).
- Dave "Karit", "Using GPS Spoofing to Control Time", DEF CON 25 (2017); GPSnitch: <https://github.com/zxsecurity/gpsnitch>
- Sahinoglu, Gezici and Guvenc, *Ultra-wideband Positioning Systems* (Cambridge University Press, 2008), on distance bounding and verifiable multilateration.
- Boukerche (ed.), *Algorithms and Protocols for Wireless Sensor Networks* (Wiley, 2008), chapter on secure localisation.
- See **space-segment-cybersecurity** for satellite-signal threats.
