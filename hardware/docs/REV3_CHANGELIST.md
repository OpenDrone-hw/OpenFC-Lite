# Rev 3 Changelist - OpenFC-Lite

Running list of changes for the next hardware revision, collected during Rev 2
bring-up and flight testing. Schematic/PCB edits are made by the maintainer.

Rev 2 bring-up context: OpenDrone-Testing bring-up logs (2026-08-05 onward).

## Implemented

1. **GND shorts on GPIO0-3, GPIO28, GPIO31** (Rev 2, KiCad-induced shorted
   traces on top + bottom copper) are fixed. All six pins now carry distinct
   nets (GPIO0/1 UART0 TX/RX, GPIO2/3 PIO UART0 TX/RX, GPIO28 M4, GPIO31 M1),
   DRC reports zero unconnected items and schematic parity is clean. This
   restores UART0, GPS pads, M1/M4, UART1 and I2C0 to their intended pins, so
   the Rev 2 rework pin mapping in the Betaflight target can be retired.
2. **Production IMU decided: BMI270** (U9, C2836813). Schematic, BOM and board
   agree. Betaflight target defaults follow.
3. **1.8 V gyro LDO resolved to LP5912-1.8DRVR** (U15, WSON-6, C2876234).
   The NCV8187AMT180TAG is gone from both the schematic and the board.
4. **3S-8S input.** Both bucks moved from LMR51430YFDDCR (36 V rated, 38 V
   absolute maximum) to **LMR51635YDDCR** (4.3-60 V, 63 V absolute maximum,
   3.5 A, 1.1 MHz PFM, SOT-23-THN, C45262770). The reference changes from
   0.6 V to 0.8 V, so the feedback dividers changed with it: R22 6.49k to
   8.87k (10 V rail, 9.82 V) and R24 13.7k to 19.1k (5 V rail, 4.99 V). The
   package footprint is unchanged but the pinout is not, so both buck corners
   were re-routed. Refdes moved to U6 and U16.

## Open

5. **Gyro LED (D9) does not light.** Previously recorded as a 180 degree
   mounting error. That diagnosis does not hold: D9 uses the stock `Device:LED`
   symbol and `LED_SMD:LED_0402_1005Metric` footprint with pin 1 (cathode) to
   GND, identical to D4, D5 and D7, all of which work.

   D9 is not a rail indicator. Its anode sits on the LDO's open-drain power
   good output (U15 pin 3) with R39 6.49k pulling that node up to +5 V, giving
   roughly 0.32 mA against 0.78 to 0.95 mA for the other three indicators.

   Measure the PG node on a Rev 2 board before changing anything:
   - near 5 V, the LED is forward biased and simply too dim; drop R39 to about
     2.4k to match D5
   - near 0 V, PG is asserting a fault, or Rev 2's NCV8187 (PG on a different
     pin number depending on package variant) never drove that node, which
     would explain a dead LED with nothing actually mounted backwards

   While fixing: verify the orientation of all power indicator LEDs against
   footprint and courtyard markings.

6. **Solder pads J45 and J46** exist in the schematic but are not placed on the
   PCB.

7. **Input capacitors stay at 4.7 uF 50 V** (C22, C23) for 8S. Rated voltage is
   adequate at 33.6 V and the pair clears the 2.2 uF minimum input capacitance
   even at worst case DC bias derating, but there is no margin for hot plug
   ringing above 50 V. Revisit if field returns show input cap failures.

8. **Inductor headroom.** L2 and L3 stay at 4.7 uH. Ripple at 33.6 V input is
   0.82 A on the 5 V rail and 1.37 A on the 10 V rail. Confirm the
   XRTC303020D4R7MBCA saturation current against the buck's 3.5 A limit before
   quoting a rail current anywhere.

9. **VBAT sense divider unchanged** at R1 100k / R2 10k. At 8S this reads
   3.06 V into the 3.3 V ADC, 3.16 V on a 4.35 V per cell pack. It does not
   clip and the 100k top resistor limits any clamp current to a few hundred
   microamps, but there is little headroom if the 3.3 V rail droops. Changing
   R2 to 6.49k would move it to 2.05 V and needs a `vbat_scale` update.
