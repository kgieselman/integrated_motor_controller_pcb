# Integrated Motor Controller PCB

A reusable base board for yearly robotics competitions. The base board handles drive, sensing, control, and radio. Per-season expansion is exposed through a small set of discrete headers (AUX, UART, QWIIC I²C, servo), not a single dedicated mechanism connector.

Adapter boards are named after creature parts: **Fang** (ping pong gripper/launcher), with future adapters extending the menagerie.

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        Base Board                                │
│                                                                  │
│  ┌───────────────────┐  ┌──────────────────────────────────────┐ │
│  │ Power             │  │  STM32 H563RIT6                      │ │
│  │ 2S/3S → 5V ×2     │  │  250 MHz, 2MB / 640KB                │ │
│  │ (aux + main)      │  │  Y1 ASE-25MHz active oscillator      │ │
│  │ 5V → 3.3V LDO     │  │  BZ buzzer, D6–D9 LEDs               │ │
│  │ USB/batt PowerPath|  └────────────┬─────────────────────────┘ │
│  └───────────────────┘               │                           │
│                                      │                           │
│  ┌───────────────────────────────────┼───────────────┐           │
│  │   DRV8874 ×2 Motor Drivers        │               │           │
│  │   ~9A peak each                   │  Encoders ×2  │           │
│  │   IN1/IN2 + IPROPI                │  TIM4 / TIM8  │           │
│  └───────────────────────────────────┴───────────────┘           │
│                                                                  │
│  Connectors (* = DNP, footprint reserved):                       │
│   J1  RCVR    1×4 pin       — ELRS/CRSF RX/TX + 5 V *            │
│   J2  SWD     2×3 pin       — SWDIO/SWCLK/nRST, 3.3 V *          │
│   J3  ENC 0   1×3 pin       — encoder 0 signal, 3.3 V *          │
│   J4  ENC 1   1×3 pin       — encoder 1 signal, 3.3 V *          │
│   J5  UART    1×3 pin       — expansion UART, JST GH 1.25 mm *   │
│   J6  CH0     1×3 pin       — servo 0, +5V_Aux, GND *            │
│   J7  I2C     1×4 pin       — I2C SDA/SCL, JST SH 1.0 mm *       │
│   J8  CH1     1×3 pin       — servo 1, +5V_Aux, GND *            │
│   J9  I2C     1×4 pin       — I2C SDA/SCL (alt), JST SH 1.0 mm * │
│   J10 CH2     1×3 pin       — servo 2, +5V_Aux, GND *            │
│   J11 USB-C   USB 2.0       — debug / VBUS-powered logic         │
│   J12 VMeter  1×2 pin       — +BATT/GND for external voltmeter * │
│   J13 BATT    XT60          — 2S/3S LiPo input *                 │
│   J14 MOT 0   1×2 pin       — motor 0 out, Molex Micro-Fit 3.0 * │
│   J15 MOT 1   1×2 pin       — motor 1 out, Molex Micro-Fit 3.0 * │
│   JP1/JP2     solder jumpers — CRSF RX path (bridged by design)  │
└──────────────────────────────────────────────────────────────────┘
```

## Key Specs

| | |
|---|---|
| **MCU** | STM32H563RIT6 — Cortex-M33 @ 250 MHz, 2 MB flash, 640 KB SRAM, LQFP-64 |
| **Clock** | Y1 ASE-25.000MHZ — Abracon active oscillator, 4-pin SMD 3.2×2.5 mm |
| **Motor drivers** | 2× DRV8874 (PWP-16, HTSSOP) — IN1/IN2 mode (PMODE driven high), IPROPI current sense |
| **IMU** | ICM-42688-P — 6-axis, SPI1 on PB3/PB4/PB5, CS=PC12 |
| **EEPROM** | M24C64 — 64 Kbit, I²C1 on PB6/PB7, WC=PA9 |
| **RC control** | External CRSF/ELRS receiver via J1 (1×4); JP1/JP2 solder jumpers bridged by default |
| **Servo channels** | 3, powered from +5V_Aux buck |
| **Expansion UART** | JST GH 1.25 mm |
| **I2C expansion** | JST SH 1.0 mm, 1×4 |
| **Power input** | 2S/3S LiPo (7.4–12.6 V) via XT60; USB-C VBUS can power logic rails only |
| **Power topology** | SW1 power switch; SI7157DP; 2N7002; AP63205WU bucks (5V_Aux + 5V); AP1117-33 LDO for 3.3 V |
| **Debug** | SWD, MCU RST, MCU BOOT0, USR button, power switch, LEDs, buzzer |
| **Board** | 4-layer FR4 |

## Schematic Hierarchy

```
integrated_motor_controller.kicad_sch (Root)
├── power_regulator_2s_3s.kicad_sch — Battery input, power switch, bucks, LDO, VBAT sense
├── mcu_h563r.kicad_sch             — STM32H563, Y1 active oscillator, bypass, pin assignments, VCAP, SW2/SW3
├── motor_driver_drv8874.kicad_sch  — DRV8874 (instantiated twice: Motor 0 + Motor 1), motor output connectors
├── imu_icm42688.kicad_sch          — ICM-42688-P on SPI1
├── eeprom_m24c64.kicad_sch         — M24C64 on I²C1
├── debug_leds.kicad_sch            — Debug LEDs (×4), buzzer, user button, transistor drivers
└── connector_usb.kicad_sch         — USB-C receptacle + USBLC6-2SC6 ESD + CC pulldowns
```

Servo, encoder, UART, I2C, RCVR, and SWD connectors are placed directly on the root sheet.

## Status

- [x] Schematic hierarchy defined
- [x] Sub-schematics created and wired
- [x] PCB layout complete (KiCad 10)
- [x] Gerber / drill files generated (`gerbers/`)
- [ ] ERC clean
- [ ] DRC clean
- [ ] BOM and pick-and-place outputs
- [ ] First board assembled and tested

## Known open items

- **(P9)** PB6/PB7 as I²C1 SDA/SCL for EEPROM — confirm against H563 AF table and physical routing before fabrication
- **(P10)** VBAT_SENSE divider Thévenin impedance (~23 kΩ) at upper edge of STM32H5 ADC preferred source impedance
- **(P11)** BOM reference-designator ordering is out of sync with schematic order in several rows

## Getting Started

1. Clone this repo
2. Open `integrated_motor_controller.kicad_pro` in KiCad 10+
3. Custom symbols are in `libraries/symbols/` — resolved via the committed `sym-lib-table`
4. Custom footprints are in `libraries/footprints/` — resolved via `fp-lib-table`

## Repo Structure

```
integrated_motor_controller-pcb/
├── integrated_motor_controller.kicad_pro              # KiCad project file — open this
├── integrated_motor_controller.kicad_sch              # Root schematic
├── integrated_motor_controller.kicad_pcb              # PCB layout
├── power_regulator_2s_3s.kicad_sch
├── mcu_h563r.kicad_sch
├── motor_driver_drv8874.kicad_sch
├── imu_icm42688.kicad_sch
├── eeprom_m24c64.kicad_sch
├── debug_leds.kicad_sch
├── connector_usb.kicad_sch
├── gerbers/                       # Gerber + drill files (ready for fab)
├── output/
│   ├── integrated_motor_controller.csv                # BOM export
│   └── integrated_motor_controller.net                # Netlist export
├── libraries/
│   ├── symbols/                   # Custom KiCad symbol libraries (.kicad_sym)
│   ├── footprints/                # Custom KiCad footprint libraries (.kicad_mod)
│   └── 3d_models/                 # STEP/WRL 3D models
├── sym-lib-table                  # Symbol library table (committed — paths are relative)
└── fp-lib-table                   # Footprint library table (committed — paths are relative)
```

## License

[CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S-2.0)](LICENSE) © 2026 Kyle Gieselman
