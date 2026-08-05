# OpenFC-Lite Design Notes

Detailed design description of the OpenFC-Lite. Values are extracted from the KiCad design files: `hardware/OpenFC.kicad_sch` (root plus six sub-sheets) and `hardware/OpenFC.kicad_pcb`.

## Architecture

Single RP2354B does everything: flight control, USB, DShot, software UARTs, LED strip, and analog OSD pixel timing, all through PIO. The design deliberately omits a barometer, an integrated receiver, and onboard blackbox flash: logging goes to a microSD card, the receiver connects externally over UART. The schematic is hierarchical:

| Sheet | Contents |
|---|---|
| `rp2350a.kicad_sch` | RP2354B (U2), USB-C (USB1), 12 MHz crystal (X1), boot button (U1), buzzer and LED-strip FETs (Q1, Q2) |
| `power.kicad_sch` | 10 V and 5 V bucks, USB/BATT power mux, 1.8 V gyro LDO, 3.3 V LDO |
| `imu.kicad_sch` | IMU (U9) on SPI0 |
| `osd.kicad_sch` | Analog OSD chain (U10-U12) |
| `blackbox.kicad_sch` | microSD slot (Card1) on SPI1 |
| `pads.kicad_sch` | JST SH connectors and through-hole solder pads |

## Key parts

| Function | Ref | Part | LCSC |
|---|---|---|---|
| MCU | U2 | RP2354B (QFN-80, 2 MB stacked flash) | C39843328 |
| IMU (schematic) | U9 | LSM6DSV16XTR (LGA-14) | C5267406 |
| 10 V buck (switchable) | U3 | LMR51430YFDDCR | C5219261 |
| 5 V buck (always-on) | U4 | LMR51430YFDDCR | C5219261 |
| Buck inductors | L2, L3 | XRTC303020D4R7MBCA 4.7 uH | C39846837 |
| 5 V power mux | U5 | TPS2116DRLR | C3235557 |
| 3.3 V LDO | U7 | LP5912-3.3DRVR | C524780 |
| 1.8 V gyro LDO | U15 | LP5912-1.8DRVR | C2876234 |
| OSD SPDT switch | U10 | SN74LVC1G3157DTBR | C2673087 |
| OSD op-amp | U11 | COS8051SOT | C7463385 |
| OSD comparator | U12 | TLV7031DPWR | C2876045 |
| microSD slot | Card1 | TF PUSH (push-push) | C393941 |
| Crystal | X1 | XTM250 12 MHz | - |
| Buzzer / LED-strip FETs | Q1, Q2 | DMN1150UFB-7B | C2849580 |

## IMU

The IMU site is an LGA-14 footprint on SPI0 with a dedicated 1.8 V analog supply, wired so that pin-compatible parts from several vendors fit (universal IMU wiring per the schematic note: BMI270, ST LSM6 family, TDK). A CLKIN line is routed but unused by the LSM6DSV16XTR; it exists for pin-compatible IMUs that support it. The schematic and BOM carry **LSM6DSV16XTR** (U9); **Rev 2 boards are assembled with BMI270**. The production IMU choice is an open Rev 3 decision, see [REV3_CHANGELIST.md](REV3_CHANGELIST.md).

## Power tree

| Rail | Source | Regulator | Notes |
|---|---|---|---|
| +10V (switchable) | +BATT | LMR51430YFDDCR buck (U3), L2 4.7 uH | EN gated by the MCU (10V_ENABLE net). Feeds the VTX connector. |
| +5V_BUCK (always-on) | +BATT | LMR51430YFDDCR buck (U4), L3 4.7 uH | Battery-side 5 V source. |
| +5V | +5V_BUCK / +5V_USB | TPS2116DRLR mux (U5) | Auto-selects battery vs USB source. |
| +3.3V | +5V | LP5912-3.3DRVR (U7) | MCU IOVDD and VREG input, IMU I/O, microSD, OSD chain. |
| +1.8V_GYRO | +5V | LP5912-1.8DRVR (U15) | Dedicated IMU analog supply. |
| +1.1V core | +3.3V | RP2354B internal switching regulator, L1 | MCU core. |

## Connectivity and I/O

