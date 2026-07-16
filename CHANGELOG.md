# Changelog

## rev A (2026-07) — initial public release

USB-C powered 40m AM linear amplifier, 10 W carrier / 40 W PEP.

### RF section
- MRF101AN/BN push-pull output pair, 48 V drain, temperature-tracked bias
  (IDQ 100 mA/device).
- 1:4 input transformer (BN-43-202), 4:1 output combiner (BN-43-3312).
- 7-element Chebyshev low-pass output filter, 7.5 MHz cutoff.
- Absorptive harmonic termination: 3rd-order Butterworth high-pass
  (C33/L8/C34, f_c ≈ 10 MHz) into a 50 Ω / 6 W dump (R34‖R35) hung on the
  pre-filter node — the PA sees a resistive load at all frequencies.

### Power section
- USB-C input (J4) with TVS clamp (D6) and fuse (F1).
- CH224K PD sink controller (U3) configured for 20 V, with CC/DP/DM fanout
  and PD_GOOD interlock.
- LM5122 synchronous boost (U4, Q6/Q7, L6, R29 sense): 20 V → 48 V, with
  L7/C28 post-filter.
- Schottky ORing (D1/D4) of the boost rail against a 48 V bench input (J3).
- Driver-supply interlock: LM2596HV (U2, 12 V VDRV) enabled by PD_GOOD;
  J5 force-enable jumper for bench-only operation.
- Status LEDs: D7 (green, PD-OK) and D8 (amber, VDD present).

### Board
- 220 × 130 mm, 2-layer, 1 oz.
- DRC clean to documented baseline (see SERVICE_MANUAL.md §8); connectivity
  verified including ratsnest.
