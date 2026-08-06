# RP2350B Two-Sided Layout Rules

Source: RP2350 Datasheet §6.3.8.1, Hardware Design with RP2350 guide (Raspberry Pi).

Scope: placement of the RP2354B internal core regulator and the top/bottom split for
OpenFC-Lite. The general layout rules (buck hot loops, crystal, USB routing, motor
outputs, ESD, GND plane) are in [EMC_CHECKLIST.md](EMC_CHECKLIST.md) and are not
repeated here.

## Hard constraint: core regulator must stay on MCU side

Direct quote from RP2350 Datasheet §6.3.8.1:

> "Don't place any of C_IN / L_X / C_OUT on the opposite side of the PCB."
> "Follow this layout as closely as possibly."

This means the following MUST stay on the top side with U2:

| Ref | Part | Role |
|---|---|---|
| C19 | 4.7 uF on +3.3V | C_IN, part of the high-current hot loop |
| L1 | AOTA-B201610S3R3-101-T, 3.3 uH, 0806 | L_X, VREG_LX switch-node path |
| C20 | 4.7 uF on +1.1V | C_OUT, part of the hot loop |
| R12 + C21 | 30R + 4.7 uF | VREG_AVDD filter, needs its own GND via back to the QFN pad |
| C5 | 100 nF on +1.1V | core decoupling on the regulator side, part of the C_OUT loop |

Footprint zone: ~6x4 mm cluster near pins 60-65.

Additional rules for this zone:
- Cut away copper immediately under L1 and the VREG_LX trace, on the top layer AND on inner layer 2 (6-layer board)
- GND vias back to the QFN centre pad: use **2 adjacent vias** to reduce impedance, "short-as-possible"
- The VREG_AVDD filter GND path MUST NOT share vias with the C_IN/C_OUT high-current GND
- VREG_FB: feed from the regulator output, do NOT route under L_X

## Can go on bottom side

| Component | Rule |
|---|---|
| Core decoupling away from the regulator (C1, 100 nF, opposite QFN edge) | 2 GND vias per cap |
| 2nd 4.7 uF on +1.1V (C2, opposite edge) | Explicitly recommended by RPi to be AWAY from the regulator |
| IOVDD, QSPI_IOVDD (pin 69), USB_OTP_VDD (pin 5), ADC_AVDD 100 nF caps | 2 GND vias per cap, directly under the pin |
| Crystal X1 (12 MHz) + load caps C4, C8 | Risky but doable, see the parasitic budget below |
| USB-C connector USB1 | OK once D+/D- has passed R10/R11 |

## Crystal on bottom: parasitic budget

- RP2350 uses 10.5 pF total load, ~3 pF parasitic budget
- Each via adds 0.3-0.5 pF, so 2 vias consume ~1 pF of that budget
- Place the crystal directly under XIN/XOUT (pins 30/31) and minimise stubs
- Retuning knobs if the oscillator misbehaves: R5 (1k damping) and the load caps C4/C8 (20 pF fitted)
- Crystal guard ring and flood rules: EMC_CHECKLIST.md section 6

## USB: near-MCU placement

- R10/R11 (30R series termination) stay on the **top side** immediately adjacent to pins 66/67
- The differential pair may only transition to the bottom layer AFTER those resistors
- Return path, impedance and stitching rules: EMC_CHECKLIST.md section 4

## Summary strategy for OpenFC-Lite

Top side:
- RP2354B (U2)
- Core regulator cluster: C19, L1, C20, R12, C21, C5 (~6x4 mm)
- USB series resistors R10, R11

Bottom side:
- Remaining +1.1V decoupling: C1 100 nF and C2 4.7 uF on the opposite QFN edge
- All IOVDD / QSPI_IOVDD / USB_OTP_VDD / ADC_AVDD 100 nF caps
- Crystal X1 + load caps, directly under pins 30/31
- USB-C connector
- All other support circuits: LDOs, power mux, IMU, OSD, SD card, connectors

## Sources

- RP2350 Datasheet: https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf
- Hardware Design with RP2350: https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf
- Reference RP2350B FC designs: madflight FC3v2, TichyTech rp2350-flight-controller
