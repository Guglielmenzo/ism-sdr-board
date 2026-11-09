# ISM-Band SDR Board

[[RF Roadmap]] [[SDR System Architecture]] [[Software Defined Radio]] [[Projects]]

Custom receive-only SDR for the US 902–928 MHz ISM band, built as one 4-layer board
(front end, tuner, on-board STM32 controller) with test breaks so each section can
still be measured on its own. Focus is RF hardware; firmware is existing or written by a
friend. Renamed from `noaa-sdr-decoder` on 2026-10-03, when the board moved off the
137 MHz weather-satellite band; the old plan is archived in
`docs/archive/sdr-pcb-137mhz-plan.md`.

Project guide (specs, block diagram, parts, weekend plan, roadmap):
https://claude.ai/code/artifact/b8735478-48b9-4492-854e-565c0434db51

## Status (2026-10-03)

- Direction changed from 137 MHz weather satellites to the 902–928 MHz ISM band.
- RTL-SDR (Nooelec NESDR SMArt v5) works on Windows (Zadig/WinUSB, validated on FM in
  SDR#). It tunes 915 MHz, so it is the reference receiver. It has no bias tee.
- 2026-10-21: plan is now one board with an on-board STM32F072. Front-end sheet in
  `hardware/01-frontend/`. Nothing ordered yet.
- No code yet. Python is not installed on this PC yet (see Setup).

## System

```
One board:
Antenna ─► [Front end: SMA → SAW → LNA → SAW] ─C6─► J2 test SMA (RTL-SDR / VNA)
                                              └──► [Tuner: I/Q demodulator (ADL5380)
                                                    ◄─ LO (ADF4351, u.FL test port) ◄─ 25 MHz TCXO
                                                    baseband filter + gain for I and Q]
           [Control/power: STM32F072 ─SPI─► LO; USB-C with data to the PC]
   ─► I/Q jacks ─► bought USB audio interface (rev B: on-board AD9238 + iCE40 + FT232H)
   ─► USB ─► PC (SDR++, rtl_433, rtlamr, gr-lora_sdr)
```

## v1 targets

| Parameter | Target |
|---|---|
| Tuning range | 902–928 MHz (433 MHz variant possible by swapping the SAW filters) |
| Instantaneous bandwidth | ≥ 150 kHz (v1, sound card); ≥ 2 MHz (v2 digitizer) |
| System noise figure | ≤ 3 dB at the antenna port |
| Cell band (698–894 MHz) rejection | ≥ 40 dB before the LNA |
| Image rejection | ≥ 35 dB in hardware; software corrects the rest |
| Frequency accuracy | ≤ 2 ppm (≈ 1.8 kHz at 915 MHz) |
| Input | 50 Ω SMA |
| Power | USB-C 5 V with data; about 0.5 A total |

## Sections (one board)

| Section | Contents | Status |
|---|---|---|
| Front end | 47 nH shunt → Abracon AFS20A26-915.00 SAW → Qorvo QPL9547 LNA (alt TQP3M9037) → SAW → C6 to J2 test port or the tuner | Designing |
| Tuner | ADI ADL5380 I/Q demodulator + TC1-1-13 baluns, ADI ADF4351 LO on board, 25 MHz TCXO, dual op-amp baseband | Next |
| Control | STM32F072CBT6, drawn on the Tuner sheet, placed at the far end | Next |
| Power | USB-C with data (USB4105-GF-A), a ferrite bead per branch, LDOs for the 3.3 V rails | Next |
| Digitizer | v1 bought USB audio interface (192 kS/s); rev B AD9238 + iCE40 FPGA + FT232H | Later |

Predicted front end + RTL-SDR: 20.9 dB gain, 2.5 dB noise figure with the SAW first
(0.75 dB with the LNA first). Measure and log in `hardware/design-log.md`.

## Bench kit

NanoVNA and tinySA (owned), SMA attenuators and 50 Ω loads, a 915 MHz LoRa or Meshtastic board as a
test transmitter, and the RTL-SDR as the reference receiver.

## Software

- Existing tools first: rtl_433 (sensors), rtlamr (utility meters), SDR++, gr-lora_sdr,
  Universal Radio Hacker.
- `software/capture/` — Python IQ capture at 915 MHz (pyrtlsdr).
- `software/decode/` — analysis and decoding experiments.
- `notebooks/` — exploratory DSP before code moves into `software/`.

## Parked: 137 MHz weather satellites

NOAA decommissioned its last APT satellites in August 2025. Meteor-M LRPT was the remaining
137 MHz target (V-dipole + RTL-SDR + existing decoders; see [[Sat Dump Build]]). Old plan
and specs: `docs/archive/sdr-pcb-137mhz-plan.md`.

## Structure

- `hardware/01-frontend/` — the Altium project and the FrontEnd sheet spec
- `hardware/02-tuner/` — Tuner sheet spec (ADL5380, ADF4351, baseband, STM32)
- `hardware/power/` — Power sheet spec and the rail tree
- `hardware/datasheets/` — datasheets for the chosen parts
- `hardware/design-log.md` — decisions, predictions vs measurements
- `docs/` — setup notes; `docs/archive/` — retired plans
- `software/capture/`, `software/decode/`, `notebooks/`
- `data/` — IQ recordings (gitignored — gets large)

## Setup

Python isn't installed on this PC yet. Install Python 3.10+ (python.org or
`winget install Python.Python.3.12`), then:

```
python -m venv .venv
.venv\Scripts\activate
pip install -e .
```

See `pyproject.toml` for dependencies.