**Serial**
- UART0, UART1: hardware UARTs
- PIO UART0, PIO UART1: software UARTs via PIO

**Buses**
- SPI0: IMU (plus interrupt line)
- SPI1: microSD blackbox
- I2C0: SDA/SCL on the 6-pin peripheral connector and pads

**Motor / actuator**
- M1-M4: signal-level DShot to the external ESC
- LED_STRIP: addressable LED output, FET-buffered (Q2)
- BUZZER-: N-FET low-side buzzer output (Q1)

**Analog inputs (each via RC filter)**
- ADC_VBAT: battery voltage sense (resistive divider)
- CURRENT: current sense from the ESC connector
- RSSI: analog RSSI
- ADC2: spare ADC pad

**OSD**
- OSD_SYNC, OSD_EN, OSD_W, OSD_LVL: composite-video sync/overlay signals between the MCU and the OSD front-end

**Status / control**
- LED0: status LED
- 10V_ENABLE: switches the 10 V VTX/camera rail

A large number of through-hole solder pads (J*) expose rails and signal lines (5 V, GND, M1-M4, UARTs, LED_STRIP, RSSI, etc.) for direct wiring.

## Analog OSD

There is no MAX7456-class OSD chip. The OSD is a discrete chain driven by RP2354B PIO: a TLV7031 comparator (U12) separates sync from the camera video, a COS8051 op-amp (U11) buffers the video path, and an SN74LVC1G3157 SPDT analog switch (U10) overlays the MCU-generated pixels. Camera video enters on the 3-pin connector (U13), overlaid video leaves toward the VTX. The board carries OSD debug/bring-up pads that the Mini omits.

## Connectors

Pin-to-net mapping extracted from the schematic. All JST SH are SMD.

| Ref | Part | Function |
|---|---|---|
| P1 | SM08B-SRSS-TB, 8-pin SMD JST SH | ESC harness: +BATT, GND, CURRENT, PIO UART1 RX (ESC telemetry), M1-M4 |
| U8 | SM06B-SRSS-TB, 6-pin SMD JST SH | VTX: +10V, GND, UART0 TX, UART0 RX, GND, UART1 RX |
| U14 | SM06B-SRSS-TB, 6-pin SMD JST SH | Peripheral: +5V, GND, PIO UART1 RX/TX, I2C0 SDA/SCL |
| CN1 | SM04B-SRSS-TB, 4-pin SMD JST SH | Receiver: +5V, GND, PIO UART0 RX/TX |
| U13 | SM03B-SRSS-TB, 3-pin SMD JST SH | Camera: +5V, GND, video in |
| USB1 | Type-C 16P | Configuration and flashing |

## Blackbox

microSD push-push slot (Card1, LCSC C393941) on SPI1. No onboard SPI flash.

## Firmware

Target firmware is **Betaflight** on the RP2350 (PICO) platform; the RP2354B uses the Raspberry Pi Pico SDK (C/C++). PIO blocks drive the DShot motor outputs, the software UARTs, the LED strip, and the analog-OSD pixel timing. The RP2350 framebuffer OSD driver (FB_OSD) is merged upstream: [betaflight/betaflight#14882](https://github.com/betaflight/betaflight/pull/14882), merged 2026-04-22.

Rev 2 boards fly with a rework pin mapping baked into the Betaflight target (`OPENFC_LITE_RP2350B`), compensating for the Rev 2 GND-short defect described in [REV3_CHANGELIST.md](REV3_CHANGELIST.md).

## Variants and revisions

A smaller sibling, [OpenFC-Lite-Mini](https://github.com/incutec-hw/OpenFC-Lite-Mini) (20 x 20 mm, RP2354A), shares this design; the two differ in MCU package, GPIO count, and some I/O. This full-size board adds bigger pads, more I/O, and OSD debug pads. Fabrication sets are generated per revision into `hardware/production/` (gitignored) with the Fabrication Toolkit from the panel project (`hardware/OpenFC_Panel.kicad_pro`); the revision history is in [CHANGELOG.md](../../CHANGELOG.md) and the staged Rev 3 changes in [REV3_CHANGELIST.md](REV3_CHANGELIST.md).
