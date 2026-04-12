# InkTime

An affordable, open-source smartwatch powered by the Nordic nRF52840 SoC and an e-paper display. Designed to be hackable, low-power, and manufacturable.

## Table of Contents

- [Block Diagram](#block-diagram)
- [Bill of Materials](#bill-of-materials)
- [Hardware Overview](#hardware-overview)
- [nRF52840 Pin Map](#nrf52840-pin-map)
- [Power Consumption](#power-consumption)
- [Test Points](#test-points)
- [Design Log](#design-log)

## Block Diagram

```
                    ╔══════════════════════════════════════════════╗
                    ║             nRF52840  (U$1)                  ║
                    ║        ARM Cortex-M4F @ 64 MHz               ║
                    ║         1 MB Flash · 256 KB RAM              ║
                    ║         BLE 5.0 · USB 2.0 FS                 ║
                    ║                                              ║
   ┌─────────────┐  ║  SPI bus: P0.02 SCK  P0.03 MOSI  P0.05 CS    ║
   │  E-Paper    │──╫──  GPIO: P0.15 DC  P0.16 RST  P0.17 BUSY     ║
   │  Display    │  ║  P1.01 EPD_PWR controls Q1 (P-MOSFET)        ║
   │  24-pin FPC │  ║                                              ║
   └─────────────┘  ║  I2C control bus: P0.06 SDA / P0.07 SCL      ║
                    ║    IC1 BQ25180 (charger config + status)     ║
   ┌─────────────┐  ║    U1  MAX17048 (fuel gauge)                 ║
   │  LiPo Cell  │──╫──    IC2 BMA423   (accelerometer)            ║
   │   1-cell    │  ║    IC3 DRV2605   (haptic driver)             ║
   └─────────────┘  ║    IC9 RT6160A   (buck-boost control)        ║
                    ║  Power path: LiPo to BQ25180 to RT6160A 3V3  ║
   ┌─────────────┐  ║                                              ║
   │  USB-C (J3) │──╫──  VBUS to BQ25180 charging path             ║
   └──────┬──────┘  ║  D+ / D- to nRF USB peripheral               ║
          │         ║                                              ║
   ┌──────┴──────┐  ║  USB ESD (D1) on D+ / D- lines               ║
   │ USBLC6-2SC6Y│  ║  GPIO lines: P0.08 IMU_INT1  P1.08 IMU_INT2  ║
   │ ESD (D1)    │  ║  P0.11 PMIC_INT  P0.12 HAPTIC_EN             ║
   └─────────────┘  ║  P0.13 SW_UP  P0.14 SW_ENT  P1.02 SW_DN      ║
   ┌─────────────┐  ║                                              ║
   │ Buttons     │──╫──  SW_UP / SW_ENT / SW_DN (EVP-AKE31A)       ║
   │ (×3 SMD)    │  ║                                              ║
   └─────────────┘  ║  32 MHz XTAL (X1) ── HFXO / RF clock         ║
                    ║  32.768 kHz XTAL (X2) ── LFXO / RTC          ║
   ┌─────────────┐  ║  ANT: L3 matching ── 2450AT18B100E           ║
   │ SWD Debug   │──╫──  SWDIO / SWDCLK / SWO / RESET              ║
   │ TC2030 (J1) │  ║      (also exposed on test pads)             ║
   └─────────────┘  ╚══════════════════════════════════════════════╝
```

## Bill of Materials

| Ref | Qty | Part Number | Description | Package | Where to buy | Datasheet |
|-----|-----|-------------|-------------|---------|--------------|-----------|
| U$1 | 1 | nRF52840-QIAA | Main SoC, BLE 5.0, Cortex-M4F | AQFN-73 7×7 mm | [JLCPCB C190794](https://jlcpcb.com/partdetail/NordicSemiconductor-nRF52840_QIAA/C190794) | [PDF](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.5.pdf) |
| IC1 | 1 | BQ25180YBGR | Single-cell LiPo charger, I2C, up to 1 A | 8-DSBGA 1.6×1.1 mm | [JLCPCB](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C2678360) | [PDF](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| IC2 | 1 | BMA423 | 3-axis accelerometer + step engine, I2C | LGA-12 2×2 mm | [JLCPCB](https://jlcpcb.com/partdetail/BoschSensortec-BMA423/C379441) | [PDF](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma423-ds000.pdf) |
| IC3 | 1 | DRV2605YZFR | Haptic actuator driver ERM/LRA, I2C | 9-BGA 1.44×1.44 mm | [JLCPCB](https://jlcpcb.com/partdetail/TexasInstruments-DRV2605YZFR/C89025) | [PDF](https://www.ti.com/lit/ds/symlink/drv2605.pdf) |
| U1 | 1 | MAX17048G+T10 | 1-cell fuel gauge, ModelGauge, I2C | TDFN-8 2×2 mm | [JLCPCB](https://jlcpcb.com/partdetail/MaximIntegrated-MAX17048GT10/C112139) | [PDF](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf) |
| IC9 | 1 | RT6160AWSC | I2C buck-boost DC/DC, 3.3 V output | 15-WL-CSP 1.4×2.3 mm | [JLCPCB](https://jlcpcb.com/partdetail/Richtek-RT6160AWSC/C389054) | [PDF](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-00.pdf) |
| ANT1 | 1 | 2450AT18B100E | 2.45 GHz chip antenna | 3216 SMD | [JLCPCB](https://jlcpcb.com/partdetail/JohansonTechnology-2450AT18B100E/C89939) | [PDF](https://www.johansontechnology.com/datasheets/2450AT18B100E.pdf) |
| J2 | 1 | Molex 503480-2400 | 0.5 mm pitch FPC connector 24-pin | SMD RA | [JLCPCB](https://jlcpcb.com/partdetail/Molex-5034802400/C393913) | [PDF](https://www.molex.com/en-us/products/part-detail/5034802400) |
| J3 | 1 | KH-TYPE-C-16P | USB-C receptacle 16-pin | SMD | [JLCPCB C2765186](https://jlcpcb.com/partdetail/Kinghelm-KH_TYPE_C_16P/C2765186) | [PDF](https://www.kinghelm.net/product/kh-type-c-16p.html) |
| J1 | 1 | TC2030-IDC | Tag-Connect 6-pin SWD footprint | PTH (no connector) | [Tag-Connect](https://www.tag-connect.com/product/tc2030-idc) | [PDF](https://www.tag-connect.com/wp-content/uploads/bsk-pdf-manager/TC2030-IDC_1.pdf) |
| X1 | 1 | 32 MHz crystal | RF / HFXO clock | 2016 SMD | JLCPCB basic parts | Nordic ref. design |
| X2 | 1 | 32.768 kHz crystal | RTC / LFXO clock | 3215 SMD | JLCPCB basic parts | Nordic ref. design |
| L1 | 1 | Wurth 744043680 | 68 µH EPD boost inductor | 4828 SMD | [JLCPCB C408334](https://jlcpcb.com/partdetail/WurthElektronik-744043680/C408334) | [PDF](https://www.we-online.com/catalog/datasheet/744043680.pdf) |
| L2 | 1 | 10 µH inductor | nRF DCC pin choke | 0201 | JLCPCB basic parts | — |
| L3 | 1 | 15 nH inductor | RF antenna matching network | 0201 | JLCPCB basic parts | — |
| L7 | 1 | FTC252012SR47MBCA | 0.47 µH buck-boost inductor | 2016 SMD | [JLCPCB C5832368](https://jlcpcb.com/partdetail/6763488-FTC252012SR47MBCA/C5832368) | [PDF](https://product.tdk.com/en/search/inductor/inductor/smd/info?part_no=FTC252012SR47MBCA) |
| D1 | 1 | USBLC6-2SC6Y | USB ESD TVS protection | SOT-23-6 | JLCPCB basic parts | [PDF](https://www.st.com/resource/en/datasheet/usblc6-2sc6y.pdf) |
| D2, D4, D5 | 3 | MBR0530 | 30 V / 0.5 A Schottky, EPD gate circuit | SOD-123 | JLCPCB basic parts | [PDF](https://www.onsemi.com/pdf/datasheet/mbr0530t1-d.pdf) |
| Q1 | 1 | P-ch MOSFET | EPD 3.3 V power rail switch | SOT-23 | JLCPCB basic parts | — |
| Q3 | 1 | SI1308EDL-T1-GE3 | N-ch MOSFET 30 V / 1.5 A, EPD gate driver | SC-70 | JLCPCB basic parts | [PDF](https://www.vishay.com/docs/63401/si1308edl.pdf) |
| SW_UP/ENT/DN | 3 | EVP-AKE31A | SMD tactile push button | SMD | JLCPCB basic parts | [PDF](https://industrial.panasonic.com/ww/products/pt/light-touch-switches/models/EVPAKE31A) |
| R1–R18 | ~20 | Resistors | 0.47 Ω, 2.2 Ω, 3.3 kΩ, 5.1 kΩ, 7.68 kΩ, 10 kΩ | 0201 | JLCPCB basic parts | — |
| C1–C43, EPD_Cx | ~60 | Capacitors | 1 pF – 22 µF decoupling / filtering | 0201 (≤100 nF), 0402 (>100 nF) | JLCPCB basic parts | — |
| TP_* | 14 | Test pads | 3V3, VBAT, GND, SDA, SCL, SWDIO, SWDCLK, SWO, RESET… | TP20R | — | — |
| SJ1 | 1 | Solder jumper | Configuration / address selection | — | — | — |

## Hardware Overview

### nRF52840 — Main Microcontroller

The nRF52840 is the heart of InkTime. It is an ARM Cortex-M4F SoC running at 64 MHz, with 1 MB Flash, 256 KB RAM, an integrated Bluetooth 5.0 + 802.15.4 radio, hardware cryptographic accelerators, and a native USB 2.0 Full-Speed controller.

Two external crystals are mandatory:
- **X1 — 32 MHz:** feeds the high-frequency oscillator (HFXO) used by the RF transceiver.
- **X2 — 32.768 kHz:** clocks the low-frequency oscillator (LFXO) that keeps timekeeping alive during deep sleep.

The DCC pin (internal DC/DC converter) is filtered with a 10 µH inductor (L2) and bypass capacitors following Nordic's reference hardware guidelines.

### E-Paper Display — SPI interface

The display panel connects through a 24-pin, 0.5 mm pitch right-angle FPC connector (J2, Molex 503480-2400). The communication protocol is **4-wire SPI**, with three additional GPIO control lines: DC (data/command select), RST (hardware reset), and BUSY (refresh-in-progress flag).

A dedicated power rail (EPD_3V3) is produced by a boost converter using L1 (68 µH, Wurth 744043680). The rail is enabled and disabled by Q1 (P-channel MOSFET) driven from nRF pin P1.01. Gate voltages for the e-paper panel (positive and negative swing) are generated by Q3 (SI1308EDL N-MOSFET) together with diodes D2, D4, D5 (MBR0530 Schottky). Twelve 1 µF / 50 V capacitors (EPD_C1–EPD_C12, 0402) buffer the gate supply transients. Panel type is selected by a 2.2 Ω resistor on J2 pin 6.

### Battery Charger — BQ25180YBGR (IC1)

The BQ25180 is a single-cell Li-Ion/LiPo charger in an ultra-compact 8-pin DSBGA package. It is configured and monitored entirely via **I2C**. The charge current is set by R1 (10 kΩ on the TS/MR pin). The SYS output (VREG net) powers the buck-boost regulator IC9 and keeps the system alive even when the battery is absent or deeply discharged. An interrupt line (PMIC_INT → P0.11) signals the nRF whenever charging state changes. The battery connects directly to test pads TP_VBAT and TP_BAT_GND — no JST connector is used, saving vertical clearance inside the case.

Decoupling on VREG: C24 (10 µF) in parallel with C23 (100 nF). SYS output: C39 (10 µF).

### Fuel Gauge — MAX17048 (U1)

The MAX17048 estimates remaining battery capacity using the ModelGauge algorithm, which requires no current sense resistor — only a direct connection between the CELL pin and VBAT. It shares the **I2C** bus with the charger, IMU, and haptic driver, and reports state-of-charge as a percentage over I2C.

### Buck-Boost Regulator — RT6160AWSC (IC9)

The RT6160A is a 15-ball WL-CSP I2C-programmable switched-mode regulator that maintains a stable **3.3 V** output across the entire LiPo discharge range (3.0–4.2 V). The switching inductor is L7 (0.47 µH, TDK FTC252012SR47MBCA). Input bulk capacitors: C25 + C33 (22 µF each, 0402). Output capacitors: C20 + C21 (4.7 µF each, 0402). The regulator is enabled by the BQ25180's SYS output and its output voltage is fine-tuned over the shared I2C bus.

### Accelerometer / IMU — BMA423 (IC2)

The Bosch BMA423 is a 12-bit, 3-axis accelerometer in a tiny 2×2 mm LGA-12 package. It contains a built-in step counter, wrist-tilt detection, and activity classification engine — all processed on-chip without burdening the nRF. The interface is **I2C** (CSB pin pulled to VDD). Two hardware interrupt outputs (INT1 → P0.08, INT2 → P1.08) allow the watch to wake from System OFF in response to motion. The I2C address (0x18) is set by pulling SDO to GND through R3. Power decoupling: C37 (1 µF) on VDDIO and C38 (1 µF) on VDD.

### Haptic Driver — DRV2605YZFR (IC3)

The DRV2605 can drive both ERM (eccentric rotating mass) and LRA (linear resonant actuator) vibration motors using a built-in waveform library of 123 effects. Communication is over **I2C**; the EN pin (HAPTIC_EN → P0.12) is used to hard-disable the driver during sleep to eliminate quiescent current. Motor terminals are accessible at test pads TP_OP and TP_ON. VDD decoupling: C34 (100 nF, 0201).

### USB-C Port — J3 + ESD Protection (D1)

The KH-TYPE-C-16P receptacle provides VBUS for charging and exposes D+ / D− to the nRF's internal USB 2.0 Full-Speed controller. CC1 and CC2 carry 5.1 kΩ pull-down resistors (R1_USB, R2_USB) to advertise a standard 5 V / 900 mA sink. The USBLC6-2SC6Y TVS array (D1, SOT-23-6) clamps ESD transients on D+ and D−. VBUS decoupling: C42 + C43 (1500 pF each).

### SWD Debug Port — TC2030-IDC (J1)

Programming and debugging are done through a Tag-Connect TC2030-IDC footprint (no physical connector soldered, saving board space and height). It exposes SWDIO, SWDCLK, SWO, RESET, VCC, and GND. All signals are mirrored on clearly labelled test pads.

### Push Buttons

Three Panasonic EVP-AKE31A surface-mount tactile switches (SW_UP, SW_ENT, SW_DN) are debounced in hardware: each signal has a 10 kΩ pull-up to 3V3 and a 1 µF capacitor to GND forming an RC low-pass filter.

### 2.4 GHz RF Section

The Johanson 2450AT18B100E chip antenna is mounted at the board edge. The matching network between the nRF ANT pin and the antenna follows the Nordic nRF52840 reference design: a 15 nH series inductor L3, shunt capacitors C3 (1 pF), C4 (1 pF), C9 (820 pF), and C11 (100 pF). There is no copper poured and no signal routed in the area beneath the antenna — the PCB is physically cut out under the antenna footprint.

## nRF52840 Pin Map

| Pin | Signal | Destination | Bus / Function | Reason for choice |
|-----|--------|-------------|----------------|-------------------|
| P0.00 / XL1 | XL1 | X2 pin 2 | LFXO — 32.768 kHz crystal | Dedicated crystal pins, cannot be reassigned |
| P0.01 / XL2 | XL2 | X2 pin 1 | LFXO — 32.768 kHz crystal | Dedicated crystal pins, cannot be reassigned |
| P0.02 | SCK | J2 pin 13 → EPD | SPI clock | Flexible GPIO; SPIM peripheral is fully remappable |
| P0.03 | MOSI | J2 pin 14 → EPD | SPI data out | Flexible GPIO; chosen to keep SPI traces grouped |
| P0.05 | EPD_CS | J2 pin 12 → EPD | SPI chip select | Flexible GPIO; adjacent to SCK/MOSI on port 0 |
| P0.06 | SDA | IC1, IC2, IC3, U1 | I2C data | Close to BQ25180 and BMA423 on PCB → short trace |
| P0.07 | SCL | IC1, IC2, IC3, U1 | I2C clock | Paired with P0.06 for I2C bus |
| P0.08 | IMU_INT1 | BMA423 INT1 | GPIO input (wake) | Free pin; interrupt-capable on port 0 |
| P0.11 | PMIC_INT | BQ25180 !INT | GPIO input | Free pin; interrupt-capable |
| P0.12 | HAPTIC_EN | DRV2605 EN | GPIO output | Free pin; any output-capable GPIO works |
| P0.13 | SW_UP | SW_UP tactile | GPIO input, pull-up | Free pin after peripheral assignment |
| P0.14 | SW_ENT | SW_ENT tactile | GPIO input, pull-up | Free pin after peripheral assignment |
| P0.15 | EPD_DC | J2 pin 11 | GPIO output | Grouped with other EPD pins on port 0 |
| P0.16 | EPD_RST | J2 pin 10 | GPIO output | Grouped with other EPD pins on port 0 |
| P0.17 | EPD_BUSY | J2 pin 9 | GPIO input | Grouped with other EPD pins on port 0 |
| P0.18 / RESET | RESET | J1, TP_RESET | Hardware reset | Dedicated reset pin |
| P1.00 | SWO | J1, TP_SWO | SWD trace output | Port 1 free pin; trace output on any GPIO |
| P1.01 | EPD_PWR | Q1 gate | GPIO output — EPD rail enable | Free port 1 pin; any output GPIO works |
| P1.02 | SW_DN | SW_DN tactile | GPIO input, pull-up | Free pin after peripheral assignment |
| P1.08 | IMU_INT2 | BMA423 INT2 | GPIO input (wake) | Free pin; interrupt-capable on port 1 |
| SWDCLK | SWDCLK | J1, TP_SWDCLK | SWD debug clock | Dedicated SWD pin |
| SWDIO | SWDIO | J1, TP_SWDIO | SWD debug data | Dedicated SWD pin |
| DCC | — | L2 (10 µH) | Internal DC/DC choke | Required by nRF power management |
| XC1/XC2 | — | X1 (32 MHz) | HFXO — RF clock | Dedicated crystal input pins |
| ANT | — | L3 → ANT1 | 2.4 GHz RF output | Dedicated antenna pad |
| VDD / VDDH | 3V3 | RT6160AWSC output | 3.3 V supply | — |
| DEC1–DEC6 | — | 100 nF / 12 pF caps | Internal regulator decoupling | Per Nordic HW guidelines |

## Power Consumption

Estimated figures derived from component datasheets and Nordic power profiler reference data:

| Operating Mode | nRF52840 | E-Paper | BMA423 | DRV2605 | System Total |
|----------------|----------|---------|--------|---------|--------------|
| BLE advertising (1 s interval) | ~15 mA avg | 0 mA | 0.13 mA | 0 mA | **~15 mA** |
| Display refresh | ~3 mA | ~30 mA peak | 0.13 mA | 0 mA | **~33 mA** |
| Haptic event | ~5 mA | 0 mA | 0 mA | ~60 mA | **~65 mA** |
| System OFF (deep sleep) | 2.5 µA | 0 mA | 0 µA | 0 µA | **~3 µA** |

Assuming a 180 mAh LiPo cell, BLE advertising every 1 second, and a display refresh once per minute, the estimated battery life is roughly **5–7 days**.

## Test Points

All test pads are silkscreened with their signal name (signal name only — no component value on silkscreen):

| Pad name | Signal |
|----------|--------|
| TP_3V3 | 3.3 V regulated output (RT6160AWSC) |
| TP_VBAT | Raw battery positive terminal |
| TP_BAT_GND | Battery negative terminal |
| TP_GND | System ground |
| TP_VREG | BQ25180 SYS / VREG output |
| TP_SDA | I2C data line |
| TP_SCL | I2C clock line |
| TP_SWDIO | SWD data |
| TP_SWDCLK | SWD clock |
| TP_SWO | SWD trace / serial wire output |
| TP_RESET | System reset |
| TP_OP / TP_ON | Haptic motor + / − terminals |

## Design Log

A summary of key decisions and accepted rule deviations:

- **PCB thickness: 1 mm.** The standard 1.6 mm board would not fit inside the provided enclosure. All mechanical clearances were verified against the case STEP file.
- **Components on TOP layer only.** Required by the project specification. No components are placed on the BOTTOM layer.
- **Ground planes on both layers.** The TOP and BOTTOM copper pours are both tied to GND. Via stitching is applied across the whole board, with extra density near the RF circuitry.
- **Antenna keepout.** The area under ANT1 (2450AT18B100E) has no copper pour, no signal routing, and a PCB cutout — all three constraints required by the Nordic reference design.
- **Power trace widths.** All power nets (3V3, VBAT, VBUS, VREG) are routed at 0.3 mm. Signal traces use a minimum of 0.15 mm. Traces that pass under BGA pads are narrowed to fit within the available fanout geometry.
- **No right-angle bends.** All trace corners use 45° chamfers throughout the board.
- **Battery direct-soldered.** The LiPo battery is connected directly to TP_VBAT and TP_BAT_GND rather than using the JST connector it ships with. This eliminates ~1 mm of vertical height that would otherwise violate the case Z-clearance.
- **Accepted ERC warning:** "Only INPUT pins on NET ID" is a known false positive in this tool version and is explicitly allowed by the project guidelines.
- **Accepted dimension errors:** The board outline DRC reports errors at the three button cutouts and the USB-C aperture. These are structural slots required by the enclosure and are permitted per project guidelines.
- **Decoupling capacitor placement.** Every 100 nF decoupling capacitor is placed within one pad-length of the IC power pin it serves, on the same layer.
