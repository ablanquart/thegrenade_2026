# MRF101 40m AM Linear Amplifier — USB-C Powered

A bench-oriented 40-meter (7.0–7.3 MHz) AM linear amplifier built around a push-pull
pair of NXP **MRF101AN/MRF101BN** LDMOS transistors — powered from a standard 100 W
USB-C PD supply. Designed for experimental use under an FCC Part 5 license and
general amateur use on 40m.

**10 W carrier / 40 W PEP · 50 Ω in/out · runs from a laptop brick · absorptive harmonic termination**

## Features

- **USB-C powered**: negotiates 20 V / 5 A from any 100 W PD supply via a CH224K sink
  controller, then boosts to 48 V with an LM5122 synchronous boost converter.
- **Dual power path**: the boost output and a 48 V bench input are diode-ORed onto the
  main VDD rail — plug in whichever you have; both is fine.
- **Absorptive harmonic termination**: a high-pass diplexer branch dumps everything the
  output low-pass filter rejects into a 50 Ω / 6 W resistive load instead of reflecting
  it back into the PA drains. The transistors see a resistive load at all frequencies.
- **Interlocked bring-up**: the driver supply (LM2596HV buck) is enabled only after PD
  negotiation succeeds, or manually via a force-enable jumper (J5).
- **Status LEDs**: green = PD negotiated and boost alive; amber = main 48 V rail present.
- Temperature-tracked gate bias for the output pair.
- 7-element Chebyshev low-pass output filter, 7.5 MHz cutoff.

## Repository layout

```
├── mrf101_40m_pa_usbpd.kicad_pro   # KiCad project
├── mrf101_40m_pa_usbpd.kicad_sch   # Schematic
├── mrf101_40m_pa_usbpd.kicad_pcb   # Board (220 × 130 mm, 2-layer)
├── BOM_mrf101_40m_pa_usbpd.csv     # Bill of materials (grouped, with quantities)
├── fab_usbpd/                      # Gerbers, drill files, paste layers
├── docs/
│   ├── USER_GUIDE.md               # Setup and operation
│   └── SERVICE_MANUAL.md           # Theory of operation, component reference, test points
├── CHANGELOG.md
└── README.md
```

## Quick start

1. Bolt the board to a heatsink — the MRF101 pair **requires** one (TO-220 tab-down).
2. Connect a 50 Ω dummy load or antenna system to **J2 (RF OUT)**. Never key without a load.
3. Power: plug a 100 W USB-C PD supply into **J4**, or 48 V bench into **J3**.
4. Green LED (D7) = PD negotiated; amber LED (D8) = 48 V rail up.
5. Apply drive (≈1 W nominal) to **J1 (RF IN)**.

Full setup, bias adjustment, and first-power-up procedures are in
[docs/USER_GUIDE.md](docs/USER_GUIDE.md). Circuit theory, a complete component function
reference, and oscilloscope test points are in
[docs/SERVICE_MANUAL.md](docs/SERVICE_MANUAL.md).

## Bill of materials

The complete grouped BOM is [`BOM_mrf101_40m_pa_usbpd.csv`](BOM_mrf101_40m_pa_usbpd.csv).
Sourcing notes for the parts that matter:

| Part | Refs | Notes |
|---|---|---|
| MRF101AN + MRF101BN | Q1, Q2 | Buy the AN/BN *pair* — mirrored pinouts for layout symmetry. |
| CH224K | U3 | USB-PD sink controller, SSOP-10. |
| LM5122 | U4 | Sync boost controller, HTSSOP-20 (exposed pad). |
| LM2596HV-12 | U2 | HV variant is mandatory — input is 48 V. |
| BN-43-202, BN-43-3312 | T1, T2 | Binocular ferrite cores; wind per schematic. |
| 330 pF C0G **250 V** | C33, C34 | Diplexer series caps — voltage rating matters. |
| 390 nH, Q > 50 @ 20 MHz | L8 | 1812 air-core/ceramic, or wind 8t on a T37-6. |
| 100 Ω **3 W** 2512 | R34, R35 | The harmonic dump — do not substitute smaller. |
| USB-C 16-pin receptacle | J4 | Mid-mount, 5 A rated. |

## Regenerating fabrication files

With KiCad 10.x:

```sh
KCLI=/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli   # adjust for your OS

$KCLI pcb export gerbers mrf101_40m_pa_usbpd.kicad_pcb -o fab_usbpd/
$KCLI pcb export drill   mrf101_40m_pa_usbpd.kicad_pcb -o fab_usbpd/ \
      --excellon-units mm --excellon-separate-th
```

Include the paste layers (F.Paste/B.Paste) if you're having a stencil made.

## Safety

This is a 40-watt RF power amplifier running on a 48-volt rail. Treat it accordingly:
hot heatsinks, RF voltage on exposed copper, and enough stored energy in the bulk
capacitors to be memorable. Operate into a proper load, within your license privileges,
and keep fingers off the output network while transmitting.

## License

TBD — pick one before publishing (CERN-OHL-S is a common choice for open hardware;
MIT/BSD if you want it maximally permissive).

---
*73 de the bench.*
