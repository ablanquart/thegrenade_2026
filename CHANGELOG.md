# Changelog

## mrf101_40m_pa_usbpd — rev A (2026-07)

USB-C PD variant, derived from the SMT board.

### Added
- USB-C power input (J4) with TVS clamp (D6) and fuse (F1).
- CH224K PD sink controller (U3) configured for 20 V, with CC/DP/DM fanout,
  CFG straps, and PD_GOOD interlock.
- LM5122 synchronous boost converter (U4, Q6/Q7, L6, R29 sense) producing 48 V
  from the negotiated 20 V rail, with L7/C28 post-filter.
- Schottky ORing (D4) of the boost rail against the existing bench input (D1),
  allowing dual-supply operation.
- Driver-supply interlock: LM2596HV enable driven by PD_GOOD; J5 force-enable
  jumper for bench-only operation.
- Absorptive harmonic termination on the PA output node: 3rd-order Butterworth
  high-pass (C33/L8/C34, f_c ≈ 10 MHz) into a 50 Ω / 6 W dump (R34‖R35).
- Status LEDs: D7 (green, PD-OK from VDD_BOOST) and D8 (amber, VDD present).
- Board outline extended to 220 × 130 mm to carry the power strip.

### Changed
- U2 (LM2596HV) enable net moved from GND to PD_GOOD.
- GND stitching densified around U3/U4/Q6/Q7.

### Board status
- DRC: clean to the accepted baseline (~60 items: legacy footprint quirks,
  deliberate SMA edge-launch copper, same-net zone overlap, silk).
- Connectivity verified via GUI DRC including ratsnest.

## mrf101_40m_pa_tht — rev A

Through-hole variant of the SMT board. Optional RD06HHF1 driver stage (Q5,
bias via RV2, shared heatsink). Fabrication files in `fab_tht/`.

## mrf101_40m_pa — rev A

Original SMT bench variant. 48 V bench supply only. Fabricated and verified.
