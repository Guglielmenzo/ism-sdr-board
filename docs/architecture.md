# Architecture

One 4-layer board, drawn as three Altium sheets. Each section can still be measured on
its own through test breaks, which is what the original three-board plan was for.

## Signal chain

1. **Front end** (FrontEnd sheet): edge SMA → 47 nH static drain → SAW1 → QPL9547 LNA →
   SAW2 → C6. Predicted 20.9 dB gain and 2.5 dB system NF with the SAW first.
2. **Tuner** (Tuner sheet): C6 → TC1-1-13M+ balun → 100 pF series caps → ADL5380 RF
   input. The ADF4351 LO drives the second balun into LOIP/LOIN. I and Q leave as
   differential pairs near 2.5 V.
3. **Baseband**: per channel, a difference amplifier then a 2nd-order low-pass near
   90 kHz (ADA4807-2), matched within 0.1 dB between I and Q.
4. **Digitizer**: a bought USB audio interface (Behringer UMC204HD, 192 kS/s) on the
   I and Q jacks. A fast on-board digitizer (AD9238 + iCE40 + FT232H) is rev B.
5. **Control**: STM32F072CBT6 on the Tuner sheet, placed at the far end of the board.
   SPI to the ADF4351, lock detect, ADL5380 enable, USB to the PC.

## Reference designators

| Sheet | Refs |
|---|---|
| FrontEnd | As drawn: U1, SAW1, SAW2, J1, J2, L1, L2, C1–C8, R2–R4, FB1 |
| Tuner | 2xx (U201 ADL5380, U202 ADF4351, T201/T202, U204/U205 op-amps); STM32 parts 3xx |
| Power | J3, R5, R6, D1, R7, U302, U303, FB301–FB304 |

## Test breaks

| Break | How | Lets you |
|---|---|---|
| Front end → tuner | C6 has two pads: A to J2 (SMA), B to T201 | Measure the front end alone, or use it in front of an RTL-SDR |
| LO → mixer | R203 0 Ω with a DNP u.FL (J201) beside it | Measure the LO, or inject a bought LO module |
| Rails | Test points on every supply branch | Check each section's current and noise |

Two of the five JLCPCB boards get only the front end and power fitted, so they work as a
stand-alone front end.

## Isolation

Every clock on the board has a harmonic inside 902–928 MHz:

| Clock | Lands at | Mitigation |
|---|---|---|
| STM32 48 MHz USB clock | 19 × 48 = 912 MHz | Far end of the board, own LDO, idle between tuning commands |
| USB full-speed 12 MHz | 76 × 12 = 912 MHz | USB-C at the far edge, short D+/D− pair |
| 25 MHz TCXO | 37 × 25 = 925 MHz | Short reference trace inside the LO area |
| SPI | Depends on rate | Only clocks while tuning |

Layout order from the antenna: front end, tuner, then controller and USB-C. Shield-can
ground rings go around the front end and the LO. L2 is one solid ground plane with no
splits; separation comes from placement and the per-branch beads and LDOs.

## Power

See [`hardware/power/power-spec.md`](../hardware/power/power-spec.md). All rails start
at USB 5 V; each section gets its own ferrite-bead branch, and the two 3.3 V rails
(STM32, LO) each have their own LDO.
