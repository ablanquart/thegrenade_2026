# User's Guide — MRF101 40m AM Linear Amplifier (USB-PD variant)

This guide covers setup and day-to-day operation. For circuit theory, component
functions, and test points, see [SERVICE_MANUAL.md](SERVICE_MANUAL.md).

## 1. What you need

- A **heatsink** for the MRF101 pair (Q1/Q2, TO-220 tab-down). At 10 W carrier AM the
  devices dissipate continuously — do not run the board without one, even briefly.
  Use thermal compound; insulating washers are not required (tabs are source/ground —
  verify against your heatsink grounding scheme before assuming).
- A **50 Ω load**: dummy load for testing, or a matched antenna system. The amplifier
  must never be keyed without a load on J2.
- **Power**, one or both of:
  - A **USB-C PD supply rated 100 W (20 V / 5 A)**. Lower-rated supplies (e.g. 65 W)
    will negotiate but will current-limit under full drive — fine for receive-level
    testing, not for transmitting.
  - A **48 V bench supply**, 2 A or better, into J3.
- A **drive source**: approximately 1 W of clean 7 MHz drive at J1 produces full output.
  An input attenuator pad is provided on-board for level matching.

## 2. Connections

| Connector | Function |
|---|---|
| **J1** | RF input, SMA, 50 Ω |
| **J2** | RF output, SMA, 50 Ω — load goes here, always |
| **J3** | 48 V bench supply input |
| **J4** | USB-C power input (PD, 20 V negotiated) |
| **J5** | Driver-supply force-enable jumper (see §5) |

Both power inputs may be connected simultaneously; Schottky ORing diodes (D1, D4)
select whichever rail is higher and prevent back-feeding.

## 3. Status LEDs

| LED | Color | Meaning when lit |
|---|---|---|
| **D7** | Green | USB-PD negotiation succeeded **and** the 48 V boost converter is running |
| **D8** | Amber | The main 48 V rail (VDD) is up — from either the boost or the bench input |

Normal USB-C operation: both LEDs lit. Bench-only operation: amber only.
Green off with USB-C connected means PD negotiation failed — see §7.

## 4. First power-up (do this once per new build)

Work through this before ever applying RF drive:

1. **Visual inspection.** Solder bridges around U3 (SSOP-10) and U4 (HTSSOP-20) are the
   most likely build faults. Check the electrolytics (C26, C27) for polarity.
2. **Resistance checks, power off.** VDD to GND and VBUS_F to GND should both read
   high (>1 kΩ rising as caps charge on the meter). A dead short here means stop.
3. **USB power, no drive, no load required yet.** Plug in a PD supply. Within about a
   second you should get: green LED, amber LED. Verify with a meter:
   - 20 V on VBUS_F (across C29/C30)
   - 48 V on VDD_BOOST (across C28)
   - 48 V on VDD
   - 12 V on VDRV (U2 output) — this rail only comes up when PD_GOOD is asserted
   - 3.3 V on U3_VDD (across C31)
4. **Set the quiescent bias.** With a dummy load on J2, no drive, and the heatsink
   attached: monitor supply current and adjust the bias trimmer for **100 mA of drain
   current per device** (200 mA total increase over the no-bias baseline). The bias
   network is temperature-compensated; let the heatsink warm and confirm IDQ stays
   put. See the service manual §4 for the bias network description.
5. **Low drive test.** Apply ~100 mW of 7.0–7.3 MHz drive. Confirm output on a scope
   or power meter, then bring drive up gradually while watching supply current and
   output power. Full drive (~1 W) should yield ≈10 W of carrier.

## 5. The J5 jumper (force-enable)

The driver supply (U2, 12 V) is normally enabled by the CH224K's power-good signal —
the amplifier stays quiet until USB-PD negotiation has actually delivered 20 V.

When running from the **bench supply only** (nothing on USB-C), there is no PD_GOOD,
so fit a **jumper across J5** to enable the driver rail manually. Remove it when
running from USB-C if you want the negotiation interlock back (recommended).

## 6. Operating notes

- **Duty cycle.** AM carrier means continuous dissipation. Watch heatsink temperature
  during long transmissions; the MRF101 is rugged but not immortal.
- **Harmonics.** The output filter plus the absorptive dump network keep harmonic
  output clean and keep the PA stable into imperfect loads. The dump resistors
  (R34/R35, near the output filter) run warm in normal operation — a few degrees
  above ambient. If they run **hot**, something is wrong upstream (see the service
  manual troubleshooting table).
- **SWR.** The amplifier tolerates moderate mismatch thanks to the absorptive
  termination, but it is not protected against gross faults. Aim for < 2:1.
- **PD supplies.** Any USB-C source that advertises a 20 V PDO works. Sources that
  top out at 15 V will light neither the boost nor the green LED — the CFG straps
  request 20 V specifically and the design assumes it.

## 7. Quick troubleshooting

| Symptom | Likely cause |
|---|---|
| No LEDs on USB-C | Bad cable (must be a full-featured/e-marked cable for 5 A), F1 fuse open, TVS D6 failed short |
| Amber on, green off (USB-C) | PD negotiation failed: 5 V-only or 15 V-max source, or CC-line issue |
| LEDs on, no RF output | No drive at J1; J5 jumper missing on bench-only power; VDRV rail down; bias mis-set |
| Output low and distorted | IDQ set too low; drive overloading the input pad; supply current-limiting (undersized PD source) |
| Dump resistors hot | Excess harmonic energy: check bias (crossover distortion), check for parasitic oscillation, verify LPF assembly |
| Boost whines / green LED flickers | PD source current-limiting under load — use a genuine 100 W supply |

Deeper diagnosis with an oscilloscope: service manual §6.

## 8. Care and feeding

Keep the heatsink interface clean, keep RF connectors torqued, and re-check IDQ after
the first few thermal cycles of a new build. The bulk electrolytics are the only
components with a wear-out mechanism; everything else should outlive your interest
in 40 meters, which is saying something.
