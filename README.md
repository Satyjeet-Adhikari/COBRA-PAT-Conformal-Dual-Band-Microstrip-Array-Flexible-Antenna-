<img width="1920" height="1020" alt="Screenshot 2026-09-29 202929" src="https://github.com/user-attachments/assets/1aa7936d-69d4-43f6-934c-dc3211e500b5" />

# Helmet-Mounted Conformal Dual-Band Microstrip Array (UHF + L-band)

A low-profile, flexible, dual-band microstrip antenna array for NSG ballistic helmets in urban CQB communications. Developed for the Smart India Hackathon problem statement *"Helmet mounted conformal antenna for tactical communications in urban CQB environments."*

> **Status: active development.** Design is frozen at topology level; HFSS tuning and validation are in progress. See [Current Status](#current-status) for the honest state of the simulations.

---

## 1. Problem

Vest-mounted radios use rigid whip antennas that snag on door frames and obstacles, sit low (worse penetration into RCC / steel-glass buildings), and radiate omnidirectionally. The antenna must move to the highest point of the operator, the helmet, without adding bulk or compromising ballistic protection.

| Requirement | Interpretation in this project |
|---|---|
| (a) Ultra-thin, flexible, lightweight patch elements | LCP / thin flexible laminate, conformal to helmet curvature |
| (b) UHF + L-band | UHF ~434 MHz (tactical radio), L-band ~1.5 GHz (helmet camera video link) |
| (c) Shielding toward the head | Full ground plane under the array; pattern directed up and outward; SAR to be evaluated |
| (d) Ruggedized coax interface | Micro-coax / shielded cable routed along the helmet to the radio |

---
![Uploading Screenshot 2026-09-26 214514.png…]()<img width="1920" height="1020" alt="Screenshot 2026-09-29 202929" src="https://github.com/user-attachments/assets/d8d29c34-c441-4a8c-bc99-496c3449deaa" />

## 2. Design Evolution

The final design is the result of several researched-and-discarded approaches, not a first guess.

| # | Approach | Outcome | Why it was dropped |
|---|---|---|---|
| 1 | Conductive traces engraved directly on the Kevlar helmet shell | Discarded | Couples the antenna to the ballistic structure; any modification risks ballistic integrity (requirement a); lossy/variable dielectric; not serviceable |
| 2 | Textile antenna (conductive fabric on felt/denim) | Discarded | Permittivity and thickness vary with compression, moisture and wear, causing detuning; poor edge definition at UHF/L-band feature sizes |
| 3 | FPC (flexible printed circuit) antenna | Discarded as the final form | Good fabrication path, but a plain FPC patch could not cover both bands compactly at UHF |
| 4 | Metamaterial-loaded hybrid PIFA array (SRR + U-slot loading, EVA foam core) | Discarded | Complex, hard-to-fabricate loading structures; difficult to reproduce and tune; replaced by a better-documented reference topology |
| 5 | **Conformal dual-band microstrip array (final)** | **Selected** | Integral ground plane shields the head, single-layer flexible fabrication, dual-band from one radiating structure, and a peer-reviewed reference topology to build on |

*(Edit the "why dropped" column to match your own calculations before publishing.)*

---

## 3. Final Architecture

Topology follows the mirror-symmetric two-element dual-band array of Nikolayev et al. (IEEE TAP), with only the dimensions rescaled for the helmet bands.

| Section | Role |
|---|---|
| **Z1** | Low-impedance rectangular microstrip linking Z2 and Z3 |
| **Z2** | High-impedance meandered (square-wave) line shorted to ground by a via; sets the **UHF (~434 MHz)** quarter-wave-type resonance |
| **Z3** | Patch section, quarter-wave; sets the **L-band (~1.5 GHz)** resonance, tuned by `lZ3` and `lfeed` |
| **Switch tab / w0** | Central switch line between the two mirrored elements: array mode vs. single-element mode for pattern reconfiguration |
| **Ground plane** | Full ground under the array: head shielding and decoupling from helmet contents |

Key design equations used (Balanis, Ch. 14; Pozar):

- Z1/Z2 resonance condition: `-Z1 + Z2 · tan(βZ2·lZ2) · tan(βZ1·lZ1) = 0`, with `βZn = 2πf·√(ε_eff)/c`
- Z3 initial length from the quarter-wave patch model: `lZ3 ≈ λ/4` using the estimated effective permittivity of the layered stack

---

## 4. Substrate and Layer-Stack Selection

Substrate selection was debugged progressively: band first, then substrate, then geometry, then layering.

| Candidate stack | εr | Thickness | Notes |
|---|---|---|---|
| **LCP (Rogers ULTRALAM 3850HT) + EVA foam** | 2.9 (tan δ 0.002) | 101.6 µm dielectric, 9 µm Cu, ~1 mm EVA foam between patch and ground | Chosen flexible stack; polyimide is the fallback if LCP is unavailable |
| FR-4 | 4.4 | 0.8 mm (mentor) / 2 mm (sweep) | Mentor-suggested baseline; rigid at this thickness |
| Rogers (low-εr) | 2.2 | 0.8 mm | Mentor-suggested baseline |
| C-MET | 10.2 | 2 mm | Miniaturisation study |
| Kevlar composite | 3.6 | 2 mm | Helmet-material study |

Bands used for design: UHF ~434 MHz (380-470 MHz band of interest), L-band ~1.5 GHz (1.35-1.85 GHz band of interest).

---

## 5. Problems Found and Fixed While Debugging

| Problem | Fix |
|---|---|
| Effective permittivity, meander length and Z3 length were inconsistent with the layered stack | Recomputed `ε_eff` per section and re-derived meander electrical length and `lZ3` |
| Feed modelled as a bare coax probe in the middle | Rejected; feed structure now follows the reference: stub at `lfeed` from the Z3 end, central feed lines and switch |
| Z1/Z3 gap not matching the reference | Rebuilt as a narrow slit bridged by a thin switch tab, as in the reference figure |
| Lumped port failed in HFSS (zero-length port line) | Port geometry rebuilt with a finite-length port line |
| Manual HFSS setup was error-prone | Fully automated native IronPython run-script (geometry, materials, radiation boundary, ports, setup, reports, parametrics/optimetrics) |

---

## 6. System-Level Enhancers

Beyond the antenna element, the following system-level research was carried out to improve the operational value of a helmet antenna array.

| Enhancer | Role |
|---|---|
| **MANET (Mobile Ad-hoc Network)** | Infrastructure-free mesh between team members' helmets; relays traffic where a direct line to base is blocked indoors |
| **Phase-difference detection** | Using the phase difference between the two array elements to estimate direction of arrival of a signal (phase-interferometry style) |

<!-- TODO: add your specific findings, protocols/algorithms studied, and expected benefit here -->

---

## 7. Current Status

| Item | State |
|---|---|
| Topology | Frozen to the reference paper's element/array layout |
| HFSS model | Built with the automated script (HFSS 2025 R2.4, student) |
| S11 (latest sweep, 0.2-2.0 GHz) | Resonant dips near ~1.04 GHz (about -8 dB) and ~1.47 GHz (about -3.5 dB): **not yet matched to -10 dB at 434 MHz / 1.5 GHz** |
| Gain plot | A run returned a maximum of about -78 dB; treated as a setup/matching issue under investigation, **not a valid gain result** |
| Next | Re-tune `lZ1/lZ2/lZ3/lfeed/w0` with optimetrics for the LCP + foam stack, re-check port and radiation boundary, then SAR and helmet-curvature bending analysis |

---

## 8. Literature

| Reference | Used for |
|---|---|
| D. Nikolayev, A. K. Skrivervik, J. S. Ho, M. Zhadobov, R. Sauleau, "Reconfigurable Dual-Band Capsule-Conformal Antenna Array for In-Body Bioelectronics," *IEEE Trans. Antennas Propag.* | Primary reference: Z1/Z2/Z3 dual-band topology, mirrored array with switch, transmission-line design equations, ground-plane shielding concept |
| C. A. Balanis, *Antenna Theory: Analysis and Design* (3rd ed.) | Microstrip patch design equations, effective permittivity, quarter-wave and transmission-line models |
| D. M. Pozar, *Microwave Engineering* | Transmission-line and impedance-step theory referenced by the primary paper |

---

## 9. Repository Layout (suggested)

```
/docs          research notes, calculations, reference figures
/hfss          IronPython automation scripts and .aedt project
/results       S11, gain and pattern exports
/README.md
```

## 10. Team and Context

Smart India Hackathon, NSG problem statement. Contributors: *(add names)*


