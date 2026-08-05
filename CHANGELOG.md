# Changelog

Newest first.

- **Rev3** (staged, not implemented): change list collected during Rev 2 bring-up, see [hardware/docs/REV3_CHANGELIST.md](hardware/docs/REV3_CHANGELIST.md). Headline items: fix KiCad-induced GND shorts on GPIO0-3/28/31, fix reversed gyro LED, decide the production IMU.
- **Rev2** (2026-08, current): flown. Boards fly with a rework pin mapping baked into the Betaflight target (`OPENFC_LITE_RP2350B`) that works around GND shorts on GPIO0-3, GPIO28, and GPIO31. Assembled with BMI270 while the schematic still carries LSM6DSV16XTR.
- **Rev1**: first prototype, bench-tested.
