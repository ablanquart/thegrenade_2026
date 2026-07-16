# Service Manual — MRF101 40m AM Linear Amplifier

Board: `mrf101_40m_pa_usbpd` · 220 × 130 mm, 2-layer · Companion to
[USER_GUIDE.md](USER_GUIDE.md)

This manual describes what every functional block does, what the individual
components are for, where to put an oscilloscope probe, and what you should see
there. Reference designators follow the schematic; where an exact value matters,
confirm against the schematic rather than this text.

---

## 1. System overview

```
                    ┌──────────────── POWER ────────────────┐
J4 USB-C ─ D6 ─ F1 ─┤ U3 CH224K → 20V VBUS_F                │
                    │ U4 LM5122 boost → 48V VDD_BOOST ─ D4 ─┼─► VDD (48V)
J3 bench 48V ───────┤ ────────────────────────────── D1 ────┘
                    │ U2 LM2596HV buck  → 12V VDRV  (gated by PD_GOOD / J5)
                    └────────────────────────────────────────┘

J1 RF in ─ pad ─ driver ─ T1 1:4 ─ Q1/Q2 MRF101 push-pull ─ T2 4:1 ─┬─ LPF ─ J2 RF out
                                    (48V drain, IDQ 100 mA,          │
                                     temp-tracked bias)              └─ harmonic dump
                                                                        (C33/L8/C34 → R34‖R35)
```

Two independent stories share the board: the RF amplifier, and the power subsystem
that lets it run from a laptop brick. The seam between them is the VDD rail and the
PD_GOOD enable signal.

---

## 2. Power subsystem

### 2.1 USB-C input and PD negotiation

| Ref | Part | Function |
|---|---|---|
| **J4** | USB-C 16-pin receptacle | Power input. VBUS pads paralleled per side; outer pads and shield are grounded. |
| **D6** | SMB TVS diode | Clamps VBUS transients (hot-plug inductive spikes, misbehaving supplies) before anything downstream sees them. |
| **F1** | Fuse, 2512 | Series protection for the 20 V input path. If it opens, find out why before replacing. |
| **U3** | CH224K (SSOP-10) | USB-PD sink controller. Handles CC-line negotiation, requests 20 V, and asserts PD_GOOD (open-drain, active low) when the contract is granted. |
| **R18/R19/R20** | CFG straps | Program U3's voltage request to 20 V (CFG1/CFG2/CFG3 per the CH224K datasheet table). |
| **R31, C31** | 3.3 V rail RC | U3's internal LDO output (U3_VDD) decoupling and feed. |
| **R32** | VBUS sense | Feeds VBUS_F to U3's voltage-sense input so it can confirm the negotiated rail arrived. |
| **R21** | 10 k pull-up | Pulls PD_GOOD to U3_VDD; U3's open-drain output pulls it low on success. |
| **J5** | 2-pin jumper | Shorts PD_GOOD to ground manually — the bench-only force-enable. |
| **C29, C30** | 22 µF ceramics | VBUS_F input bulk for the boost converter. |

USB data lines (D+/D−) route to U3 for legacy BC1.2 detection only; they carry no
USB traffic. CC1/CC2 carry the actual PD negotiation.

### 2.2 Boost converter, 20 V → 48 V

