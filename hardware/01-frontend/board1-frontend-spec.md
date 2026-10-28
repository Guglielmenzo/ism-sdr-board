# Front end — FrontEnd sheet (spec)

SAW-filtered LNA for 902–928 MHz, the first sheet of the single 4-layer board
(2026-10-21: the old Boards 1/2/3 became one board). C6 steers its output either to
the J2 test SMA (RTL-SDR or VNA, front end alone) or into the tuner's balun T201.
Parts, reasons and sources are in the project guide:
https://claude.ai/code/artifact/b8735478-48b9-4492-854e-565c0434db51

## Targets

| Parameter | Target | How to check |
|---|---|---|
| Passband | 902–928 MHz | NanoVNA S21, 600–1200 MHz sweep |
| Gain at 915 MHz | ≈ 21 dB (SAW first) | S21 |
| Noise figure, board alone | ≈ 2.4 dB SAW-first; ≈ 0.7 dB LNA-first | Y-factor later; decode counts now |
| Cell band (698–894 MHz) | ≥ 40 dB down before the LNA; ≥ 80 dB end to end | S21 (the NanoVNA runs on harmonics here and may hit its floor) |
| Input and output return loss | ≥ 10 dB in band | S11, S22 |
| Stability | No oscillation anywhere 0–6 GHz | tinySA (High input / Ultra mode), input terminated |
| Supply | 5 V branch via FB1, ≈ 65 mA | Lift R3 for LNA current |
| Board | Section of the one board; JLCPCB 4-layer JLC04161H-7628, impedance controlled | — |

## Schematic, in signal order

```
J1 SMA (ANT IN)
 ├─ L1 47 nH to GND                    static drain
 ├─ [optional R8/R9 0 Ω: route through SAW1 or bypass it]
SAW1 Abracon AFS20A26-915.00           pad B = in, D = out, A and C = GND
 ├─ [DNP shunt pad]                    matching experiments
C1 100 pF C0G                          DC block
U1 Qorvo QPL9547 pin 2 (RF in) → pin 7 (RF out)
C2 100 pF C0G                          DC block
 ├─ [DNP shunt pad]
SAW2 Abracon AFS20A26-915.00
C6 100 pF C0G, two pads (fit one)      keeps bias-tee DC off SAW2 (3 V DC max)
 ├─ pad A → J2 SMA (front-end test port → RTL-SDR or VNA)
 └─ pad B → port RF_FE_OUT → T201 on the Tuner sheet
```

Optional pi footprints (shunt, series 0 Ω, shunt; 0402) at SAW1 → LNA and
LNA → SAW2, for matching fixes found with the NanoVNA.

U1 support — copy Qorvo's QPL9547EVB-01 schematic (data sheet p. 4) exactly:

- 5 V → R3 (0 Ω) → VDD node with C4 1 µF, C5 100 pF and C3 100 pF to GND.
- L2 18 nH wirewound from the VDD node to pin 7 (RF out doubles as the supply pin).
- R4 3.32 kΩ from the VDD node to pin 1 (Vbias) — sets ≈ 65 mA.
  For the TQP3M9037 alternative, leave R4 off (pin 1 is NC there).
- Pin 6 (shutdown) → R2 100 kΩ to GND, so the LNA is always on.
- Pins 3, 4, 5, 8 are NC: tie their pads to ground. Paddle = GND.

Power: the 5 V rail from the Power sheet → FB1 (600 Ω @ 100 MHz, ≥ 1 A) →
C7 10 µF + C8 100 nF → +5V_LNA → R3. J3 (now the data USB-C USB4105-GF-A), R5/R6
and the power LED D1 + R7 moved to the Power sheet.

## Layout rules

- Stackup: L1 RF and parts; L2 solid ground with nothing routed on it; L3 and L4
  ground pours (route the 5 V rail on L3 or L4 if it can't stay on top).
- RF traces: 50 Ω grounded coplanar waveguide on L1 over L2. Width from JLCPCB's
  impedance calculator (expect ≈ 0.3–0.35 mm) with a 0.2 mm gap. Make it an `RF`
  net class.
- Keep the RF path in one straight line between the SMA jacks: short, no vias,
  45° bends or arcs only.
- Ground-via fence along both sides of every RF trace and around the board edge,
  every 2–3 mm, close to the gap.
- U1: three vias under the paddle in Qorvo's pattern (0.35 mm drill); every ground
  pad straight to a via.
- SAW filters: a via right at each ground pad.
- L2 and C3 right at pin 7; C4 and C5 within a few mm.
- SMA edge launch: keep the centre-pin pad short and taper into the trace. If S11
  is poor later, a ground cutout under the pad on L2 is the usual fix.
- Antenna SMA on one short edge; J2 on a long edge beside SAW2. USB-C, LED and the
  controller sit at the far end of the board. Shield-can ground ring around this section.
- Silkscreen: "ANT IN", "J2 TEST OUT", "C6: A = J2, B = tuner", board name, rev, date.

## Bring-up

Write each prediction in `../design-log.md` before measuring.

1. Visual check; meter VDD to GND for shorts.
2. Power from USB-C through a USB meter on a board with only the front end and power
   fitted (C6 on pad A): expect ≈ 65–75 mA with the LED.
3. NanoVNA S21 at J1 → J2, 600–1200 MHz: ≈ 21 dB at 915 MHz; cell band ≥ 40 dB below in-band.
4. S11 and S22 ≥ 10 dB in band; if not, try the shunt pads.
5. Stability: 50 Ω on the input, output to the tinySA in Ultra mode, 0–6 GHz:
   nothing that moves when you touch the board.
6. System: antenna → front end → J2 → RTL-SDR. Compare the noise floor in SDR++ and the
   rtl_433 / rtlamr decode counts over 15 minutes against the bare-RTL-SDR baseline.
   Turn the RTL-SDR gain down to avoid overload with 21 dB in front of it.
