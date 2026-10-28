# Hardware

| Path | Contents |
|---|---|
| [`01-frontend/board1-frontend-spec.md`](01-frontend/board1-frontend-spec.md) | FrontEnd sheet: SAW → LNA → SAW, targets, layout rules, bring-up |
| [`02-tuner/tuner-spec.md`](02-tuner/tuner-spec.md) | Tuner sheet: ADL5380, ADF4351 LO, baseband, STM32 |
| [`power/power-spec.md`](power/power-spec.md) | Power sheet: USB-C input, rail tree, bead and decoupling rules |
| [`design-log.md`](design-log.md) | Decisions and predicted vs measured numbers |
| [`datasheets/`](datasheets/) | Datasheets and reference circuits |

The Altium project (`.PrjPcb`, `.SchDoc`, `.PcbDoc`) lives alongside these specs once
it is committed. Generated output folders are ignored by `.gitignore`.