| Ref | Part | Function |
|---|---|---|
| **U4** | LM5122 (HTSSOP-20) | Synchronous boost controller. |
| **R29** | Shunt, 2512 | Input-side current sense for U4's peak-current-mode control (CSP/CSN). |
| **L6** | Power inductor | The boost inductor — the energy-storage element. |
| **Q6** | NMOS, TO-252 | Low-side switch. Its drain tab **is** the SW2 switching node. |
| **Q7** | NMOS, TO-252 | High-side synchronous rectifier; source on SW2, drain to VBOOST_RAW. |
| **C22** | 100 nF | Bootstrap (BST) capacitor for the high-side gate driver, flying on SW2. |
| **C24, C25** | 10 µF ceramics | VBOOST_RAW output ceramics — first, low-ESR line of defense against switching ripple. |
| **C26, C27** | Electrolytics (radial) | Bulk output capacitance; holds the rail up under AM envelope current swings. |
| **L7** | Filter inductor | Second-stage LC with C28 — scrubs residual switching ripple off the rail before it reaches the PA. |
| **C28** | Output ceramic | Companion to L7. VDD_BOOST is measured here. |
| **R22, R23** | UVLO divider | Sets the input undervoltage lockout so the boost won't try to run from a half-negotiated rail. |
| **R24** | RT resistor | Sets switching frequency. |
| **C19** | Soft-start cap | Ramps output at startup; prevents inrush tripping the PD source. |
| **R25** | Slope comp | Slope-compensation programming. |
| **R26, C20, C21** | Compensation | Error-amplifier network on COMP (Type-II: R26+C20 series, C21 parallel). |
| **R27, R28** | Feedback divider | Sets the 48 V output: VBOOST_RAW → FB → GND. |
| **R33** | MODE strap | Selects forced-PWM operation. |
| **C32** | RES cap | Hiccup-mode restart timing. |
| **C23** | VCC bypass | U4's internal gate-drive LDO (≈7.5 V) decoupling. |

### 2.3 Rail ORing and the driver supply

| Ref | Part | Function |
|---|---|---|
| **D4** | SMC Schottky | ORs VDD_BOOST onto VDD. |
| **D1** | SMC Schottky | ORs the bench input (VDD_IN, via J3) onto VDD. Higher rail wins; neither back-feeds the other. |
| **U2** | LM2596HV-12 | 48 V → 12 V buck for VDRV, the driver-stage supply. Its EN pin is active-low, driven by PD_GOOD — so the driver (and therefore the whole RF chain) stays off until power is confirmed, or until J5 is jumpered. |
| **D7 + R36** | Green LED + 6.8 k | "PD OK" indicator, powered from VDD_BOOST: lights only when negotiation succeeded and the boost is delivering. |
| **D8 + R37** | Amber LED + 15 k | "VDD present" indicator on the ORed rail; lights from either power source (dimmer at 20 V-only fault conditions, ~3 mA at 48 V). |

---

## 3. RF chain

| Ref | Part | Function |
|---|---|---|
| **J1** | SMA edge-launch | 50 Ω drive input. |
| Input pad | Resistive attenuator | Sets drive level and presents a stable broadband source impedance to the driver (~0–3 dB as built). |
| Driver stage | — | Raises drive to the level the output pair needs; powered from VDRV (12 V). |
| **T1** | 1:4 on BN-43-202 | Input transformer/balun: splits drive into balanced push-pull gate drive and steps impedance down to the low gate impedance of the pair. |
| **Q1, Q2** | MRF101AN / MRF101BN | The output pair, TO-220 tab-down on the heatsink. The AN/BN complement exists purely for layout symmetry (mirrored pinout). 48 V drain, class AB. |
| Bias network (**Q4** et al.) | Temperature-tracked bias | A sensing transistor thermally coupled to the output devices servos the gate bias so IDQ (100 mA/device) holds as the heatsink warms, instead of running away. Adjust with the bias trimmer. |
| **T2** | 4:1 on BN-43-3312 | Output combiner: sums the push-pull drains and steps up to 50 Ω. Its output node is **RFOUT_PA**. |
| **L1…** + shunt Cs | 7-element Chebyshev LPF | 7.5 MHz cutoff. Passes 7 MHz; rejects the 2nd (14 MHz) and up by tens of dB. |
| **J2** | SMA edge-launch | 50 Ω output. |

### 3.1 The harmonic dump (absorptive termination)

A conventional LPF *reflects* the harmonics it rejects, and that reflected energy
arrives back at the drains with arbitrary phase — worsening IMD and inviting
instability. This board instead hangs a high-pass branch on RFOUT_PA, terminated in
a resistor, forming a first-order diplexer with the LPF:

