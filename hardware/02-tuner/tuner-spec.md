# Tuner — Tuner sheet (spec)

Mixes the filtered 902–928 MHz band down to I and Q at audio frequencies, with the LO
on the board. The STM32 controller is on this sheet too (decided 2026-10-21), so the
SPI and enable lines stay local; in layout it still goes at the far end from the
antenna. Refs from 201 (STM32 parts keep 3xx). Full parts table in the guide:
https://claude.ai/code/artifact/b8735478-48b9-4492-854e-565c0434db51

## Targets

| Parameter | Target | How to check |
|---|---|---|
| Image rejection | ≥ 35 dB before software correction | 915 MHz tone in SDR++ |
| I/Q balance | Channels matched within 0.1 dB | Scope on I and Q, same tone |
| LO | 902–928 MHz, ≤ 2 kHz steps, within 2 ppm | tinySA / RTL-SDR at J201 |
| Baseband | Low-pass near 90 kHz, noise ≈ 10 dB above the sound card's | Sweep, SDR++ noise floor |
| Spurs | Nothing from the controller above the noise floor | Antenna port terminated, SDR++ |

## I/Q demodulator (ADL5380, data sheet Figure 78)

```
RF_FE_OUT (from C6 pad B)
 └─ T201 TC1-1-13M+ pin 6 (primary dot); pin 4 → GND; pins 2, 5 open
     ├─ pin 1 → C201 100 pF (SERIES) → RFIP (22)
     └─ pin 3 → C202 100 pF (SERIES) → RFIN (21)
LO: U202 RFOUTA → R203 0 Ω (test break; DNP u.FL J201 beside it)
    → [R204–R206 optional pi pad] → T202 pin 6; pin 4 → GND
     ├─ pin 1 → C203 100 pF (series) → LOIP (9)
     └─ pin 3 → C204 100 pF (series) → LOIN (10)
```

- The RF and LO coupling caps are in SERIES. A 100 pF cap to ground is ≈ 1.7 Ω at
  915 MHz and would short the signal (caught in the 2026-10-27 schematic review).
- VCC1/2/3 (6, 13, 24): +5V_MIX via FB301 + 10 µF + 100 nF; at each pin 0.1 µF + 100 pF.
- ADJ (19): R201 1.5 kΩ to +5V_MIX, the same filtered rail as VCC1–3 (no extra bead).
- ENBL (7): low = on. Tied to GND, or R202 10 kΩ pulldown plus an STM32 pin.
- NC (12) open. EP (25) and GND pins 1, 2, 5, 8, 11, 14, 17, 18, 20, 23 → GND,
  nine vias under the paddle. The Altium symbol is split: U3A signals, U3B power.

## LO (ADF4351) and reference

- U202 ADF4351 + Y201 25 MHz TCXO ±0.5 ppm, both on +3V3_LO from U203 (low-noise LDO,
  e.g. TPS7A20) fed from 5 V through its own bead.
- Loop filter values from ADIsimPLL; RFOUTA output network per the ADF4351 data sheet.
- ADF4351 kept over the TI LMX2572 ($12.71 vs $16.49 on LCSC, 2026-10-21): the
  saving didn't justify redoing the design and firmware.

## Baseband

- U204, U205 ADA4807-2, one per channel: difference amp, then 2nd-order low-pass
  near 90 kHz. +5V_BB via FB302. 2.5 V mid-rail reference shared by both channels.
- C211–C214 10 µF AC coupling on IHI/ILO/QHI/QLO. Thin-film resistors, C0G caps,
  identical values in I and Q. Filter values: not chosen yet.
- J202, J203: I and Q out to the USB audio interface.

## Controller (STM32F072CBT6, LQFP-48)

- +3V3_DIG from U303 (Power sheet). 100 nF per VDD/VDDIO2 pin, 4.7 µF bulk,
  VDDA 1 µF + 10 nF, NRST 100 nF, BOOT0 10 kΩ pulldown + button (USB DFU), SWD 2×5 1.27 mm.
- To the tuner: SPI (LO_CLK, LO_DATA, LO_LE, LO_CE), LO_MUXOUT (lock detect),
  optional ADL_ENBL, optional status LED. 3.3 V logic both sides, no level shifting.
- USB D+/D− to J3 on the Power sheet (ports USB_DP, USB_DM).
- Why not the STM32G031 the user first picked: no USB, and the Y suffix is an
  18-ball WLCSP.
