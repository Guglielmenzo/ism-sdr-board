# Archived: 137 MHz SDR PCB plan

Archived 2026-10-03, when the board moved to the 902–928 MHz ISM band. Kept for
reference: the architecture, modular build approach, bench kit and learning path
still apply. The current plan is in the README and the
[ISM-Band SDR Board — Project Guide](https://claude.ai/code/artifact/b8735478-48b9-4492-854e-565c0434db51).

Original guide (137 MHz):
https://claude.ai/code/artifact/e3947771-cfda-4de7-8b77-4126b8c95aca

Target update: NOAA decommissioned NOAA-19 on 2025-08-13 and NOAA-15 on
2025-08-19 ([NOAA OSPO](https://ospo.noaa.gov/data/messages/2025/08/MSG_20250820_1410.html)),
so APT was gone and Meteor-M LRPT was the remaining 137 MHz target.

---

The section below is the README's "Custom SDR PCB (v1)" text as it stood before
the change.

## Custom SDR PCB (v1)

Receive-only board for the 137 MHz weather-satellite band that streams I/Q to a PC.
Focus is RF hardware; firmware/host software will be existing or written by a friend.
Full guide: https://claude.ai/code/artifact/e3947771-cfda-4de7-8b77-4126b8c95aca

**Target signal — unverified:** NOAA-15/18/19 APT may have been shut down in 2025.
If so, Meteor-M LRPT (~137.1 / 137.9 MHz, ~120 kHz wide) is the target. Confirm on a
current satellite-status page before fixing the band plan.

**Architecture:** low-IF direct conversion (LO 25–50 kHz off the signal), integrated
I/Q demodulator, PLL LO from a shared TCXO, two-channel ADC at ~192 kS/s, USB
(stereo USB audio codec is the simplest path). Build each block as its own SMA-to-SMA
board first, then integrate. SA602A superhet = side experiment, not the v1 core.

```
Antenna → Protection/bias-tee → Preselector → LNA → Post-LNA filter → I/Q mixer
       → Baseband filter + gain (I, Q) → Dual ADC → USB → PC
                       TCXO → PLL LO ──┘ (TCXO also clocks the ADC)
```

**v1 system targets**

| Parameter | Target |
|---|---|
| Tuning range | 136–138 MHz (stretch 100–180 MHz) |
| Instantaneous bandwidth | ≥ 150 kHz |
| System noise figure | ≤ 3 dB at antenna port |
| FM (88–108 MHz) rejection | ≥ 40 dB |
| Image rejection | ≥ 35 dB (hardware; software adds 20–30 dB) |
| Frequency accuracy | ≤ 2 ppm |
| Input | 50 Ω SMA, ESD-protected, optional bias-tee |
| Host / power | USB 2.0, < 500 mA from 5 V |

**Per-stage targets**

| Stage | Key targets |
|---|---|
| Antenna port | Return loss ≥ 10 dB in-band; ESD diode < 1 pF |
| Preselector | IL ≤ 1.5 dB; ≥ 40 dB at FM, ≥ 30 dB at 150–175 MHz |
| LNA | 15–20 dB gain, NF ≤ 1 dB, OIP3 ≥ +25 dBm, unconditionally stable (K > 1) |
| Post-LNA filter | IL ≤ 3 dB; ≥ 50 dB at FM |
| I/Q mixer | IIP3 ≥ +10 dBm; I/Q balance ≤ 0.2 dB / ≤ 2°; LO-RF isolation ≥ 40 dB |
| LO + reference | ≤ 1 kHz step; ≤ −90 dBc/Hz @ 10 kHz; spurs ≤ −60 dBc; TCXO ±0.5–2 ppm |
| Baseband | 20–40 dB gain, matched I/Q anti-alias filters, AC coupling below the low-IF |
| ADC | 2 ch simultaneous, ≥ 192 kS/s, 24-bit audio-class (≥ 100 dB DR) |
| Power | Separate LDOs for RF, LO, ADC analog, digital; no switchers in v1 |
| Layout | 4-layer, 50 Ω grounded coplanar, solid ground, shield-can footprints, test points |

Watch for USB/MCU clock harmonics in-band (48 MHz × 3 = 144 MHz).

**Roadmap**

0. Requirements + cascade budget spreadsheet (gain / NF / IP3 per stage)
1. Simulate filters and matching (Qucs-S / LTspice, vendor S-parameters)
2. Front-end board (preselector + LNA + post-filter) — test it in front of the RTL-SDR
3. Mixer + LO board — I/Q on a scope, image rejection ≥ 30 dB
4. Baseband + ADC + USB — test tone appears in SDR software
5. Integrated rev A — receives a real satellite pass
6. Measure + rev B — match or beat the RTL-SDR in-band

**Bench kit to get before the first board:** NanoVNA/LiteVNA + cal kit, TinySA Ultra,
step attenuator + loads + SMA adapters, broadband noise source (Y-factor NF).