| Ref | Value | Function |
|---|---|---|
| **C33, C34** | 330 pF C0G, 250 V | Series elements of a 3rd-order Butterworth high-pass, f_c ≈ 10 MHz. |
| **L8** | 390 nH | Shunt element of the high-pass. |
| **R34 ‖ R35** | 2 × 100 Ω, 3 W (= 50 Ω, 6 W) | The dump. Harmonic energy ends here as heat. |

At 7 MHz the branch looks like a high impedance and steals < 0.2 dB from the
fundamental. At 14 MHz and above it looks like 50 Ω, so the PA sees a resistive
load at every frequency of interest. Expected dissipation in normal operation is
well under 1 W; the 6 W rating is margin against mistuning and fault conditions.

---

## 4. Bias system

The output pair idles at **100 mA per device** (class AB), set by a trimmer and held
by the temperature-tracking network: the sense transistor (Q4), mounted in thermal
contact with the output devices' heatsink environment, has its own V_BE tempco
subtracted from the gate bias line, roughly cancelling the MRF101's threshold drift.

**Setting IDQ:** no drive, dummy load fitted, heatsink attached. Note supply current;
adjust the trimmer for +200 mA total (two devices). Re-verify after ten minutes of
idle warm-up — it should move only a few mA.

---

## 5. Protection & sequencing summary

1. **Hot-plug**: D6 clamps, F1 fuses.
2. **Negotiation**: U3 requests 20 V; nothing downstream runs until granted.
3. **Boost UVLO** (R22/R23): the LM5122 won't start on a sagging input.
4. **Soft-start** (C19): the 48 V rail ramps; no inrush trip.
5. **Driver interlock**: VDRV enabled by PD_GOOD (or J5).
6. **ORing** (D1/D4): dual supplies coexist safely.
7. **Absorptive output**: the PA is resistively loaded at all frequencies.

---

## 6. Oscilloscope test points

Ground your probe close to each measurement (short spring ground, not the alligator
tail, for anything above DC). ⚠ marks nodes carrying voltages or RF levels that
deserve respect. Values assume a 100 W PD source, dummy load, IDQ set.

### 6.1 Power subsystem

| # | Probe at | Net | Expect | Notes |
|---|---|---|---|---|
| TP1 | C29/C30 positive | VBUS_F | 20 V DC, <100 mV ripple | Brief 5 V interval at plug-in before negotiation completes is normal. |
| TP2 | C31 | U3_VDD | 3.3 V DC | CH224K internal LDO. |
| TP3 | J5 pin 1 | PD_GOOD | ≈0 V when negotiated; ≈3.3 V when not | Open-drain: watch it snap low ~500 ms after plug-in. |
| TP4 | **Q6 tab** ⚠ | SW2 | Rectangular wave, 0 ↔ ≈48 V, duty ≈ 58 %, at the R24-set switching frequency | The single most informative node in the boost. Clean fast edges with modest overshoot = healthy. Ringing >20 % of amplitude suggests layout/probing artifacts — reprobe with a spring ground before worrying. |
| TP5 | Q6 gate | BOOST_LO | 0 ↔ ≈7.5 V rectangular | Low-side gate drive from U4's VCC rail. |
| TP6 | Q7 gate ⚠ | BOOST_HO | Same shape, but riding on SW2 | High-side drive is SW-referenced — use a differential probe or interpret accordingly; a single-ended probe here shows SW2 plus the gate offset. |
| TP7 | C28 | VDD_BOOST | 48 V DC, ripple < ~50 mV after the L7/C28 filter | Sawtooth ripple at f_sw before L7 (probe C24/C25) is normal and larger. |
| TP8 | D4/D1 cathode side | VDD | 48 V DC | Dips of more than a volt or two on modulation peaks mean bulk capacitance or the PD source is struggling. |
| TP9 | U2 output | VDRV | 12 V DC | Only present when TP3 is low (or J5 jumpered). 52 kHz-ish LM2596 ripple is normal. |

