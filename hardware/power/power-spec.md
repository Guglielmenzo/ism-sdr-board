# Power — Power sheet (spec)

Everything starts at J3's USB 5 V. About 0.5 A total, so plug into a USB-C or USB 3
port on the PC (USB 2.0 only promises 500 mA). Ferrite beads filter noise only; they
don't change the DC voltage. The 3.3 V rails come from LDOs, never resistors.

```
J3 USB4105-GF-A (USB-C with USB 2.0 data); R5, R6 5.1 kΩ on CC1/CC2; U302 USBLC6-2SC6 on D+/D−
VBUS 5 V
├─ D1 LED + R7 1 kΩ (power indicator, before any bead)
├─ FB1   → +5V_LNA   QPL9547 (~65 mA)                         [FrontEnd sheet]
├─ FB301 → +5V_MIX   ADL5380 VCC1–3 and ADJ (~245 mA)          [Tuner sheet]
├─ FB302 → +5V_BB    op-amps, 2.5 V mid-rail reference
├─ FB303 → U303 LDO → +3V3_DIG   STM32F072
└─ FB304 → U203 low-noise LDO → +3V3_LO   ADF4351 + TCXO
```

| IC | Voltage | From |
|---|---|---|
| QPL9547 | 5 V | FB1 |
| ADL5380 | 5 V | FB301 |
| ADA4807-2 ×2 | 5 V | FB302 |
| USBLC6 | 5 V | VBUS directly |
| U303, U203 LDOs | 5 V in, 3.3 V out | FB303, FB304 |
| STM32F072 | 3.3 V | U303 |
| ADF4351, TCXO | 3.3 V | U203 |

- Beads: one part everywhere, 0603, ≈ 600 Ω @ 100 MHz, ≥ 1 A, DCR ≤ 0.15 Ω
  (a JLCPCB Basic part). The 1 A rating matters on FB301: bead impedance falls as DC
  current nears the rating.
- After each bead: 10 µF X5R (≥ 10 V) + 100 nF. At each IC pin: its own small caps.
  Both are needed: the pair after the bead makes the low-pass filter; the pin caps
  supply fast current within 1–2 mm of the pin.
- Each rail gets its own power-port name in Altium so the branches stay separate.
- Layout: USB-C, LED and U303 at the far end from the antenna.
