# Rev 3 Changelist - OpenFC-Lite

Running list of changes for the next hardware revision, collected during Rev 2
bring-up and flight testing. Schematic/PCB edits are made by the maintainer. Items are
appended as found; nothing here is implemented yet.

Rev 2 bring-up context: OpenDrone-Testing bring-up logs (2026-08-05 onward).

## Confirmed defects to fix

1. **GND shorts on GPIO0-3, GPIO28, GPIO31** (Rev 2, KiCad-induced shorted
   traces on top + bottom copper). Kills stock UART0/VTX connector, GPS pads,
   M1/M4 motor pads; Rev 2 boards fly with the rework pin mapping baked into
   the Betaflight target (`OPENFC_LITE_RP2350B/config.h`). Root-cause the KiCad
   artifact in the Rev 2 layout and add DRC coverage that would have caught it.
   Fixing this restores UART0, GPS pads, M1/M4, UART1, I2C0 to their intended
   pins.
2. **Gyro LED mounted 180 degrees wrong** (reversed polarity): does not light.
   No functional impact otherwise. While fixing: verify orientation of all
   power-line indicator LEDs against the footprint/courtyard markings; check
   whether the silk/footprint invited the wrong orientation.

## Decisions needed

3. **Production IMU**: Rev 2 boards are assembled with BMI270 while KiCad/BOM
   still says LSM6DSV16X; footprint also accepts ICM-426xx/456xx. Decide after
   flight testing, then align schematic, BOM, and Betaflight target defaults.