### 6.2 RF chain

| # | Probe at | Net | Expect | Notes |
|---|---|---|---|---|
| TP10 | J1 side of the input pad | RF in | 7 MHz sine, ~20 Vpp at 1 W drive | Verifies the exciter before blaming the amplifier. |
| TP11 | Q1/Q2 gate (either) | Gate bias | ≈2–3 V DC + drive superimposed | DC level is the bias servo output; both gates should match within tens of mV. |
| TP12 | C33's tap trace / L1 pad ⚠ | RFOUT_PA | 7 MHz, ≈32 Vpk carrier, ≈63 Vpk at PEP crest (10 W/40 W into 50 Ω) | Pre-filter node — visible harmonic content and flat-topping on peaks is *expected here*; the LPF cleans it. Use a ×10 probe minimum. |
| TP13 | J2 (through a power attenuator!) ⚠ | RF out | Clean 7 MHz sine, same amplitudes | **Never** probe J2 directly at power — use a tap or attenuator. Compare spectral purity against TP12 to see the filter earn its keep. |
| TP14 | Across R34/R35 | HD_DUMP | Small residual RF, typically < 5 V RMS | **The built-in harmonic meter.** P_dump = V_RMS² / 50. Under 0.5 W is healthy. A sudden rise here is your earliest warning of bias drift, parasitic oscillation, or filter damage — before anything smokes. |

### 6.3 Diagnostic patterns worth knowing

- **SW2 stuck low, no green LED:** boost never started — check UVLO divider values,
  VBUS_F actually at 20 V, C19 not shorted.
- **SW2 switching in bursts:** hiccup mode — output overloaded or shorted; also the
  signature of an undersized PD source current-limiting.
- **TP14 grows with heatsink temperature:** bias tracking failing (Q4 thermal contact),
  crossover distortion rising as IDQ falls.
- **TP14 shows non-harmonic frequencies:** parasitic oscillation — stop transmitting,
  check gate resistors and T1/T2 integrity.
- **TP12 asymmetric top/bottom:** push-pull imbalance — one device biased off or dead;
  compare TP11 on both gates.

---

## 7. Troubleshooting matrix

| Symptom | Check, in order |
|---|---|
| Dead on USB-C | Cable (needs e-marked 5 A), F1 continuity, D6 not shorted, TP1 |
| TP1 = 5 V, never 20 V | CC lines / CFG straps R18–R20, source's 20 V capability, TP2 present |
| TP1 = 20 V, TP7 dead | UVLO (R22/R23), TP4 activity, U4 VCC at C23, FB divider R27/R28 |
| TP7 = 48 V, TP9 dead | TP3 state, J5 if bench-only, U2 itself |
| All rails up, no output | TP10 (drive present?), TP11 (bias?), driver stage on VDRV |
| Low gain / distortion | IDQ setting, input pad value, supply sag at TP8 under drive |
| Dump resistors hot | §6.3 patterns — bias, oscillation, LPF |
| Blows F1 repeatedly | D6 shorted, C29/C30 shorted, Q6 failed short (SW2 reads 0 Ω to GND) |

---

## 8. Fabrication & rework notes

- 2-layer, 1 oz copper. B.Cu is a ground plane with a handful of signal crossings
  (gate drives, USB pair, PD_GOOD run, small-signal reroutes) — if you cut the plane
  during rework near U4 or the USB section, check those.
- The MRF101 tabs and the TO-252 FETs are the significant thermal masses for rework:
  preheat.
- U3 (0.5 mm pitch) and U4 (0.65 mm pitch, exposed pad) are the fine-pitch parts;
  everything else is hand-iron friendly.
- Known-accepted DRC baseline: U2's footprint annular/clearance quirks, U4's
  footprint-intrinsic 0.19 mm pad-gap flags, SMA edge-launch copper (deliberate),
  and same-net zone overlaps at the y = 107 mm seam. Roughly 60 items total; anything
  *beyond* that set after a modification deserves a look.
