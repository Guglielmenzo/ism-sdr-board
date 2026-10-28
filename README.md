# ISM-Band SDR Board

A receive-only software-defined radio for the US 902–928 MHz ISM band, on one 4-layer
board: a SAW-filtered LNA front end, a direct-conversion I/Q tuner with an on-board
synthesizer, and an STM32 controller. I and Q come out at audio frequencies and go into
a USB audio interface, so SDR++, rtl_433, rtlamr and gr-lora_sdr can decode LoRa,
Meshtastic, utility meters and sensors.

**Status:** schematic capture in Altium Designer. Nothing ordered yet.

```
Antenna ─► Front end: SMA → SAW → LNA (QPL9547) → SAW ─C6─► J2 test SMA
                                                       └──► Tuner: balun → ADL5380 I/Q demodulator
                                       ADF4351 LO + 25 MHz TCXO ──► (u.FL test port)
                                       STM32F072 ─SPI─► LO, USB-C ◄─► PC
            I and Q baseband filters ─► jacks ─► USB audio interface ─► PC
```

## Targets (v1)

| Parameter | Target |
|---|---|
| Tuning range | 902–928 MHz |
| Instantaneous bandwidth | ≥ 150 kHz (192 kS/s audio interface) |
| System noise figure | ≤ 3 dB at the antenna port |
| Cell band (698–894 MHz) rejection | ≥ 40 dB before the LNA |
| Image rejection | ≥ 35 dB in hardware, software corrects the rest |
| Frequency accuracy | ≤ 2 ppm (≈ 1.8 kHz at 915 MHz) |
| Input | 50 Ω SMA |
| Power | USB-C 5 V with data, about 0.5 A |

## Main parts

| Section | Part | Job |
|---|---|---|
| Front end | Abracon AFS20A26-915.00 (×2) | 902–928 MHz SAW filters before and after the LNA |
| Front end | Qorvo QPL9547 | LNA, 24.6 dB gain, 0.38 dB NF at 900 MHz |
| Tuner | ADI ADL5380 + Mini-Circuits TC1-1-13M+ baluns | I/Q demodulator |
| Tuner | ADI ADF4351 + 25 MHz ±0.5 ppm TCXO | Local oscillator |
| Tuner | ADI ADA4807-2 (×2) | I and Q difference amp and 90 kHz low-pass |
| Control | ST STM32F072CBT6 | Tunes the LO over SPI, USB to the PC |
| Power | GCT USB4105-GF-A, ferrite beads, LDOs | One filtered branch per section |

## Repository layout

| Path | Contents |
|---|---|
| [`docs/architecture.md`](docs/architecture.md) | Signal chain, sheets, isolation and test breaks |
| [`docs/roadmap.md`](docs/roadmap.md) | Staged bring-up with done-when criteria |
| [`hardware/`](hardware/) | Per-sheet specs, power tree, design log |
| [`hardware/datasheets/`](hardware/datasheets/) | Datasheets and their reference circuits |
| [`docs/archive/`](docs/archive/) | The retired 137 MHz weather-satellite plan |
| `software/`, `notebooks/` | Python IQ capture and DSP experiments (to come) |

## Build

- **EDA:** Altium Designer, a multi-sheet project (FrontEnd, Tuner, Power) with flat
  net scope: ports between sheets, power ports for rails.
- **Fab:** JLCPCB 4-layer JLC04161H-7628 with impedance control. RF runs as 50 Ω
  grounded coplanar waveguide on the top layer over a solid L2 ground.
- **Assembly:** LCSC numbers as Altium component parameters; Gerbers, BOM and
  pick-and-place from an Altium Output Job in JLCPCB's format. SAW filters from Mouser.

## Test gear

NanoVNA (S21/S11), tinySA (spectrum and spurs), a Nooelec NESDR SMArt v5 as the reference
receiver, SMA attenuators, and a 915 MHz LoRa board as a known transmitter.

## Software setup

```
python -m venv .venv
.venv\Scripts\activate
pip install -e .
```
