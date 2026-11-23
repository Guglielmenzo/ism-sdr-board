# ISM-Band SDR Board

A file and prject log for a 915 MHz Software Defined Radio, designed in Altium,
to be produced on a 4-layer board. The hardware chain is roughly as follows:
SAW Filter -> LNA -> Saw Filter 2 -> Balun -> IQ Demod -> Pre-ADC filtering -> ADC -> STM32 processer.
The board's power is all from a USBC input with various dropout regulators and voltage dividers.
The clock signal is produced by a crystal oscillator fed into a chip integrated PLL, driven by the same STM32.

The full write-up (components, block diagram, and the signal chain) is in [README.pdf](README.pdf).

Schematic and PCB files can be found in [ISM_SDR_PCB](ISM_SDR_PCB)

Project revisions and old schematic analysis & notes can be found in [PCB_History](Documentation/PCB_History)
