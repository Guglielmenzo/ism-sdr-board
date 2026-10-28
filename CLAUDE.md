# CLAUDE.md — ism-sdr-board

## Current status (updated 2026-10-28)

- Direction changed: the custom SDR board now targets the US 902–928 MHz ISM band,
  not 137 MHz, and the folder was renamed from `noaa-sdr-decoder` to `ism-sdr-board`.
  The 137 MHz plan is archived in `docs/archive/sdr-pcb-137mhz-plan.md`; satellite
  reception is parked (NOAA APT ended Aug 2025).
- Guide (Claude Doc, the source of truth for specs, parts, weekend plan, roadmap):
  https://claude.ai/code/artifact/b8735478-48b9-4492-854e-565c0434db51
  Keep the README's "v1 targets" and "Sections" tables in sync with it.
- Owned: Nooelec NESDR SMArt v5 (R820T2 + RTL2832U, TCXO, **no bias tee**) — the
  reference receiver; it tunes 915 MHz.
- Front-end sheet design is in `hardware/01-frontend/` (spec file there).
  Altium Designer (the EDA tool; not KiCad) and LTspice are installed. Nothing ordered yet.
- No code written. Python is **not installed** on this Windows machine (only the
  Microsoft Store alias) — install it before running anything in `software/`.

## Decisions so far

- Band: 902–928 MHz (US ISM; also the 33 cm ham band). 433 MHz is a possible second
  band (the QPL9547 covers it; swap the SAW filters). 2.4 GHz deferred.
- Architecture: direct conversion to low IF, I/Q demodulator, PLL LO from a TCXO,
  dual-channel digitizer, USB. **2026-10-21: everything goes on ONE 4-layer board**
  (front end, tuner, on-board STM32F072CBT6 controller, power), multi-sheet Altium
  project with three sheets: FrontEnd (refs as before), Tuner (ADL5380, ADF4351,
  baseband AND the STM32, so SPI stays local; tuner refs 2xx, STM32 parts 3xx), Power.
  Specs per sheet: hardware/01-frontend/, hardware/02-tuner/, hardware/power/.
  Test breaks keep sections measurable alone: C6 steers to J2 (SMA test port) or the
  tuner; R203 0 Ω + DNP u.FL J201 on the LO. MCU/USB at the far end from the antenna
  (48 MHz x19 = 912 MHz, 12 MHz x76 = 912, 25 MHz x37 = 925 land in band).
- Front end (old Board 1): SMA → 47 nH shunt → Abracon AFS20A26-915.00 SAW → Qorvo QPL9547 LNA
  (alt TQP3M9037, same footprint; leave R4 off) → SAW → 100 pF DC block → SMA.
  LNA circuit copied from Qorvo's QPL9547EVB-01. Power now from one data USB-C
  (GCT USB4105-GF-A, 5.1 kΩ CC pulldowns) on the control/power sheet.
  Optional: DNP shunt pads around the LNA; 0 Ω steering pads to bypass SAW1.
- Tuner (old Board 2): ADI ADL5380 I/Q demodulator + Mini-Circuits TC1-1-13 baluns,
  ADI ADF4351 LO on the board, 25 MHz ±0.5 ppm TCXO, STM32F072CBT6 control (user
  chose it over STM32G031, which has no USB), dual op-amp baseband with ~90 kHz
  low-pass. ~0.5 A at 5 V total.
- Digitizer: v1 a bought USB audio interface (192 kS/s, e.g. Behringer UMC204HD);
  rev B adds AD9238 + iCE40 FPGA + FT232H on the board, with the friend.
- Fab: JLCPCB 4-layer JLC04161H-7628 with impedance control; RF as 50 Ω grounded
  CPWG on top over solid L2 (≈0.3–0.35 mm, confirm in JLCPCB's calculator);
  LCSC numbers as Altium component parameters; Gerbers, BOM and pick-and-place
  from an Altium Output Job in JLCPCB's format. SAWs from Mouser.
- Bench (owned, 2026-10-21): NanoVNA and tinySA; no LiteVNA purchase. NanoVNA runs on
  harmonics near 900 MHz, so deep SAW rejection may hit its floor. Still to get: SMA
  attenuators, a 915 MHz LoRa board as a test transmitter.
- Predicted front end + RTL-SDR: 20.9 dB gain, NF 2.5 dB SAW-first vs 0.75 dB
  LNA-first (RTL-SDR NF assumed 6 dB). Verify by measurement.
- Out of v1 scope: transmit, 2.4 GHz, FPGA processing (until rev B).

## Next steps

1. Schematic (front-end sheet first) + layout (checklist in the guide's
   "Schematic, layout and order" section),
   get it reviewed, order from JLCPCB; SAW filters from Mouser.
2. Install rtl_433, rtlamr and SDR++; record what the bare
   RTL-SDR decodes at 915 MHz in 15 minutes as a baseline.
3. Bring up the front end (S21/S11, then in front of the RTL-SDR) and log predicted vs
   measured in `hardware/design-log.md`.
4. Then (same board, staged): controller + LO first light → tuner → SDR v1 with
   the sound card → rev B with the fast digitizer. See the guide's Roadmap.

## Notes for future sessions

- Keep guidance concrete: exact parts, reference designs, step lists with
  done-when criteria, one section at a time.
- Preferred language: Python (not MATLAB).
- This folder is inside the user's Obsidian vault (`D:\Engineering`), so these
  markdown files are vault notes. READMEs use unique names (`README-<project>.md`)
  for that reason; keep the convention. Exception: the root `README.md` is the GitHub
  front page (repo github.com/Guglielmenzo/ism-sdr-board); `README-ism-sdr-board.md`
  is a short vault stub pointing to it. Folder READMEs under `hardware/` are GitHub indexes.
- Commits to the GitHub repo carry no Claude co-author or session trailers (user's request). Renaming or moving files here does not
  update vault links: fix references in `RF Roadmap.md`, `Projects.md`,
  `SDR System Architecture.md`, `Software Defined Radio.md` and `Sat Dump Build.md`.
- `data/` is gitignored — don't assume recordings are tracked in git.
- Ties into DSP coursework: explain the RF/DSP "why" alongside implementation.
