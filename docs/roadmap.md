# Roadmap

Each step ends with something measured. Write the prediction in
[`hardware/design-log.md`](../hardware/design-log.md) before measuring.

1. **Front-end bring-up.** A board with only the front end and power fitted, C6 on
   pad A. NanoVNA S21 and S11 at J1 → J2, then the board in front of the RTL-SDR.
   *Done when:* about 21 dB of gain, at least 40 dB of rejection at 869–894 MHz, and
   rtl_433 and rtlamr decode more than the bare RTL-SDR baseline.
2. **Controller and LO first light.** A fully built board with R203 lifted and J201
   fitted. *Done when:* the PC tunes 902–928 MHz over USB, the LO locks, and the tone
   lands within 2 ppm.
3. **Tuner.** R203 fitted, C6 on pad B. *Done when:* a 915 MHz tone gives I and Q 90°
   apart on a scope, with the image at least 35 dB down in SDR++.
4. **SDR v1.** Board plus USB audio interface. *Done when:* it decodes packets from a
   915 MHz LoRa or Meshtastic board with gr-lora_sdr, and no controller spurs show
   above the noise floor with the antenna port terminated.
5. **Rev B.** On-board fast digitizer (AD9238 dual 12-bit ADC, drivers, low-jitter
   clock, iCE40, FT232H) and shield cans where rev A measurements ask for them.
   *Done when:* 2 MHz or more streams to the PC with no dropped samples, and it
   equals or beats the RTL-SDR at 915 MHz on sensitivity and cell-band rejection.

Out of scope for v1: transmit, 2.4 GHz, FPGA processing.
