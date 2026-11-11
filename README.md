# ISM-Band SDR Board

I'm building a receive-only software defined radio for the US 902–928 MHz ISM band,
on a single 4-layer board designed in Altium. A SAW-filtered low-noise front end feeds an
ADL5380 I/Q demodulator driven by an ADF4351 LO, and the I and Q outputs go to a
192 kS/s USB audio interface for processing on the PC. An STM32F072 tunes the LO
over USB.

The full write-up (components, block diagram, and the signal chain with gain and
noise figure) is in [README.pdf](README.pdf).

Schematic and PCB files can be found in [PCB_Directory](ISM_SDR_PC)
