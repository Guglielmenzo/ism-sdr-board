# Design log — ISM-band SDR

One entry per decision or measurement: what, why, and predicted vs measured.
Write the prediction before you measure.

## 2026-10-03 — Retarget to 902–928 MHz

- Decision: the board targets the US 902–928 MHz ISM band instead of 137 MHz.
- Why: NOAA APT ended in Aug 2025; 915 MHz has signals around the clock (meters,
  sensors, LoRa), off-the-shelf SAW filters, small antennas, and a legal path to
  transmit later.
- Kept from the 137 MHz plan: direct conversion to low IF, one SMA-to-SMA board per
  stage, measure each block alone. Old plan: `../docs/archive/sdr-pcb-137mhz-plan.md`.

## 2026-10-03 — Board 1 parts

- LNA: Qorvo QPL9547 (JLCPCB C5185128). At 900 MHz: 24.6 dB gain, 0.38 dB NF,
  OIP3 +38.5 dBm; covers 0.1–6 GHz. Alternative TQP3M9037 (LCSC C415712) fits the
  same footprint.
- SAW filters: 2 × Abracon AFS20A26-915.00. 1.7 dB loss typ; 55 dB typ / 40 dB min
  rejection at 698–894 MHz; 50 Ω, no matching; 2.0 × 1.6 mm castellated.
- Power: USB-C 5 V, because the NESDR SMArt v5 has no bias tee.

## Board 1 predictions

| Quantity | Predicted | Measured | Notes |
|---|---|---|---|
| Gain at 915 MHz, SAW-first | 20.9 dB | | 0.3 dB connector/trace loss assumed |
| NF, board alone, SAW-first | 2.4 dB | | |
| NF, board alone, LNA-first | 0.7 dB | | SAW1 bypassed |
| NF, board + RTL-SDR, SAW-first | 2.5 dB | | RTL-SDR NF assumed 6 dB |
| Rejection at 880 MHz, end to end | ≥ 80 dB | | VNA range may limit |
| S11 in band | ≥ 10 dB | | |
| Supply current at 5 V | ≈ 65–75 mA | | LNA ≈ 65 mA + LED |

## 2026-10-21 — One board, on-board STM32

- Decision: front end, tuner and controller go on ONE 4-layer board; the separate
  Boards 1/2/3 plan is retired. A bought USB audio interface stays the v1 digitizer;
  the fast digitizer (AD9238 + iCE40 + FT232H) becomes rev B.
- Why: build it all at once, with the controller as an on-board chip instead of a
  Pico module.
- Kept from the old plan: each section measurable alone. C6 has two pads (J2 test
  SMA or the tuner); R203 0 Ω + DNP u.FL J201 break the LO; staged JLCPCB assembly
  (two of five boards with only the front end and power fitted).
- Controller: STM32F072CBT6 (crystal-less USB, USB bootloader, LQFP-48), picked over
  the STM32G0B1KBT6. The STM32G031Y8Y6TR was rejected: no USB, and an 18-ball WLCSP.
- Risk: clock harmonics in band (48 MHz × 19 = 912, 12 MHz × 76 = 912, 25 MHz × 37 =
  925 MHz). Controller and USB at the far end, own LDO, quiet after tuning,
  shield-can footprints over the front end and LO.
- Sheets: FrontEnd (refs as before), Tuner (ADL5380, ADF4351, baseband and the STM32;
  refs 2xx, STM32 parts 3xx), Power. Flat net scope: ports between sheets, power
  ports for rails.
- LO stays ADF4351 ($16.49, LCSC C71362) over the TI LMX2572 ($12.71, C2665711).
- Test gear: the NanoVNA and tinySA already owned replace the planned LiteVNA 64.
  The NanoVNA may hit its floor on SAW rejection near 900 MHz.

## 2026-10-27 — Tuner schematic review

- Fix: C201/C202 (100 pF) were drawn as shunts to ground on RFIP/RFIN; they must be
  in series. As shunts they're ≈ 1.7 Ω at 915 MHz and would short the signal.
- ADJ: R201 1.5 kΩ to the same filtered +5V_MIX rail as VCC1–3, no extra bead.
- Power: one bead per branch, LDOs for both 3.3 V rails. See `power/power-spec.md`.

## Rev A predictions (whole board)

| Quantity | Predicted | Measured | Notes |
|---|---|---|---|
| Total supply current at 5 V | ≈ 0.5 A | | ADL5380 245 mA, LNA 65 mA, rest estimated |
| Image rejection at 915 MHz | ≥ 35 dB | | Before software correction |
| LO frequency error | ≤ 2 ppm | | TCXO ±0.5 ppm |
| Controller spurs in band | Below the noise floor | | Antenna port terminated |
